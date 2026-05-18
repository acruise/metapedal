# Metapedal DSP and Audio Processing

This document covers the DSP architecture, the Faust language choice, the format and latency configuration matrix, external DSP acceleration options, and a detailed DSP budget analysis with worked examples for guitar pedalboard, electronic music, and busker use cases.

The audience is engineers writing effects and synth code for the platform, and platform implementers making decisions about what fits on the H7 versus what needs the BX variant's SHARC.

For the analog signal path that feeds into and out of the DSP, see `tech-02-audio-analog.md`.

For the streaming router that connects DSP blocks within a module, see `tech-05-web-platform.md`.

## The DSP Story

The DSP capability on a Cortex-M7 is neither a separate ASIC nor a software library competing for general CPU cycles. It is the CPU's instruction set augmented with specialized hardware blocks for math-heavy operations like multiply-accumulate, saturating arithmetic, packed SIMD operations, and single-cycle floating-point math. The compiler emits instructions that use these specialized blocks for DSP inner loops. The CPU executes those instructions on the dedicated hardware in the same pipeline as everything else.

This means DSP processing competes for CPU cycles with other software running on the CPU, but it does not compete with audio I/O itself. The audio I/O on the H7 is handled by dedicated hardware that runs autonomously. Specifically, the audio codec connects through the Serial Audio Interface peripheral, which manages sample-by-sample I/O with the codec through hardware FIFOs. The Direct Memory Access controller continuously moves samples between the SAI's FIFOs and a circular buffer in memory, without CPU involvement. The CPU sees only a block-boundary interrupt, once per processing block, when a new block of samples is ready for processing. At a sixty-four-sample block size and forty-eight kilohertz sample rate, this is an interrupt every one and a third milliseconds, which is a very modest interrupt rate.

The audio path through the system therefore looks like this. The codec sends samples to the SAI. The SAI buffers samples and signals the DMA. The DMA writes samples to memory without CPU involvement. Once per block, the DMA tells the CPU that a block is ready. The CPU runs the DSP processing on that block. The CPU writes the processed output back into a different region of the circular buffer. The DMA reads from that region and sends samples to the SAI. The SAI sends samples to the codec. The CPU is involved only at the block boundaries, and the actual sample-by-sample I/O happens in hardware while the CPU is free to do other work.

This pattern is called rate-monotonic scheduling and it has been the right way to build real-time audio systems for decades. The audio processing task runs at the highest priority and preempts everything else when a block is ready. Lower-priority tasks like the OLED display updates, button polling, USB handling, and inter-module bus protocol all run in the time between audio processing runs. As long as the audio processing comfortably finishes each block in time, the other tasks all get adequate service in the remaining time.

The latency budget that emerges from all this is genuinely good. With a sixty-four-sample block size, the analog-to-analog latency for processing through a single module is around three and a half milliseconds. With a more aggressive thirty-two-sample block size, it can come down to about two and a quarter milliseconds. Both are well below the threshold where guitarists notice latency, which is somewhere around five to ten milliseconds depending on the player. This is competitive with the lowest-latency commercial digital amp modelers on the market, which typically achieve around three to five milliseconds through similar engineering.

The variability of the latency, the jitter, matters as much as the absolute value for the feel of the instrument. A pipeline that produces a stable three milliseconds of latency feels much better than one that produces three milliseconds on average but occasionally five or seven when something else preempts the audio processing. The deterministic scheduling on the H7 is what makes this possible.

## The Format and Latency Configuration Matrix

The audio format is not a fixed product decision but a tunable runtime parameter, with the right setting depending on what the user is doing. The dimensions of configuration are sample rate, bit depth, channel count, and block size. The realistic ranges for each give a multi-dimensional matrix of possible configurations, most of which are not useful, with a small set of meaningful operating points scattered through it.

The sample rate axis sits in the 48 kilohertz family. This is a deliberate editorial choice, and it deserves explanation. The 44.1 kilohertz rate, ubiquitous in consumer audio because of its CD heritage, has no acoustic or signal-processing justification. It is a number derived from the line and field structure of analog television, because in the late 1970s digital audio was stored on videotape using PCM adaptors, and the sample rate had to be compatible with PAL and NTSC video formats simultaneously. The 48 kilohertz rate, by contrast, was specified by the Audio Engineering Society as the professional standard and was later adopted by DVD, Blu-ray, and digital television. It gives more headroom for anti-aliasing filter design, it divides cleanly into many useful sub-rates for clock generation, and it has no historical baggage. For a platform launched in 2026, there is no good reason to carry forward 44.1 kilohertz as a foundational choice.

Real-time sample rate conversion between 44.1 and 48 kilohertz is also genuinely difficult to do without compromise. The ratio between the two rates is 160 over 147, which is not a simple ratio, which means the resampling filters have to be more complex than they would be for nicer ratios. Good real-time converters can do this with measurable quality losses below the threshold of audibility, but the conversion always costs something in latency or computation, and the cost accumulates if multiple conversions happen in series. The Metapedal architecture avoids this entirely by operating at 48 kilohertz throughout the chain, with conversion to other rates happening only at the system boundary if needed.

The sample rates supported internally are therefore members of the 48 kilohertz family: 12 kilohertz, 24 kilohertz, 48 kilohertz, and 96 kilohertz. These are all related by simple powers-of-two scaling, so conversion between them is trivial decimation and interpolation that introduces no artifacts. The lower rates are efficiency modes for narrow-bandwidth signals. A bass guitar has meaningful spectral content only up to a few kilohertz, so 24 kilohertz sampling is plenty for it, with significant savings in buffer memory, processing time, and latency. A general-purpose audio path uses 48 kilohertz. A high-resolution mode uses 96 kilohertz. Conversion to 44.1 kilohertz is supported but only at the output boundary, in software on a connected computer or in a single dedicated boundary module, and is treated as a legacy option for users who specifically need it.

The bit depth axis is more straightforward. Sixteen-bit audio gives ninety-six decibels of theoretical dynamic range, which is more than the dynamic range of most musical instruments and their analog signal chains. Twenty-four-bit audio gives one hundred and forty-four decibels, which is overkill for most uses but provides headroom for premium applications and for signals that need to survive multiple stages of processing without quality loss. The supported bit depths are sixteen and twenty-four.

The channel count axis is either one or two. Stereo matters for some instruments and contexts and not for others. A bass guitar is intrinsically mono. A keyboard or a stereo effects chain is intrinsically stereo. The system supports both.

The block size axis controls the latency-versus-throughput tradeoff. Smaller blocks mean lower latency but more interrupt overhead and less efficient processing per sample. Larger blocks mean higher latency but more efficient processing and more headroom for complex algorithms. The supported block sizes are 32, 64, 128, and 256 samples.

The cross-product of all these axes is too many configurations to expose directly to users. The right user-facing model is a small set of named configurations, each corresponding to one cell in the matrix that is sensible for a specific use case. The named configurations are organized first by instrument type, then by use case within that type.

For bass guitar, the configurations might be "bass, low latency" at sixteen-bit mono 24 kilohertz with thirty-two-sample blocks, "bass, balanced" at sixteen-bit mono 24 kilohertz with sixty-four-sample blocks, and "bass, studio" at twenty-four-bit mono 48 kilohertz with one hundred and twenty-eight-sample blocks. For electric guitar, the configurations might be "guitar, low latency" at sixteen-bit mono 48 kilohertz with thirty-two-sample blocks, "guitar, balanced" at twenty-four-bit mono 48 kilohertz with sixty-four-sample blocks, "guitar, studio" at twenty-four-bit mono 48 kilohertz with one hundred and twenty-eight-sample blocks, and "guitar, stereo" at twenty-four-bit stereo 48 kilohertz with one hundred and twenty-eight-sample blocks. For voice or general instrument input, the configurations might be "voice, live" at twenty-four-bit mono 48 kilohertz with sixty-four-sample blocks and "voice, studio" at the same format with one hundred and twenty-eight-sample blocks. For synthesizers and full-range sources, the configurations might be "synth, low latency" at twenty-four-bit stereo 48 kilohertz with sixty-four-sample blocks, "synth, studio" at twenty-four-bit stereo 48 kilohertz with one hundred and twenty-eight-sample blocks, and "synth, high resolution" at twenty-four-bit stereo 96 kilohertz with two hundred and fifty-six-sample blocks.

The user picks a named configuration that matches their situation. The underlying parameters get set accordingly. The system operates at that point in the matrix. Advanced users can dig into the underlying parameters and build custom configurations, but the typical user just picks from the catalog of named options.

The user interface can also be predictive rather than reactive. The module knows what is plugged into it, in the sense that the user has declared the instrument type. The module knows what effects are configured and what the inter-module chain looks like. The system can recommend a configuration based on this information, showing estimated latency and processing headroom for each option, letting the user make informed choices through a visual interface that exposes the implications. The configuration becomes a tool for understanding the tradeoffs, not an opaque setting that users tweak by trial and error.

## External DSP Acceleration

Some users want serious DSP horsepower that goes beyond what the onboard H7 can deliver. High-resolution neural amp modeling with cabinet impulse responses, large convolution reverbs, real-time spectral processing, and other premium algorithms can outrun what fits in a few hundred microseconds of H7 budget. The architectural answer is an external DSP peripheral that plugs into one of the module's expansion ports and takes over the heavy lifting.

The host module routes audio to the external DSP through one of its SAI peripherals, the processing happens on the external chip, and the result comes back through the same or a different SAI port. The H7 acts as a sample-accurate router rather than as the primary processor for the audio. Latency overhead is small if the H7 and the external DSP share a master clock and use small block sizes for the transfers between them, on the order of a single sample period in each direction.

The right external DSP chip is an open question and probably should remain an implementation choice rather than a specification mandate. The classic answer is the Analog Devices SHARC family, which has dominated high-end audio DSP for decades, with proven tooling and a mature algorithm ecosystem. A more modern answer might be a chip from the Texas Instruments TMS320 family with more open tooling. An adventurous answer might be a small FPGA from Lattice or Xilinx, giving users with the skills the ability to implement custom DSP architectures that no fixed-architecture chip can match. The specification defines the protocol and capabilities for an external DSP peripheral, and different peripherals can use different underlying silicon as long as they implement the protocol. A budget peripheral using a TI part, a high-end one using a SHARC, and an experimental one using an FPGA could all coexist.

The external DSP, from the perspective of the rest of the Metapedal chain, is invisible. The host module is still the audio processor in the chain, with its parameters and presets and routing. The external chip is just an accelerator that the host module uses. This abstraction is what keeps the chain protocol simple and pushes the complexity of DSP acceleration into the peripheral interface where it belongs.

## The DSP Language: Faust

DSP algorithms running on Metapedal modules need to be written in some language. The choice of language affects the developer experience, the ecosystem of available effects, the portability of code across processing targets, and the barrier to entry for users who want to write their own effects.

The pragmatic answer for Metapedal is to adopt Faust, which stands for Functional Audio Stream, as the primary DSP language for the platform. Faust has been developed since the early 2000s at GRAME in Lyon and is the most mature open-source domain-specific language for audio. The language is purely functional, with signal flow expressed as composition of operators. A simple low-pass filter in Faust is a single line that reads almost like mathematical notation. The Faust compiler can target C, C++, Rust, JavaScript, WebAssembly, LLVM, several DSP-specific architectures, and various plugin formats. There is an active community, a substantial library of pre-written algorithms, and good documentation. Faust has been used in commercial products, academic research, and open-source projects, so it is battle-tested.

The case for Faust is strong on several dimensions. The language is well-suited to the kinds of effects algorithms that pedals run. The compiler can generate efficient code for the H7 and for the external DSP targets. The existing library covers many common cases out of the box. The syntax is approachable enough that motivated users can write their own effects without being embedded systems experts. The compiler produces deterministic output with predictable timing characteristics, which matches the real-time constraints the platform requires.

There are some concerns worth being explicit about. Faust is a separate language with its own learning curve, which is a real barrier for some users. The functional model is great for stateless or simple-state effects but awkward for effects with complex internal state or non-trivial control flow. Most pedal effects fit Faust's model comfortably, but the ones that do not fit can be painful to express. Performance is usually good but not always optimal; for the most demanding algorithms, hand-optimized native code can be somewhat faster than Faust-generated code. These are real limitations but they are manageable.

The integration with the rest of the Metapedal runtime takes some work. Faust algorithms are stateless from the compiler's perspective, with all state passed explicitly through algorithm interfaces. The Metapedal runtime needs to manage that state across preset switches, save it in presets, restore it when presets are loaded, and so on. This requires a runtime design that knows how to introspect Faust-compiled algorithms and extract their state. The Faust project has thought about this and there is reasonable infrastructure for it, but the integration is a non-trivial engineering effort that the platform needs to do once and then can reuse.

The licensing situation is favorable. Faust is dual-licensed under LGPL for the runtime and GPL for the compiler. The LGPL runtime is compatible with proprietary use, which matters because module manufacturers might want to ship Faust-based effects without open-sourcing their entire firmware. The GPL compiler is more restrictive but only applies to the compiler tooling, not to the generated code. This is essentially the same licensing model as GCC, which has worked well for commercial use, so it should be acceptable for Metapedal.

The consequences of choosing Faust are mostly upside. The existing Faust library becomes available to Metapedal users, providing a substantial collection of audio processing primitives from basic filters and oscillators through compressors, reverbs, distortions, modulators, and the other common building blocks. Effects developed for Faust on other platforms can potentially be ported to Metapedal with minimal effort. Faust algorithms can target both the H7 onboard processing and the external DSP accelerator from the same source code, with the compiler choosing the appropriate target based on what hardware is present. Users can prototype custom effects in Faust's web-based playground without any local toolchain setup, hearing results immediately, and export the algorithm to their Metapedal module once they are happy with it.

Effects can be shared as text files that are essentially small programs, rather than as binary firmware images. This has nice implications for the ecosystem. Users can email each other interesting effects. There can be a community repository of user-contributed effects with searching, rating, version history, the whole open-source software experience. The Faust source is human-readable, so users can learn from each other's effects, modify them, build derivatives. The cultural artifact of the Metapedal community starts to look like an open-source software project rather than a closed hardware ecosystem.

Faust should not necessarily be the only language for Metapedal effects. Some module manufacturers will want to write effects in C or C++ directly, perhaps because they have existing algorithm investments or want maximum performance. Some hobbyists will want to use higher-level graphical programming tools. The architectural pattern is that the Metapedal Effect Interface is the contract any audio processing function must implement, and Faust is the recommended way to produce effects conforming to that contract but not the only way. Future versions can add other languages or visual tools that produce conforming effects without users writing text. The contract stays stable while the languages that produce effects evolve.

For version one, the right move is to ship Faust as the recommended effect description language, with a small set of effects implemented in Faust as part of the platform's standard library, with the web-based Faust playground integrated into the documentation as the recommended way to experiment, and with a clear path for users to write their own effects and load them onto their modules.

## DSP Budget Analysis: What Fits Within One H7

A practical question for both users and developers is what kinds of DSP loads fit within a single H7's compute budget. This section provides a technical analysis of effect complexity and example algorithm costs, complementing the user-facing capacity discussion in the vision document.

### Available Budget

The STM32H7 family parts run at 480 MHz (some variants up to 550 MHz, but treat 480 MHz as the conservative number). At 48 kHz mono audio, this gives 480,000,000 / 48,000 = 10,000 cycles per sample of total compute. The system reserves perhaps 20-30% of this budget for control loops, I/O servicing, USB protocol handling, routing graph maintenance, and other non-DSP duties, leaving roughly 7,000-8,000 cycles per sample for the user's DSP work. Stereo processing halves this per-sample budget but doubles the per-second throughput, so the total DSP capacity is similar regardless of channel count.

In megacycles per second of pure DSP, the available budget is roughly 350-400 megacycles, which is substantial for embedded audio. By comparison, classical pedal-targeted DSP chips like the Spin FV-1 have 100-megacycle equivalents at 32 kHz; the H7 has roughly 4x the compute budget of typical commercial pedal DSP. By comparison, desktop CPUs running plugins have 1000-10000x more raw compute, so the H7 is meaningfully more limited than a laptop but meaningfully more capable than traditional pedal DSP.

### Algorithm Cost Tiers

DSP algorithms can be tiered by their per-sample cost relative to the H7's budget.

**Trivial tier**, under 100 cycles per sample: gain, panning, polarity invert, hard clipping at known thresholds, simple one-pole filters, A/B switching, tuner display, level meter, parameter smoothing. The H7 can run hundreds of these simultaneously without impacting the main DSP budget.

**Lightweight tier**, 100-500 cycles per sample: parametric EQ filters (biquad implementations cost ~30 cycles each), graphic EQ (multiple parallel biquads), noise gates with envelope following, optical compressors (LA-2A-style smoothed control voltage), simple distortions using waveshaping (Tube Screamer, basic Big Muff style), tremolos with LFO modulation, simple choruses with one delay line, ring modulators, simple wah filters with manual control. Each costs 1-5% of the per-sample budget. A typical pedalboard chain of 6-10 lightweight effects in series fits comfortably with 60-80% budget headroom remaining.

**Medium tier**, 500-2,000 cycles per sample: digital delays with feedback, tap tempo, and modulation (Strymon DIG-style); spring reverb simulations (using cascaded all-pass filters and small delay lines); multi-voice choruses (3-4 delay lines with separate LFOs); phasers (4-6 stage all-pass cascades with LFO sweep); flangers (modulated short delay with feedback); pitch shifters using PSOLA at moderate ratios; octavers using envelope-following pitch detection; envelope-following wahs; plate reverbs (Dattorro-style algorithm with all-pass network); basic algorithmic reverbs (Freeverb-style). Each costs 5-20% of the budget. 4-6 medium effects fit in a single H7 with reasonable headroom.

**Heavy tier**, 2,000-7,000 cycles per sample: realistic amp simulations with multi-stage tube preamp modeling, power amp saturation, tone stack EQ, and convolution-based cabinet/IR simulation (Axe-FX-style amp models cost 3,000-5,000 cycles per sample for the amp plus 2,000-4,000 cycles for a typical IR convolution); convolution reverbs with 1-2 second impulse responses (around 100,000 samples at 48 kHz, requires partitioned convolution at ~3,000-5,000 cycles per sample for direct implementation, less with FFT acceleration); spectral pitch shifters using STFT (FFT-based, around 3,000-6,000 cycles per sample depending on window size and overlap); granular synthesizers with 8-16 simultaneous grains; complex modular oscillator banks with many simultaneous voices. A single H7 can run one heavy effect plus a couple of lighter effects, or two medium-heavy effects if the loads are balanced.

**Beyond a single H7**, requiring more compute than the H7 budget provides: neural-network amp models that match Neural DSP's commercial quality (these require tens to hundreds of millions of operations per second of additional compute beyond what the H7 has); studio-quality real-time pitch correction (Antares Auto-Tune at maximum quality settings); large convolution reverbs with 5-10 second impulse responses (the partitioned-convolution cost grows linearly with IR length); high-quality time-stretching at extreme stretch ratios (requires high-resolution spectral analysis and synthesis); high-channel-count processing (16+ channels with full per-channel DSP processing). These either require multiple H7s working together (distributed across modules in a chain), or they require waiting for a more capable processor variant in a future version of the platform.

### Concrete Algorithm Cost Examples

Specific algorithms with measured or estimated costs:

A single biquad filter (used as a building block in EQs, simple resonant filters, etc.) costs roughly 30 cycles per sample. A four-band parametric EQ uses 4 biquads, costing 120 cycles. A 31-band graphic EQ uses 31 biquads, costing roughly 1,000 cycles, which is a medium-tier load.

A standard delay with feedback, tap tempo, and modulation costs maybe 200-500 cycles per sample depending on features (interpolation quality for the delay tap, filtering in the feedback path, modulation LFO).

A Freeverb-style algorithmic reverb costs about 800-1,500 cycles per sample depending on configuration. Freeverb is the de facto reference open-source algorithmic reverb and serves as the "what reasonable algorithmic reverb costs" baseline.

A Dattorro-style plate reverb costs roughly 1,500-2,500 cycles per sample, similar order of magnitude to Freeverb but with different sound character.

A typical convolution reverb with 1-second impulse response (48,000 samples at 48 kHz) using partitioned convolution costs roughly 2,500-4,000 cycles per sample using the H7's SIMD capabilities efficiently. Without FFT acceleration the cost would be much higher; the algorithm benefits substantially from spectral implementation.

A multi-stage tube amp model with preamp distortion, tone stack, and power amp saturation costs roughly 1,500-2,500 cycles per sample for the amp itself, plus the cabinet/IR convolution cost which depends on IR length.

An STFT-based spectral pitch shifter with 1024-sample window and 75% overlap costs roughly 3,000-5,000 cycles per sample, putting it firmly in the heavy tier.

A neural network-based amp model with quality competitive with commercial neural amp products like Neural DSP's plugins would require roughly 50-100 megacycles per sample using current neural network architectures, which exceeds the H7's per-second total budget. Smaller, simpler neural models could fit, but matching commercial neural amp quality is beyond a single H7.

### Memory Budget

Compute is one constraint; RAM is another, but the on-chip RAM picture for the H7 family is much more generous than typical guitar pedal DSP chips, which meaningfully affects what fits in one module.

The STM32H743/H753 has 1 MB of on-chip SRAM organized in several blocks: 64 KB of instruction tightly-coupled memory (ITCM) and 128 KB of data tightly-coupled memory (DTCM) for time-critical code and data with zero-wait-state access, 512 KB of main AXI SRAM accessible from any bus master, 288 KB of SRAM in additional blocks, and 64 KB more in another block, plus 4 KB of backup SRAM. The STM32H7B0/H7B3 variants have 1.4 MB total with similar architecture.

After firmware code, control structures, audio buffers, the routing graph state, USB and protocol stacks, and other system overhead, perhaps 700-800 KB of RAM is realistically available for user DSP work on the H743. This is dramatically more than typical commercial pedal DSP chips. The Spin FV-1 (a common pedal DSP) has roughly 32 KB of working memory; classical reverb-on-a-chip parts have 16-64 KB; the H7 has 10-25x more usable RAM than these traditional pedal DSP options. This matters because RAM-intensive effects (delays, reverbs, convolution) are inherently more tractable on the platform than they would be on traditional pedal hardware.

The TCM portion of the RAM (192 KB total) deserves separate mention because it has zero-wait-state access from the CPU, making it ideal for the inner loops of DSP code where every cycle matters. Critical filter coefficients, delay line read/write pointers, and other hot-path data structures live in TCM; the bulk of delay buffers and other large data structures live in the main SRAM blocks. The Faust compiler and the platform's DSP runtime can be set up to place code and data appropriately across the memory hierarchy.

Delay lines and convolution buffers dominate the memory budget for serious effect chains. A 1-second mono delay at 48 kHz is 48,000 samples; at 4 bytes per sample (32-bit float) that is 192 KB. A 2-second delay is 384 KB. A typical reverb has multiple smaller delay lines totaling perhaps 100-300 KB depending on the algorithm. A convolution reverb's impulse response storage is roughly the IR length times 4 bytes per sample; a 1-second IR is 192 KB, a 2-second IR is 384 KB.

With 700-800 KB realistically available, a single module can run substantial RAM-intensive effects: two 1-second delays plus an algorithmic reverb plus a full chain of lightweight effects, for instance. Or one 2-second convolution reverb (384 KB) plus algorithmic reverb plus an amp simulator plus several lightweight effects. Or one 1-second delay plus a 1-second convolution reverb plus everything else. The 384 KB convolution reverb that I had been describing as "would tax a single module" is actually quite reasonable on the H7's RAM budget; the compute side stays heavy but the memory side is comfortable.

This is one of the platform's quiet advantages over traditional pedal DSP. Effects that are difficult or impossible on commercial pedal DSP chips because of RAM limitations become natural on the H7 because the H7 simply has the memory. Users who care about long reverbs, multi-tap delays, granular effects, or other RAM-hungry algorithms get meaningfully more capability per module than competitive products at similar price points.

### External SDRAM Expansion

For users who need more RAM than 1 MB on-chip provides, the H7 supports external SDRAM through its FMC (Flexible Memory Controller) interface. The FMC connects to standard SDR-SDRAM chips at clock rates up to 100 MHz with 16-bit data width on the H743 (133 MHz maximum but limited by power voltage scaling), giving substantial bandwidth for DSP work. Typical SDRAM chips for this application are 16-256 megabit parts (2-32 megabytes), costing $2-5 in production quantities and providing 16-32x more RAM than the on-chip alone.

External SDRAM enables effects that would not otherwise fit in any single module: very large convolution reverbs with multi-second impulse responses (a 5-second stereo IR is roughly 4 MB), large sample libraries for sampler modules, multi-track audio buffers for live looping with many minutes of capacity, or scientific-quality audio analysis with large historical context.

The cost of supporting external SDRAM on the motherboard is real and worth being explicit about. The FMC routing requires roughly 30-40 traces between the H7 and the SDRAM chip, with length matching and impedance control at the high speeds involved. The PCB area for this routing is substantial on a small pedal-sized board. The SDRAM chip itself adds the $2-5 BOM cost plus the supporting decoupling capacitors and termination resistors.

For version one, the right decision is probably to NOT include FMC routing on the standard A and B motherboards. The reasoning is that the on-chip RAM is generous enough for the platform's launch use cases, the PCB area cost of FMC support is meaningful on a small board, and adding SDRAM as a populate-on-demand option creates user confusion about which modules have it. The standard v1 modules use on-chip RAM only, which keeps the design simple and the BOM minimal.

Users who need more RAM for specific use cases have two paths. The first is multi-module scaling: a chain of multiple modules effectively has more RAM available because each module has its own RAM, and effects can be distributed across modules. A 4-module chain has roughly 4 MB of usable RAM in aggregate, which is more than most use cases need. The second is waiting for a future variant or v2 module that includes SDRAM. A "Pro" variant with external SDRAM could come as a follow-on product for users who need single-module RAM beyond what on-chip provides.

This positioning is similar to how the platform handles other capabilities. Most users get a good baseline experience with the standard modules; users with specific advanced needs get specialized variants or wait for future versions. The platform's modularity makes this gradual capability expansion possible without breaking compatibility with earlier modules.

### Multi-Module Scaling

A chain of modules has compute budget that scales linearly with module count. A chain of 4 B variant modules has roughly 4x the compute budget of one module, allowing heavier configurations than a single module supports. The routing graph distributes work across modules, with the inter-module bus carrying audio between processing stages.

For users running heavy effects, the practical approach is to dedicate one module to each heavy effect plus combine multiple lighter effects on one shared module. A chain might allocate one B2 to an amp simulation with cabinet IR, one B2 to a high-quality reverb, and one B2 to a combination of modulation and delay effects. The combined chain runs effects that exceed what any single module could do.

The DSP capacity story is therefore better than any single module suggests: users can run high-end production-quality effect chains by combining modules in chains, with the routing graph handling the audio flow between processing stages. The platform's modular architecture means DSP capacity scales with hardware investment, with no software architecture work required to add more compute.

### Concrete Scenario: The One-Person Band on a Single Module

A useful exercise is to walk through a specific demanding scenario and see what fits on a single H7. The "busker" scenario combines three distinct DSP loads:

1. A polyphonic synthesizer (DX7-style FM or Hammond-style organ) playing back a MIDI file from microSD as a backing track. The synth provides backing instrumentation while the busker plays guitar and sings over it.
2. Voice processing on the busker's microphone: high-pass filter, compressor, EQ, and reverb appropriate for live vocal performance.
3. Guitar processing: noise gate, compressor, amp simulation with cabinet emulation, EQ, modulation, delay, and reverb. A complete guitar effects chain.

The MIDI file playback itself is essentially free (under 100 cycles per sample for parsing events and triggering note-on/note-off in the synth engine). The interesting costs are the synthesis triggered by those events plus the audio processing for the vocal and guitar channels.

**Cycle budget estimates:**

The DX7 synth engine costs depend on voice count. The MiniDexed project (which runs Synth_Dexed, the embedded port of the Dexed DX7 emulation) has established that Teensy-class hardware (Cortex-M7 at H7-comparable clock rates) can run 8 voices of polyphonic 6-operator FM synthesis. For a backing track scenario with reasonable musical density, 8-12 voices are appropriate. Cost estimate: roughly 2,000-3,000 cycles per sample for 8-12 voices of DX7-quality synthesis. The exact cost depends on patch complexity (some patches use more operators heavily than others).

The vocal processing chain at modest quality: high-pass filter (30 cycles), compressor (150 cycles), 3-band parametric EQ (90 cycles), plate-style algorithmic reverb (1,500-2,000 cycles). Total: roughly 1,800-2,300 cycles per sample.

The guitar processing chain with amp simulation: noise gate (50 cycles), compressor (150 cycles), amp model with tube saturation stages (1,500-2,000 cycles), cabinet IR convolution (1,000-1,500 cycles depending on IR length), EQ (60 cycles), modulation effect like chorus (500 cycles), delay with modulation (400 cycles), algorithmic reverb (1,000 cycles). Total: roughly 4,700-5,700 cycles per sample.

**Total combined budget:** 8-12 voices DX7 (~2,500 cycles) + vocal chain (~2,000 cycles) + guitar chain (~5,200 cycles) = roughly 9,700 cycles per sample. This exceeds the H7's usable budget of 7,000-8,000 cycles per sample by 20-40%. The full maximum-quality version of this scenario does not fit on a single H7.

**The confident single-chip configuration** (within budget):

- DX7 backing track with 6-8 voices (~1,500 cycles instead of 2,500): patches need to be chosen for moderate polyphony, or the MIDI file needs to be arranged for fewer simultaneous voices. Most actual backing track music works fine with 4-8 simultaneous voices.
- Vocal chain at modest quality with smaller reverb (~1,300 cycles instead of 2,000): replace the plate reverb with a simpler algorithmic reverb that costs less.
- Guitar chain with simplified amp model (~3,500 cycles instead of 5,200): use a parametric cabinet simulation rather than full IR convolution, saving 1,000-1,500 cycles.
- Total: roughly 6,300 cycles per sample, comfortably within the 7,000-8,000 budget.

This is the configuration the platform should confidently claim to deliver. A user has a complete one-person performance rig on a single $80-95 module: backing tracks from microSD, vocal effects, guitar processing with amp simulation, all running simultaneously. The cost is using moderately optimized DSP rather than the absolute maximum-quality version of each capability.

**The stretched single-chip configuration** (marginal):

- 8-12 voices DX7 at full quality (~2,500 cycles)
- Full vocal chain with plate reverb (~2,000 cycles)
- Full guitar chain with IR-based cabinet (~5,200 cycles)
- Total: ~9,700 cycles, exceeding budget by 20-40%

This configuration would require either accepting some compromise (occasional buffer underruns under maximum load, reduced quality on one of the chains during demanding passages) or careful optimization (better Faust compilation, hand-tuned inner loops, SIMD optimization). Not the configuration to confidently commit to.

**The comfortable multi-module configuration:**

- Module 1: DX7 backing track at full quality with 12-16 voices
- Module 2: Vocal processing chain with plate reverb and any specialty effects
- Module 3: Guitar chain with full IR-based amp sim and complete effects
- Total: ~9,700 cycles across three modules, each well within its own budget
- Total cost: three modules instead of one (~$240-285 retail instead of $80-95)

This is the right setup for a busker who wants the full experience without compromise. The platform's modularity means the upgrade path is clean: start with one module, add more as the use case demands.

**RAM budget for the same scenario:**

DX7 synthesis uses minimal RAM (sine wave tables and voice state are tiny, total well under 100 KB). The vocal chain's reverb uses ~100-200 KB for delay lines. The guitar chain's cabinet IR uses ~200 KB (for a 1-second 48 kHz mono IR), delay lines use ~100-200 KB, and the algorithmic reverb uses ~100 KB internally. Total: roughly 500-700 KB out of the H7's 700-800 KB available. Tight but feasible on H7; comfortable on H7B0/H7B3 with 1.4 MB.

### Concrete Scenario: The Electronic Music Performance Rig

A different demanding scenario worth working through is electronic music performance: 808-style drum machine emulation plus 303-style bass synth plus optional additional synth voices plus vocal processing. The 808 and 303 are culturally important enough that the platform should support them well, and the DSP economics are different from the guitar-centric busker scenario in interesting ways.

**The TR-808 emulation** is a drum machine with 16 voices, each generated by synthesis rather than samples. The voices have different cost profiles: kick drum is the most expensive (oscillator with frequency modulation, amplitude envelope, pitch envelope, transient generator, perhaps 100-300 cycles per sample when active). Snare combines a tonal component with filtered noise, maybe 150-250 cycles when active. Hi-hat is filtered noise with envelope, 100-200 cycles. The cymbal is more complex with six square wave oscillators, 300-500 cycles. Toms, congas, claves are 100-200 cycles each.

Critical insight: 808 voices are mutually exclusive in interesting ways. A drum pattern has many voices defined but only some are playing at any moment. Realistic active voice count during a busy pattern is 6-8 simultaneous voices, not all 16. Total 808 emulation cost for a busy pattern: roughly 1,000-2,000 cycles per sample. Much less RAM than sample-based drums because voices are synthesized rather than played back from memory.

**The TB-303 emulation** is a monophonic bass synth with an oscillator (cheap, 50 cycles per sample), a famously characteristic resonant ladder filter (the expensive part, 500-800 cycles for nonlinear modeling at quality matching the original's resonance behavior), envelope generators (50 cycles), accent and slide processing (60 cycles). Total: roughly 700-1,000 cycles per sample for a quality 303 emulation. Monophonic so no voice-count multiplication; one 303 is one 303.

**Combined 808 + 303 cost:** roughly 1,700-3,000 cycles per sample for the rhythm section running together. Medium tier. RAM usage minimal because both are synthesis-based; total under 50 KB.

**The electronic busker scenario** combines 808 + 303 + vocal processing + a small additional synth voice:

- 808 drum machine (~1,500 cycles)
- 303 bass synth (~900 cycles)
- 4-voice DX7 pad or lead synth (~1,000 cycles)
- Vocal chain at modest quality (~1,800 cycles)
- Total: ~5,200 cycles per sample

This is comfortably within the single-H7 budget of 7,000-8,000 cycles. Plenty of headroom for additional effects on the synth outputs (drive, delay, reverb on the bass; reverb on the vocals; filter modulation on the lead). The user has a complete electronic music performance rig on a single module: classic 808 + 303 rhythm section, additional synth voice for melodic content, vocal processing with effects. Comparable to commercial gear costing $1,500-3,000.

**Key architectural insight:** electronic music users get more capability per module than guitar users do. The primary DSP needs (drum synthesis and analog-style monosynth) are inherently cheaper than guitar processing (amp simulation with cabinet IR convolution, which dominates the guitar chain cycle budget). A single H7 module that struggles to deliver a maximum-quality guitar rig delivers a full electronic music rig with headroom.

**The hybrid scenario** (electronic backing plus live instruments) is more demanding:

- 808 + 303 (~2,500 cycles)
- 4-voice DX7 pad (~1,000 cycles)
- Vocal chain (~1,800 cycles)
- Simplified guitar chain with amp sim (~3,500 cycles)
- Total: ~8,800 cycles

This exceeds single-H7 budget by 10-15%. Marginal on one module; comfortable on two (synth module handling drums + bass + pad, live audio module handling vocals + guitar).

**The hip-hop producer scenario:**

- 808 (~1,500 cycles)
- 4 simultaneous sample voices for additional percussion or melodic content (~400 cycles when active)
- Vocal chain with light autotune (~2,500 cycles, autotune at modest quality costs 1,500-2,000 cycles)
- Total: ~4,400 cycles

Comfortably within single-H7 budget. The RAM consideration matters here: sample libraries need to live in RAM or be streamed from microSD. A typical hip-hop sample library of 16 drum/percussion samples at 1-2 seconds each in 16-bit format is roughly 1-3 MB, which exceeds the H7's RAM. The module could either limit sample count, stream from microSD with appropriate buffering, or use the H7B0 with 1.4 MB RAM for more headroom. Streaming from microSD is the right answer for serious sample work; the SD card holds the full library and the module loads samples on demand.

### Broader Electronic Music Coverage

The 808 and 303 are iconic examples but the same DSP techniques cover a wider range:

Other drum machine emulations follow similar cost patterns: the TR-909 (more analog-modeled drum sounds, similar cost to 808), the LinnDrum (sample-based but with characteristic processing, RAM-dependent), and various others.

Other monosynth emulations follow the 303's pattern: Minimoog (analog synth with similar filter complexity), Pro-One, SH-101, various analog monosynths. Each costs roughly 700-1,200 cycles per sample for quality emulation.

Sample playback engines for general sampler functionality: cost per voice is low (50-100 cycles when active for a 16-bit sample voice with envelope) but voice count matters and RAM is the main constraint. The microSD streaming approach lets users have arbitrarily large sample libraries while keeping per-module RAM usage manageable.

Wavetable synthesis (PPG/Waldorf style): cycle-cheap (similar to FM synthesis) but has wavetable storage requirements. Cost is mainly in the wavetable storage (perhaps 100 KB per instrument for high-quality wavetables) rather than per-sample compute.

Granular synthesis: more expensive than the above (heavy tier, 2,000-4,000 cycles per sample for many simultaneous grains). Useful for textural and experimental music.

All of these share roughly the same DSP infrastructure: oscillators, envelopes, filters, modulation, sample playback. Faust implementations of each compose with the existing effect library, so a user's synth voice can be processed through the same effects chain as a guitar signal. The platform's modularity extends naturally to electronic music: each module can be configured as a drum machine, a bass synth, a pad voice, or a vocal processor, with the routing graph composing them into a complete performance rig.

### What This Means for the Platform's Story

The "single H7 delivers" story should be honest about what fits and what does not. The platform can confidently claim: a complete one-person performance rig with backing tracks, vocals, and full guitar processing including amp simulation. This is genuinely impressive and competitive with multi-thousand-dollar commercial gear, all on a single $80-95 module.

The platform should not claim that absolute maximum-quality versions of every capability run simultaneously on a single module. That requires either compromises (reduced voice count, simplified amp models) or multiple modules. The honest framing is "modest quality of everything on one module, maximum quality with multiple modules."

This positioning matters for user expectations and platform credibility. Users who buy one module and find that their specific maximalist configuration does not work will be more frustrated than users who buy one module knowing it supports a substantial but not unlimited workload. The platform's modularity is itself a feature: users buy more modules when their needs grow, with predictable scaling.

The same scenario analysis applies to other demanding combinations. A user wanting realistic amp simulation plus high-quality convolution reverb plus several modulation effects might find that the combination exceeds one H7 but fits comfortably on two. A user wanting multi-channel recording (8 channels at 96 kHz / 24-bit) plus full mixing console functionality might want three or four modules. The platform serves these use cases through composition rather than through any single module being magic.
