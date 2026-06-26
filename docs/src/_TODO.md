# Metapedal TODO: Variant Rebranding and Input-Only Module

This document captures proposed changes and ideas that do not yet warrant disrupting the main vision and technical documents. The thinking is preserved here for future review.

## The "I'm Feeling Lucky" Feature

A purely software feature idea, requires no hardware changes, would fit naturally into the platform's overall character once the routing graph and topology awareness are in place.

The idea: a button or footswitch action that rolls dice to generate a random preset, applies it instantly, and lets the user keep pressing to keep rolling. The user explores the parameter space by stomping rather than by tweaking knobs or navigating menus. If they hit something they like, they save it. If they hit something terrible, they roll again. The interaction loop is fast and musical rather than slow and computational.

The topology awareness is the key insight that makes this work well. The system knows what is plugged in and what role each module plays in the chain. A guitar plugged into the front means the dice roll generates effects appropriate for guitar input. An amp on the output means the generated effects should produce signals appropriate for amp consumption. The topology constraints shape what the dice roll is choosing from, so the results are more likely to be musically useful even though the specific choices are random.

This is genuinely different from random parameter generation in DAW plugins. The platform knows the physical setup; software plugins typically just know what is loaded. The richer constraints from physical setup dramatically improve the hit rate of useful generated presets.

Levels of sophistication for the dice rolling:

The simplest version randomizes parameter values within the currently loaded effect chain. Topology stays the same; only parameter values change. Genuinely fun and produces real variety within the constraints the user has set.

A more ambitious version randomizes the effect topology itself. The dice roll picks not just parameter values but which effect algorithms are loaded in which slots, with the routing graph rewiring to match. A distortion-delay-reverb chain might become a flanger-tremolo-pitchshifter chain after one stomp.

The most ambitious version generates new effects on the fly using procedural DSP synthesis. The dice roll picks not just from a catalog of pre-existing effects but constructs new effects from primitives: filters at randomized cutoffs, delay lines at randomized times, distortion curves with randomized shapes, modulators at randomized rates, all wired together in randomized configurations. Faust is well-suited to this because Faust algorithms are essentially small programs built from a vocabulary of primitives. A procedural generator can produce a continuous stream of new DSP algorithms that the platform can compile and run on the fly.

Interesting variations to consider:

Guided exploration where the system tracks which generated presets the user has saved versus rejected, biasing future rolls toward characteristics of saved presets. Over time the dice rolling drifts toward the user's preferences while still exploring around what they already know they like. Turns the feature from pure randomness into a preference-learning exploration tool.

Genetic-algorithm-style mutation rather than random generation. Starting from a saved preset, each dice roll produces a small mutation of the existing preset rather than a complete random one. The user evolves a sound through stages: good, mutate, better, mutate, gone too far, back up, try a different mutation. Becomes a kind of sculpting through dice rolls.

Cultural fit:

The feature fits the platform's overall character nicely. Guitar pedals have a long tradition of users finding sounds through happy accidents (the Big Muff was reportedly discovered this way). Many classic effects originated as bugs that musicians embraced. The dice roll explicitly embraces this tradition: instead of designing your sound, you stumble onto your sound. The platform makes the stumbling fast and convenient.

The feature opens up performance use that traditional pedals do not support. A musician can use the dice button as part of their performance: stomp at the start of an improvisation, stomp midway through a song to surprise themselves, stomp at the start of every gig to keep the sound fresh.

The "I'm Feeling Lucky" naming ties into Google's well-established positive associations. The platform could lean into the naming with a dice icon in the phone app or a distinct color assignment for the footswitch.

Implementation effort is modest because the routing graph and the Faust integration are doing most of the heavy lifting. The dice rolling logic is a relatively small layer that picks values, picks algorithms, or constructs Faust programs procedurally. The user-facing integration is a routing graph action assigned to a control source like any other.

Worth implementing in version one if effort budget allows, or as an early version-two feature otherwise.

## Variant Rebranding and Input-Only Module

This document captures a proposed restructuring of the variant naming and a new product permutation that does not yet warrant disrupting the main vision and technical documents. The idea is worth thinking through but is not yet decided.

### The Proposal

The current variant nomenclature has the A variant as the cheap controller-grade module with TRRS audio only, and the B variants (B1, B2) as the audio-grade modules with quarter-inch jacks. A natural extension would be to introduce variants that have only one direction of audio rather than both, specifically an input-only variant that takes a quarter-inch instrument input but does not have a quarter-inch output. The output, if needed, goes through the TRRS jack for headphone monitoring or through the inter-module bus to other modules in the chain that have outputs.

The proposed renaming under this scheme:

The current A variant (controller-grade, TRRS only) stays as A.

A new B variant introduces an input-only configuration with quarter-inch input plus TRRS plus inter-module audio routing, but no quarter-inch output. The mezzanine for this variant is simpler and cheaper than the current B1 because it has fewer analog components and no output stage.

The current B1 (one combo jack with input and output) becomes C1. Broadly compatible high-fidelity input and output through one combo jack.

The current B2 (two combo jacks) becomes C2.

So the naming would be: A is controller-grade, B is input-only, C is full-duplex audio. The numeric suffixes on the C variants indicate channel count as before.

### Why the Input-Only Variant Might Be Useful

The product justification for an input-only variant is real and worth being explicit about. Several use cases want audio capture without local audio playback:

A pure capture module that digitizes the guitar signal and routes it over the inter-module bus to a downstream Processor or Recorder module. The capture module sits at the front of a chain, with all the processing and output happening on modules further down. The user does not need a quarter-inch output on the capture module because the audio leaves over the inter-module bus.

A USB audio interface mode where the module is connected to a computer and the user only needs audio capture. The dry guitar signal goes to USB for recording, with the user monitoring through the computer's existing audio path. The quarter-inch output that the current B variants include is unused in this configuration and represents wasted cost.

A controller-with-audio-analysis module that uses the input signal for audio-to-control extraction (envelope following, pitch tracking, trigger detection) without producing any audio output. The microphone input on the current A variant serves this role with limited quality through the H7's built-in ADC. An input-only variant with a real codec input gives audio-grade analysis quality without the cost of the output side.

A practice module where the user wants to plug in their guitar, hear it through headphones (via TRRS), and not feed any audio to a pedalboard or PA. The TRRS handles the monitoring; the quarter-inch output is unused.

The input-only variant fits comfortably between the A and C variants on the cost axis. The mezzanine has the codec (so quality is real) and one quarter-inch input jack with proper instrument-level analog circuitry, but no quarter-inch output jack and no output amplifier. The bill of materials saves maybe two to four dollars compared to a C1 mezzanine, depending on the specific parts.

### Why an Output-Only Variant Might Also Make Sense

By symmetry, an output-only variant is also worth considering. The use cases:

A monitor amp module that drives a headphone or line output but does not capture any audio. The module receives audio from the inter-module bus and renders it through a quarter-inch output or the TRRS headphone jack. No input jack needed.

A synthesizer module that generates audio internally (DX7 emulation, Hammond emulation, whatever) and outputs to a quarter-inch jack for the user's amp or PA. The MIDI input comes from a USB MIDI controller or from another module over the inter-module bus, so no audio input is needed.

A PA driver module at the end of the chain that takes the fully-processed audio from upstream modules and produces the final output to the user's PA or recording interface.

The output-only variant skips the input side of the codec (or uses a DAC-only chip instead of a full codec) and has only the output analog circuitry. The bill of materials saves a different two to four dollars compared to the C1.

### The Variant Matrix Under This Scheme

Putting it all together, the proposed variant lineup becomes:

A: Controller-grade. TRRS only. No quarter-inch jacks. Cheap motherboard configuration with basic DAC and microphone preamp.

B-input: Input-only audio. Quarter-inch instrument input plus TRRS plus inter-module routing. No quarter-inch output. Audio-grade input through codec ADC.

B-output: Output-only audio. Quarter-inch output plus TRRS plus inter-module routing. No quarter-inch input. Audio-grade output through codec DAC (or dedicated audio DAC chip).

C1: Full-duplex audio. One combo jack supporting input or microphone. Quarter-inch output. TRRS. Inter-module routing. Audio-grade in both directions.

C2: Full-duplex audio. Two combo jacks. Quarter-inch outputs. TRRS. Inter-module routing. Audio-grade in both directions across two channels.

The mezzanine for B-input is the simplest of the audio-capable mezzanines. The mezzanines for C1 and C2 are the most capable. B-output sits between them in complexity.

### Concerns and Counterarguments

The most obvious concern is permutation explosion. The current A/B1/B2 lineup is already three variants. Going to A/B-input/B-output/C1/C2 is five variants, with potentially more if we add C-input-only and C-output-only specialized versions. This is a lot of products to manufacture, document, support, and explain to users.

The mitigating factor is that all the variants share the same motherboard PCB design. The variant difference is captured at the mezzanine level, with each mezzanine being a separate small PCB with its own components. Adding a new mezzanine variant is a real engineering effort but not enormous; the mezzanines are small simple boards. Manufacturing handles them through the existing castellated attachment process.

The other concern is user comprehension. The current A/B1/B2 naming was already pushing at the limits of what users will tolerate before consulting a feature matrix. A/B-input/B-output/C1/C2 is worse. Users will reasonably ask why the platform has so many variants and what the difference is, and the answer is more complicated than it needs to be for most users.

A simpler approach would be to keep the variant naming as is (A, B1, B2) and treat the input-only and output-only configurations as board-options on the existing variants, similar to how the microphone preamp is currently a board-option on the B variants. The user buys a B1 and the manufacturer populates the mezzanine with or without certain components based on the specific configuration. This avoids variant proliferation while still supporting the use cases.

But the board-option approach has its own problems. The user does not know what they are getting unless they look carefully at the specific manufacturer's variant. Two B1 modules from different manufacturers might have meaningfully different capabilities. This breaks the "B1 means a specific thing" promise of the variant naming.

### The Rebranding Question Specifically

The proposed rebranding of B to C is more contentious than the input-only variant itself. The argument for rebranding is that "B" is currently overloaded with "input-and-output audio capability" specifically, and introducing an input-only variant under the same B name would be confusing. Renaming to C for the full-duplex variants and using B for the input-only variants is more orderly.

The argument against rebranding is that we have already used B1 and B2 in the existing documents, and changing the naming would invalidate that vocabulary. Users who have read the current documents would have to relearn the names. Marketing material in development would have to be reworked. The cost of changing the naming is real and the benefit is mostly cosmetic.

A compromise that preserves the existing naming would be to introduce the input-only variant under a separate naming convention entirely, perhaps as "B-Capture" or "B-Input" rather than as a numbered variant in the same family. This signals that it is a specialized variant rather than a step in the main product line. The output-only variant similarly becomes "B-Render" or "B-Output." Users learn the variant names as: A is the cheap controller, B1 and B2 are the audio modules, B-Capture is the specialized input-only variant, B-Render is the specialized output-only variant.

This is probably the right answer. Keep the current naming, treat the directional variants as named specializations rather than as part of the main numbered sequence.

### Decision and Next Steps

Not yet decided. This document captures the thinking for future review.

If we do introduce the input-only and output-only variants, the path of least disruption is probably:

Keep A as is.
Keep B1 and B2 as the full-duplex audio variants (no rebrand).
Add B-Capture as an input-only variant (or whatever name we settle on).
Add B-Render as an output-only variant if the use cases justify the additional product.

The vision and technical documents would need light edits to mention these specialized variants in the module catalog and to acknowledge them in relevant sections, but the bulk of the existing content would remain valid.

If we decide the rebranding is worth doing despite the cost, the path is:

Rebrand B1 and B2 as C1 and C2 throughout both documents.
Introduce B-input and B-output as new variants with the appropriate mezzanines.
Update the module catalog and version one scope to reflect the new naming.

This is more disruptive but produces a cleaner naming convention going forward.

Either path is defensible. The decision depends on how much the project values naming cleanliness against the cost of changing established vocabulary.

## Graphics-Accelerator Display Breakout Peripheral (Largely Displaced by Web Platform)

Earlier thinking treated a graphics-accelerator display breakout as a natural future peripheral that would consume a custom data-channel stream and render external displays through chips like the Bridgetek BT815 family. The web platform approach to the presentation protocol displaces most of the use cases for this peripheral, since any cheap computing device that has a browser and a video output can serve the same role.

The use cases the peripheral was meant to serve are now better served by commodity hardware:

A sound mixer position display showing the mixer UI on a passive monitor: any Raspberry Pi or similar single-board computer with HDMI output, running a browser pointed at the rig's web UI. Cost is $50-100 for the SBC plus a cheap HDMI monitor. The setup is "plug in HDMI, plug in power, open browser to the rig's address."

Audience-facing visualizations showing chain audio activity: same single-board computer with HDMI output, plus a custom web page that subscribes to the WebSocket data streams from the rig and renders custom visualizations using Canvas, WebGL, or any of the many web-based audio visualization libraries (Three.js, p5.js, Tone.js for audio analysis). The platform inherits the web visualization ecosystem essentially for free.

A band-rehearsal-space display showing preset, tempo, transposition, lyrics, chord charts: a tablet propped on a stand running the rig's web UI in a dedicated view mode. No additional hardware needed beyond what bands already have.

A stage information display: same pattern. Browser pointed at the rig's web UI, in whatever view mode shows the relevant production information.

The remaining use case where a dedicated graphics-accelerator breakout might still make sense is one where minimal additional hardware is preferred over a full single-board computer. Some users might prefer "small dedicated peripheral that does one thing well" over "small computer running a web browser." For these users, a graphics-accelerator-based peripheral could be designed that hosts a tiny WebKit instance and renders the rig's web UI to an HDMI output, packaged as a small box with HDMI out and power in. This is essentially a "Metapedal display dongle" similar in spirit to a Chromecast or a small smart TV box, dedicated to the rig's UI. Cost might be similar to a Pi Zero 2 W ($15-25) but with a more polished product experience.

For v1 scope, this is firmly outside what needs to ship. The web-platform approach means users can drive external displays today using commodity hardware they already own; a dedicated peripheral can come later if there is demand for a more polished form factor. The architectural commitment from v1 that supports any future variation here is just that the web UI works well, which is already a v1 commitment.

The FTDI EVE chips remain interesting for other potential uses where browser-based rendering is not appropriate (very low-power passive displays, scenarios where booting a browser is overkill), but they are no longer central to the presentation architecture.

## Splitter Mezzanine for Fat DSP and Other Expansion

A useful architectural insight came late in v1 design: the 40-pin mezzanine connector has secondary I2S data lines (SDI2/SDO2) allocated specifically for future expansion. These lines enable a "splitter mezzanine" approach that turns the single 40-pin connector into a bus capable of hosting two sub-mezzanines simultaneously, without requiring any motherboard redesign.

The platform's approach to fat DSP capability is twofold: a BX variant that ships as a finished product for users who want fat DSP in a single box, and a published mezzanine bus specification that lets third parties build splitter mezzanines and sub-mezzanines for users with specific needs not served by standard variants.

### The BX Variant as a v1 Product

The BX variant is a B mezzanine that includes both the standard audio codec and a dedicated fat DSP chip on the same mezzanine board. From the user's perspective, BX is just another variant in the family: they buy a BX module the way they buy a B1 or B2 module. Plug-and-play, single enclosure, no special considerations.

The BX mezzanine combines the standard B1 or B2 audio codec circuitry (one combo jack input plus output, or two combo jacks), a fat DSP chip alongside the codec, I2S routing between the codec, fat DSP, and motherboard MCU, power circuitry for both chips, and I2C control plumbing for motherboard configuration.

**The fat DSP chip choice** for BX leans toward a SHARC DSP from Analog Devices (ADSP-21489 or similar) at roughly $20-30 in production volumes. SHARC is the established audio DSP technology with a mature development ecosystem and meaningfully better audio performance per dollar than the H7 (~2.7 GFLOPS of single-precision floating-point versus the H7's lower FPU throughput). The tools and libraries for SHARC audio development are well-understood and have been in use across the audio industry for decades.

The alternative would be a more capable MCU like the Renesas RA8P1 with its Ethos-U55 NPU, which adds neural processing capability that SHARC does not have. This is more interesting strategically for users specifically wanting neural amp modeling, but represents more bet-the-product risk because the toolchain for targeting the NPU effectively (especially from Faust or other audio DSP frameworks) is less mature than the SHARC ecosystem. SHARC is the safer bet for the platform's first fat-DSP product; the NPU path can come later as a second BX variant if user demand justifies it.

**BX cost picture:**
- Standard B1 or B2 mezzanine BOM components: ~$15-20
- SHARC fat DSP chip: ~$25
- Additional power circuitry: ~$3
- Additional I2S routing and bypass switches: ~$3
- Larger PCB for the additional silicon: ~$2 incremental
- **BX mezzanine BOM total: ~$48-53**

BX module retail (motherboard + BX mezzanine + enclosure): ~$140-170, versus standard B1/B2 at ~$80-95. The BX premium of $60-90 reflects the substantial additional capability.

**BX firmware integration** is moderate complexity. The motherboard's H7 manages both the codec and the fat DSP through standard I2C control. The fat DSP runs its own program loaded from microSD or over the network. Audio routes between codec, fat DSP, and the inter-module bus through the motherboard. The platform's existing Faust-based DSP toolchain compiles to the H7 or to the SHARC depending on which chip should run a given algorithm; the user does not need to know which.

### The Mezzanine Bus Specification

Alongside BX as a finished product, the platform publishes the mezzanine bus specification. This is the contract that lets third parties build splitter mezzanines, sub-mezzanines, and arbitrary expansion hardware that works with v1 motherboards. The specification is the platform's commitment to expandability; hackers and third-party vendors do the actual innovation.

**What the bus specification covers:**

**Physical interface.** The 40-pin connector pinout (already specified for v1), the mezzanine mechanical envelope (dimensions, screw hole positions, height clearance), and orientation conventions for sub-mezzanines.

**Pin allocation rules for multi-mezzanine configurations.** How sub-mezzanines share the I2S streams (primary for one, secondary for another), how the I2C address space is partitioned (different mezzanine categories get different address ranges to prevent conflicts), how the SPI chip-selects are multiplexed, how GPIOs are allocated.

**Power distribution.** Current limits per sub-mezzanine, which rails are available, how splitters distribute power, what happens when sub-mezzanines exceed their current budgets.

**Identification protocol.** The EEPROM or small I2C device on each mezzanine reports vendor ID, product ID, version, audio I/O channel counts, and control channel definitions. The motherboard reads this on boot to discover what is plugged in and configure firmware appropriately.

**Audio routing protocol.** How splitters indicate which sub-mezzanine uses primary I2S and which uses secondary I2S, how channel counts on each stream are negotiated, how sample rates are coordinated across the configuration.

**Control protocol.** How sub-mezzanines expose their configurable parameters through the platform's standard interfaces, the message formats, how the motherboard relays web-UI controls to the sub-mezzanines.

**Mechanical constraints.** Which enclosures are appropriate for splitter configurations (1590BB and 1590DD for stacked geometry), stress isolation considerations, connector reliability specifications.

**Compliance testing.** A reference design third parties can build against, plus a test suite vendors can use to verify their mezzanines work correctly.

**The specification is the platform's commitment**; the actual splitter mezzanines and exotic sub-mezzanines are products that come from the community.

### What This Enables Without the Platform Building It

The published bus specification enables third parties to build:

**Splitter mezzanines** that turn one motherboard connection into two sub-mezzanine slots. Users wanting custom configurations buy a splitter plus two appropriate sub-mezzanines from any vendor.

**Specialty fat DSP mezzanines** beyond what BX provides. Hackers can build mezzanines with Renesas NPU silicon, FPGA-based custom DSP, or other specialty silicon that the platform itself does not commit to producing.

**CV/gate mezzanines** for modular synth integration. Eurorack users build mezzanines with CV inputs and outputs at appropriate voltage levels and impedances, plus gate/trigger inputs and outputs. The platform's GPIO and analog I/O on the connector are exactly what these need.

**Specialty audio I/O mezzanines.** XLR connector mezzanines for users wanting professional balanced audio, mezzanines with specific input impedance matching for vintage microphones, mezzanines with built-in tube preamps for users wanting analog warmth before the digital conversion.

**Prototyping mezzanines** for hobbyists experimenting with custom circuits. A breakout board exposing the 40-pin signals to standard headers, for users developing their own designs.

**MIDI mezzanines** with 5-pin DIN MIDI in and out, for users with legacy MIDI gear that does not work over USB or TRRS.

**Wireless mezzanines** with Bluetooth, Wi-Fi, or other wireless audio capabilities, for users wanting to free their pedalboard from cables.

The platform builds none of these directly. The community builds whatever the community wants, knowing that conforming to the spec means their mezzanines work on any v1 motherboard with the standard firmware.

### The Firmware Commitment

The v1 motherboard firmware needs to support multi-mezzanine configurations even though the platform ships only single-mezzanine variants in v1. This is a real commitment because firmware that discovers and handles arbitrary sub-mezzanine combinations is more complex than firmware for the platform's specific variants.

The required firmware capabilities:
- Reading EEPROM identification from any mezzanine on boot
- Discovering whether the connected mezzanine is a single mezzanine or a splitter with sub-mezzanines
- Configuring I2S routing based on what is discovered
- Detecting and reporting I2C address conflicts if sub-mezzanines collide
- Exposing the sub-mezzanines' capabilities through the platform's control interfaces
- Updating the routing graph and DSP scheduling to account for additional sub-mezzanines

This firmware ships with v1. Users who only ever use standard A or B variants never see this complexity; it operates entirely behind the scenes for them. Users who use splitter mezzanines benefit from the firmware support without the platform having to ship custom firmware for each splitter configuration.

### The Platform's "Hackers Gonna Hack" Commitment

The deeper principle is that the platform respects users who want to extend it. Rather than locking users into specific variants the platform produces, the bus specification opens the door to third-party innovation. Users with specific needs design their own mezzanines or buy from vendors who build for those niches. The platform's commitment is to maintain the bus contract over time, not to anticipate every possible mezzanine.

This is genuinely valuable for the platform's long-term health. The community builds modules the platform team would never have prioritized; users get capabilities the platform team would never have predicted; the platform's ecosystem becomes richer than any single vendor's roadmap could deliver.

The same principle applied to Eurorack synthesis (where the standard is just connector dimensions, voltage ranges, and signal conventions, and the ecosystem built thousands of modules), to USB peripherals (where the standard is just bus protocol and class definitions, and the ecosystem built every imaginable device), to Raspberry Pi HATs (where the standard is just connector pinout, and the ecosystem built sensor breakouts, display drivers, AI accelerators, and a thousand other things). Metapedal's mezzanine bus specification follows the same pattern.

### v1 Commitment Summary

The platform's v1 commitments around mezzanines:

**Ships as v1 products:** A variant (no mezzanine, audio components on motherboard), B1 variant (one combo jack mezzanine), B2 variant (two combo jack mezzanine), BX variant (B1 or B2 with fat DSP on the same mezzanine).

**Published as v1 specifications:** The mezzanine bus specification, including pin allocation, identification protocol, control protocol, and compliance testing requirements.

**Firmware committed to v1:** Multi-mezzanine support that handles splitter configurations and any conforming sub-mezzanines.

**Expected from third parties:** Splitter mezzanines, specialty sub-mezzanines, CV/gate mezzanines, MIDI mezzanines, and other expansion hardware that the platform team does not ship as products.

This division gives mainstream users plug-and-play products while giving the hacker community everything they need to extend the platform indefinitely.

## Digital Audio I/O and the S/PDIF Lesson

A specific specialty mezzanine worth thinking about is digital audio I/O: S/PDIF (both coaxial and optical TOSLINK), AES3, possibly ADAT or MADI for users who need these interfaces. This category serves the use case of connecting Metapedal to external digital audio gear (CD players, DAT machines, soundbars, game consoles, professional audio gear) without going through analog conversion at the boundary.

This is a natural fit for a third-party specialty mezzanine. The platform itself does not need to ship it; the mezzanine bus specification gives someone else everything they need to build it.

### The S/PDIF Insight Worth Capturing

There is a useful architectural lesson buried in S/PDIF history that informs how the platform thinks about interfaces. S/PDIF was defined in 1985 (IEC 60958), making it older than USB, older than consumer Ethernet, older than the Web. TOSLINK (the optical variant) was developed by Toshiba in 1983 specifically for CD player audio output. These are genuinely old standards that have persisted essentially unchanged for four decades in consumer audio.

The surprising part is why TOSLINK is optical. The intuitive assumption is "optical = bandwidth." This intuition comes from fiber optic communications where single-mode fiber carries terabits per second; people hear "optical audio cable" and assume optical fiber is needed for bandwidth reasons.

This is completely backward for TOSLINK. The actual bandwidth requirement for stereo 24-bit 192 kHz S/PDIF is roughly 24.576 Mbps at the optical layer (12.288 Mbps audio rate doubled by biphase-mark encoding). The 1980s plastic optical fiber used in TOSLINK supports tens of megabits per second; modern POF can do hundreds of megabits; glass fiber can do gigabits. TOSLINK uses a tiny fraction of optical fiber's capability.

The actual reason TOSLINK is optical is **electrical isolation**. Consumer audio gear in the 1980s had real problems with ground loops. Connecting two pieces of gear with shared ground references through a coaxial cable created paths for ground potential differences to flow, producing audible hum and noise at 60 Hz (or 50 Hz in Europe) plus harmonics. Optical isolation breaks the electrical connection entirely. The fiber carries light, not current. No ground loop possible because there is no ground connection. The receiving device sees the optical signal, converts it back to electrical inside its own grounded chassis, and the original device's ground is irrelevant.

This is why TOSLINK won over coaxial S/PDIF for consumer applications. Coaxial S/PDIF works fine technically (slightly better signal-to-noise ratio than TOSLINK because the optical conversion adds some noise), but it requires the user to think about grounding. TOSLINK just works regardless of how the user's gear is grounded.

### The Lesson for Platform Thinking

The "optical = bandwidth" assumption leads people to misunderstand interfaces. The right question when you see an optical digital interface is "what is the optical there to solve?" Sometimes it is bandwidth (datacenter fiber, optical Ethernet at 10+ Gbps). Sometimes it is isolation (TOSLINK, certain medical equipment, some industrial control). The two motivations point at completely different design considerations.

For Metapedal, the platform's internal mezzanine bus is electrical because everything is on the same module's ground reference. The platform's external interfaces (USB inter-module, M8 peripheral, TRS audio jacks) handle electrical isolation through other means: USB isolation transformers on the inter-module connections, careful grounding on the audio jacks, isolation where appropriate on peripherals. The platform achieves the same isolation goal that TOSLINK achieves, just through different mechanisms.

### A Digital Audio I/O Mezzanine

If a third party builds a digital audio I/O mezzanine, the design considerations:

**Coaxial S/PDIF**: small differential transformer for galvanic isolation, RCA jack for the standard connector. The coaxial connection works fine technically and has slightly better signal characteristics than optical, but requires the user to understand grounding.

**Optical TOSLINK**: F05 connector with integrated transmitter/receiver modules (parts like the Toshiba TORX/TOTX series, ~$2-5 each in production). The optical conversion adds slight latency (the LED/photodiode transition time is microseconds, negligible for audio) and very slightly degrades signal-to-noise ratio (also negligible). But the electrical isolation is complete.

**AES3 / AES/EBU**: balanced differential pair through XLR connector for professional applications. Higher signal levels than S/PDIF, longer cable runs supported, better noise immunity. The pro audio standard.

**ADAT**: optical interface for 8-channel digital audio over a single TOSLINK fiber. Used in pro audio gear from the 1990s and still common in studios. The same TOSLINK physical layer as S/PDIF but a different protocol carrying more channels.

The mezzanine takes I2S audio from the bus through the lease protocol, converts to the appropriate digital audio format using small CPLD or dedicated converter chips, and outputs through the standard connectors. The mezzanine bus does not need to natively carry S/PDIF or AES3; it just needs to deliver the I2S audio that the mezzanine converts.

This is the kind of specialty mezzanine the platform's bus specification was designed to enable. The platform itself does not ship it; someone in the third-party ecosystem builds it when there is enough user demand to justify the effort. The bus specification is rich enough that this mezzanine is straightforwardly buildable against the published contract.

### v1 Reserved Pins Do Not Need S/PDIF Allocation

The 80-pin connector specification reserves 30 pins for future protocol extensions. None of these need to be specifically allocated for S/PDIF or related digital audio interfaces. The existing I2S, SPI, and GPIO buses are sufficient for a specialty mezzanine to implement S/PDIF locally. The reserved pins should stay genuinely reserved for protocol extensions we cannot currently predict.

## The Robot Finger Peripheral: Bridging Digital Control to Analog Gear

A specific peripheral concept worth capturing for future development is what we are calling the "robot finger": a small motorized peripheral that physically actuates controls on existing analog gear. The motivating use case is users who own beloved analog pedals (vintage Tube Screamers, hand-built fuzz pedals, treasured compressors, whatever) and want the platform's digital control plane to reach into those pedals without modifying the pedals themselves.

The concept takes inspiration from the Useless Machine, the iconic novelty toy where flipping a switch causes a small mechanical finger to emerge from the box and flip the switch back off. The toy is itself a piece of folk engineering that demonstrates a simple idea (a mechanism that responds to physical input with physical output) in a charming and memorable way. The robot finger peripheral takes the same idea seriously: instead of returning a switch to off, the finger turns a knob or flips a switch in response to control signals from the Metapedal routing graph.

### Why This Matters

The platform's "you do not have to adopt the whole thing" positioning depends on Metapedal integrating with existing gear rather than replacing it. Most existing gear has MIDI for control, and Metapedal modules can drive MIDI directly through their TRRS jacks or peripheral ports. But a large category of beloved analog pedals predate widespread MIDI adoption and have no electronic control inputs at all. The only way to change parameters on these pedals is to physically turn the knob or flip the switch.

The robot finger bridges this gap. A user with a treasured analog overdrive can put a robot finger on the drive knob, configure the Metapedal routing graph to send control signals from a footswitch or sensor to the robot finger, and now their analog pedal's drive parameter is software-controlled. The pedal itself is not modified; the robot finger sits on top of the existing knob and turns it through small motorized motion.

### Implementation Concept

The robot finger is a small enclosure (perhaps the size of a couple of stacked coins) that attaches to an existing analog pedal through some mechanical means: adhesive, a small clamp, a magnetic mount for pedals with steel enclosures, or a custom bracket for specific pedal form factors. Inside the enclosure is a small servo or stepper motor, a tiny microcontroller, and the M8 connection back to the host Metapedal motherboard.

The servo or stepper drives a small mechanical interface that engages with the analog pedal's control. For a knob: a soft rubber or silicone disc that presses lightly against the knob and rotates it through friction, with enough grip to turn the knob and enough slip to handle the user manually turning the knob without damaging the actuator. For a switch: a small lever that pushes the switch up or down (or stomps the footswitch, for floor-based controls). For a toggle: a small bar that flips the toggle to its other position.

The peripheral communicates with the host module through I2C over the M8 Smart Digital connection. The host module sends "set position to N" commands; the robot finger's microcontroller drives the motor to the requested position. Position calibration happens at setup time, where the user manually rotates the knob to its minimum and maximum, the robot finger records the corresponding motor positions, and subsequent control commands interpolate within that calibrated range.

### The Range of Form Factors

Different analog pedals have different control geometries. The robot finger concept probably needs a small family of variants to cover the common cases:

**Knob variant.** The most common case. A rotational actuator that drives a knob through friction. Various knob diameters need different actuator sizes; a small family (maybe small, medium, large) covers most pedal knobs.

**Toggle switch variant.** A small mechanism that flips a toggle switch between two positions. The actuator is binary rather than continuous, which simplifies the mechanical design.

**Slide switch variant.** A small mechanism for slider switches, which require linear motion rather than rotational or pivoting motion.

**Footswitch variant.** A larger mechanism for the stompbox footswitch itself, which requires more force and travel than the knob or toggle variants. The footswitch variant would be a small motorized stomper that lives on top of the pedal and presses the switch down on command. This variant is the closest in spirit to the original Useless Machine.

Each variant has its own engineering challenges, and the user might own different pedals that need different variants. The Metapedal peripheral catalog could include the most common ones (knob and footswitch are probably the highest-priority); the others come along as the community builds them.

### Engineering Considerations

The robot finger is genuinely a small mechanical engineering project. Key concerns:

**Mechanical reliability.** The actuator engages with the pedal's control mechanism repeatedly over the pedal's lifetime. The interface needs to hold up under normal use without slipping, without damaging the original control, and without becoming unreliable as the rubber friction surfaces age.

**Mounting stability.** The robot finger has to stay attached to the analog pedal during a chaotic performance. Adhesive mounts work for some pedals but not all; clamp mounts are more universal but more fiddly. The mounting solution may be the hardest part of the engineering.

**Power consumption.** The actuator motors draw current. The host Metapedal module's M8 port provides limited current; high-torque actuators may exceed the budget. Either the robot finger uses low-current actuators with appropriate gear ratios, or it has its own power input separate from the M8 connection.

**Acoustic noise.** Servo motors and stepper motors make audible noise when they move. For pedals that are being used during recording, the actuator noise could bleed into microphones. Low-noise actuators (smooth-running steppers, fluid-damped servos) cost more but eliminate this concern.

**User override.** A user touching the knob while the robot finger is also driving it should not damage anything and should produce a sensible behavior. The simplest design lets the user manually override the position; the robot finger detects the discrepancy at its next update and either accepts the new position or drives back to the commanded position depending on configuration.

### Strategic Value

The robot finger peripheral is not necessary for v1 and probably should not be in the v1 product line. But the concept is worth capturing because it represents a specific bridge between the platform's digital control plane and the analog gear that musicians actually own and love. The "you do not have to adopt the whole thing" positioning depends on these kinds of bridges existing; the robot finger is one of the most evocative examples.

The third-party ecosystem can develop these. A maker with a 3D printer, some small servos, and a basic microcontroller can prototype a robot finger for a specific pedal in an afternoon. The Metapedal peripheral protocol gives them the host-side interface they need; the mechanical work is bounded and well within hobbyist capability. A community of makers building robot fingers for different pedals would be exactly the kind of ecosystem the platform is designed to enable.

The Useless Machine framing is also charming in a way that fits the platform's voice. The original toy is a piece of folk engineering that demonstrates a simple idea with humor and precision; the robot finger inherits that spirit while applying it to a genuinely useful problem. Marketing the peripheral with light Useless Machine references would land well with the platform's audience.

## The Hybrid Analog-Digital Studio Use Case

A specific configuration worth capturing because it demonstrates the platform's partial-adoption story particularly well: the B variant module as a software-controllable analog send and return loop.

The configuration is simple. A B variant module's DAC drives audio out through its analog output. That signal flows through one or more external analog pedals (a beloved analog overdrive, a vintage compressor, a hand-built filter, a short chain of such pedals). The chain's output flows back into the B variant module's analog input. The module's ADC digitizes the analog-processed audio and returns it to the routing graph. The onboard DSP is entirely uninvolved; the module is functioning purely as a digital-to-analog-to-digital round trip.

### Why This Is Useful

Several substantive benefits emerge from this configuration:

**Beloved analog pedals at any point in the chain.** The user's treasured analog gear effectively lives inside the digital routing graph at any point the routing specifies. The same overdrive pedal can appear before reverb in one preset, after reverb in another preset, and bypassed entirely in a third preset, all controlled through software rather than through cable repatching.

**Multiple analog chains as composable elements.** A short chain of analog pedals (compressor into overdrive into specific filter) becomes a single addressable element in the user's mental model. The routing graph treats this analog chain like any other audio processing block, sending signals to its input and consuming signals from its output. Multiple such analog chains can coexist, each accessible from the routing graph independently.

**Parallel analog and digital processing.** A signal can be sent through both an analog round-trip and a parallel digital effects chain simultaneously, with software-controlled wet-dry mixing between the two. The analog character contributes to the sound without dominating it; the wet-dry mix is determined by the routing graph.

**Software bypass of analog gear.** Analog pedals that lack true-bypass or remote-bypass capability become effectively bypass-able through the routing graph: the routing simply does not route signals through the B variant module hosting that pedal's send-return loop. The analog pedal does not need bypass support of its own.

**Pre-and-post recording.** The B variant module captures the post-analog-chain signal at its input. The user can record this captured signal alongside the dry pre-analog signal from earlier in the chain, getting both tracks simultaneously. This preserves the production option of re-amping or replacing the analog character later.

**A/B comparison between analog and digital paths.** The same source can be sent through an analog round-trip (via the B variant) and through a digital effect (via Metapedal DSP) simultaneously. The user can compare or blend the two paths to find what works best for their specific source material.

### The Combination With the Robot Finger

When combined with the robot finger peripheral (covered in the previous section), the analog gear becomes fully software-controllable. The routing graph determines whether the analog chain receives signal; the routing graph determines what parameter values the robot finger sets on the analog pedal's knob; the routing graph determines how the analog-processed output is mixed back into the chain. The user's analog gear becomes a fully integrated element of the digital signal flow, with its tonal character preserved but its control surface fully digital.

This is the closest thing to "best of both worlds" that the platform offers for users who genuinely love their analog gear. The analog signal path is preserved with all its character; the control over the analog gear is as flexible as anything purely digital.

### The Hybrid Studio Pain Point

The hybrid analog-digital studio has been a long-running pain point in audio production. Producers who own specific analog gear (specific outboard preamps, vintage compressors, specific overdrive pedals, hand-built filter banks) have wanted to integrate this gear with modern DAW workflows for decades. The pain point is that getting analog gear into and out of the DAW cleanly requires expensive multi-channel I/O and complex routing through audio interfaces, with all the analog-digital boundary management that implies.

Various commercial products attempt to address this: the Black Lion Audio Sparrow, certain Antelope Audio interfaces with bundled analog effects, the SSL pure-analog products. None are open hardware. None offer software-defined routing through arbitrary analog gear in the way Metapedal does.

A few Metapedal B variant modules, each hosting a piece of beloved analog gear in a send-return loop, give the user an elegant hybrid setup. The DAW sees the analog-processed results through the Metapedal modules functioning as USB audio interfaces. The user's analog gear lives at exactly the right point in the digital signal flow as configured by the routing graph. This is a real solution to a real problem that many producers have been working around for years.

### Implementation Notes

The configuration does not require new hardware or new firmware capabilities beyond what v1 already covers. A standard B1 or B2 module supports this use case out of the box; the user just routes signals through the module's audio path and configures the routing graph accordingly. The web platform's routing UI should make this configuration easy to set up, perhaps with a specific "analog send-return" template that users can apply to a module to configure it for this role.

One refinement worth thinking about for the analog send-return template: the user probably wants the option to compensate for the slight latency introduced by the DAC and ADC conversion (about 0.6 ms each, so roughly 1.2 ms round-trip in addition to the analog chain's own latency). For most uses this latency is inaudible; for production work where phase alignment matters, the routing graph could automatically delay parallel dry signals by the same amount to keep them aligned with the analog-processed wet signal.

### Why This Belongs in the Documentation Set

The hybrid analog-digital configuration is captured here in the TODO document because it represents a use case that deserves explicit treatment in the platform's marketing and onboarding materials but does not require any specific v1 product or feature work beyond what already exists. The platform's web UI should make this configuration easy to set up; the platform's marketing materials should feature this use case as an example of partial adoption that respects users' existing gear; the platform's example presets and tutorials should include "analog pedal send-return" as a standard pattern users can apply.

The hybrid configuration is also the natural pairing with the robot finger peripheral. The two ideas together (send-return loop plus parameter control via robot actuator) form a complete bridge between digital control planes and analog signal paths. Either idea alone is useful; together they are a meaningful contribution to the audio production toolkit.

## Audio-to-Control Extraction: The Algorithm Catalog

The platform's ability to analyze audio and produce control signals is mentioned in the vision document as a meaningful user-visible capability, but the specific algorithms and their DSP cost have not been fully scoped. This section captures the algorithm catalog and the scoping question for v1 versus v1.x firmware.

The architectural foundation is already in place. Every module's audio input feeds through the codec into the H7's audio processing pipeline. The routing graph handles control signals as first-class typed events. The only missing pieces are the specific analysis algorithms running on the H7's DSP capacity, the configuration UI for setting them up, and the control signal types that audio analysis produces.

### The Algorithm Catalog

**Envelope follower.** Continuous output tracking the smoothed amplitude of the audio input. DSP cost: trivial, a few dozen cycles per sample. Output: a control signal in the 0.0 to 1.0 range updated at audio rate or downsampled to control rate. Use cases: dynamics-responsive parameters, ducking, automatic gain adjustment.

**Transient detector.** Discrete trigger events emitted when the audio crosses a configured threshold with appropriate hysteresis and refractory periods. DSP cost: low, a few hundred cycles per sample. Output: trigger events with optional velocity information based on peak amplitude. Use cases: kick drum triggers, hand clap detection, transient-driven step sequencers.

**Voice activity detector.** Binary output indicating whether the audio contains speech/singing or just noise/silence. DSP cost: low to moderate, often combining envelope and spectral features. Output: a binary control signal with hysteresis to prevent rapid toggling. Use cases: automatic muting of channels during quiet sections, automatic engagement of vocal effects, communication-system style automatic gain.

**Pitch detector.** Continuous output of the fundamental frequency of the audio input, with confidence indication. DSP cost: moderate, a few thousand cycles per sample for a competent autocorrelation-based detector running at audio rate. Output: a control signal carrying frequency in Hz (or note number with cents detuning) plus a confidence value. Use cases: filter that tracks vocal pitch, auto-harmonizer that produces fixed intervals, synth voice that doubles a melodic line.

**Beat tracker.** Periodic output of tempo estimate plus phase information indicating where downbeats fall. DSP cost: moderate per analysis frame but low per audio sample (frames happen 10-100 times per second). Output: tempo in BPM, phase position within the measure, confidence values. Use cases: tempo-locked delays, beat-synced arpeggiators, automatic tempo matching for backing tracks.

**Spectral feature extractor.** Continuous output of various spectral characteristics: low/mid/high band energy, spectral centroid (perceptual brightness), spectral flux (rate of spectral change), spectral entropy (signal complexity). DSP cost: moderate, dominated by the FFT (a few thousand cycles per FFT frame, frames at 100-1000 Hz). Output: a vector of control signals, one per spectral feature. Use cases: EQ that responds to band density, reverb that adapts to spectral context, dynamics that responds to perceptual brightness.

**Room ambient analyzer.** Combined analysis of overall amplitude plus spectral features plus voice activity to characterize the room context: how loud is the room, how busy is the spectral content, is anyone speaking/singing or is the room quiet. DSP cost: moderate, basically the sum of the underlying analyses. Output: a small vector of control signals characterizing the room. Use cases: parameters that respond to performance context.

**Onset detector.** Discrete events emitted when new sound events begin (different from pure transient detection because onsets can be in the middle of ongoing sound). DSP cost: moderate, requires spectral flux analysis. Output: onset events with timing precision better than transient detection alone. Use cases: tighter beat tracking, more precise tempo-locked effects, music information retrieval applications.

**Chord recognizer.** Discrete events emitted when chord changes are detected, plus continuous output of detected chord identity. DSP cost: moderate to high, requires chromagram analysis. Output: chord identity (root note plus quality) plus change events. Use cases: harmonically-aware effects, automatic accompaniment, key-detection for melodic algorithms.

### DSP Cost Headroom

The H7 has comfortable DSP headroom for several analysis algorithms running alongside normal effects processing. A typical B variant module configured for guitar pedalboard use (a few effects in series, perhaps a delay and reverb plus modest distortion) uses perhaps 30-50% of the H7's DSP budget. The remaining headroom comfortably supports several analyses: envelope follower plus transient detector plus voice activity detector use perhaps 10-15% of the H7. Adding a pitch detector or beat tracker brings the total to maybe 50-70%, still comfortable.

The BX variant with its SHARC chip has substantially more headroom. The SHARC can run heavier algorithms (full beat trackers with high accuracy, complex pitch detection with low latency, FFT-based spectral analysis at high frame rates) without affecting the H7's capacity for effects processing.

### V1 Scoping

For v1, the recommended scope is the basic five algorithms: envelope follower, transient detector, voice activity detector, simple pitch detector, basic beat tracker. These cover the most common user-visible use cases and have well-understood implementations. The DSP cost is comfortable; the user-facing value is high.

V1.x can add the more sophisticated algorithms: high-quality pitch detector, full beat tracker with downbeat detection, spectral feature extractor, onset detector. These benefit from BX-class compute or from H7 modules dedicated to analysis rather than effects.

V2 territory: chord recognition, full music information retrieval, machine-learning-based source separation, advanced room analysis. These probably need substantially more compute than v1 silicon provides.

### The User-Facing Configuration

The web platform's routing graph editor needs to expose audio-to-control extraction as a configurable element. Users add an "audio analyzer" node to the graph, configure which algorithms it runs (checkbox list with parameter tweaks), and connect its outputs (envelope, triggers, pitch, tempo, etc.) to parameters on other modules. The configuration UI should make this approachable for non-technical users who do not want to think about FFT sizes or analysis frame rates.

The default settings should produce useful behavior out of the box. A user who plugs in a microphone and selects "beat tracking" should get reasonable tempo extraction without further configuration. Advanced users can tweak thresholds, smoothing constants, and analysis parameters as needed.

### The "Listening Module" as a Specific Product

The audio-to-control capability suggests a specific product framing worth considering: a "Listening Module" or "Audio Sensor Module" that is configured by default as the ears of a rig. The module has a microphone input optimized for ambient capture, default analysis algorithms running, control signal outputs flowing to the routing graph. Users buy this module specifically to add audio-driven control to their existing rig.

This is essentially an A variant with default-on audio analysis, perhaps with a wider-pickup microphone preamp and clearer documentation about what the module is for. The hardware is identical to a standard A variant; the configuration and marketing are different. This is the kind of product variation that the platform's open architecture enables without requiring different hardware.

### Strategic Value

The audio-to-control capability is a real differentiator from traditional gear. The traditional separation between audio path and control path is something musicians experience as a limitation: they want their effects to respond to the music, but the gear cannot hear the music. Metapedal lets the gear listen, which sounds like science fiction to musicians who have only worked with traditional gear.

The marketing story writes itself. "Your delay locks to the drummer automatically." "Your reverb gets wetter when the singer goes quiet." "Your bass synth ducks under every kick hit without sidechain wiring." Each of these is concrete, evocative, and demonstrates a capability that traditional gear simply does not have.

Worth emphasizing this capability in the executive summary, the vision doc, the casual pitch, and any marketing materials. It is one of the platform's strongest user-facing stories and falls out naturally from the architecture without requiring new hardware development.

## Version Two: An Open Question

Earlier drafts of this document treated v2 as a scheduled successor to v1 with specific feature commitments. That framing was wrong. v2 should not be a scheduled future commitment but a response to potential future needs that the platform stays ready to address. The honest assessment of v1's capabilities suggests that most of the features that v2 would have added are not actually needed.

### Why v2 Is Less Compelling Than It First Appeared

Looking at v2 candidate features against actual user needs:

**Native gigabit ethernet.** v1 at 100 Mbps fits every realistic v1 use case with substantial headroom. The marginal latency improvement from gigabit (a few microseconds per hop) is below the perceptual threshold. The bandwidth ceiling matters only for installation-class configurations beyond v1's scope. Gigabit is a "nice to have" rather than a need.

**Native USB 3 with 5-10 Gbps inter-module bandwidth.** The original justification was supporting more channels at high quality tiers across long chains. v1 USB high-speed handles every realistic v1 use case. USB 3 silicon would be paying for capability that nobody is asking for.

**Direct DisplayPort output for module-driven displays.** Originally justified by an imagined "modules with built-in displays" use case. With the web platform decision, displays are external (laptops, tablets, USB-C touchscreen monitors, single-board computers with HDMI). Users already have display devices; modules do not need to drive video output. DisplayPort capability is a feature looking for a use case.

**Higher DSP capability for heavier effects per module.** The 60% performance improvement from Cortex-M85 with Helium is real but does not unlock fundamentally new capabilities. It moves the "comfortable single-module" boundary moderately. Users who hit single-module limits can buy more modules; the cost is comparable to a v2 premium.

**Neural amp modeling at commercial quality.** The most substantive v2 candidate. The Renesas RA8P1's NPU could deliver Neural DSP-class quality. But users who want commercial-quality neural amp modeling can already get it from existing products: Neural DSP plugins on a laptop, Quad Cortex hardware, Kemper hardware. Metapedal's value proposition is architectural (decoupled controls, software-defined routing, multi-channel recording, modular configuration), not "best neural amp model in a pedal." The platform competes by being substantially more flexible at modest cost, not by being best-in-class at any single capability.

**Embedded NVM with larger capacity.** The V8's 4 MB PCM versus H7's 2 MB Flash is real but not breaking v1.

### What This Means

v1 is the platform's main commitment for the foreseeable future. The platform should treat v1 not as "the first version of an evolving platform" but as "the standard architecture" that users can confidently invest in.

The implications of this framing are substantial and positive:

**Stable platform attracts more development.** Web frameworks like React and database stacks like Postgres became dominant partly through stability. The platform's open ecosystem benefits from architectural stability; firmware contributors, mezzanine designers, and effect developers know their work has years of relevance.

**User value goes up with platform stability.** A user buying v1 in 2026 knows they are buying the current platform, not an interim product. Resale value of v1 modules stays higher because they are not "old v1" versus "new v2." Users do not delay purchases waiting for the next generation.

**v2 becomes demand-driven rather than calendar-driven.** If users start asking for capabilities v1 cannot deliver, v2 development begins. Until that demand exists, v1 evolves through firmware updates and additional module variants. This is how Apple manages iPhone generations: ship when there are meaningful improvements, not on a fixed schedule.

**Manufacturing economics improve.** BOM costs drop as v1 production volume grows over multiple years. Component sourcing is easier when the platform commits to specific parts for years. Pricing can drop over time as production scales, rather than being reset with each generation.

### What Might Eventually Justify v2

v2 development should begin when one of these scenarios emerges:

**The H7 going end-of-life.** Microcontrollers stay in production for 10-15 years typically, so the H7 should be available for the lifetime of v1 plus several years. But eventually it will be discontinued, and v2 will need to handle the transition. This is a multi-year horizon at minimum.

**A genuinely new use case requiring capabilities v1 cannot provide.** Some capability that does not exist yet but might emerge. The platform should remain open to this possibility without trying to predict it. If a future user need cannot be served by additional modules or by alternative configurations of v1 hardware, v2 becomes the answer.

**A breakthrough that changes user expectations.** New wireless standards becoming ubiquitous, new audio compression algorithms enabling much higher channel counts at lower bandwidth, new neural processing techniques transforming the synthesis landscape. The platform should track these without committing prematurely.

**Substantially better silicon at the same price point.** If a new MCU appears with 5x the performance of the H7 at the same cost, the decision depends on what the additional performance enables. Better numbers alone do not justify disrupting v1.

### MCU Options When v2 Eventually Happens

When v2 development begins, the MCU landscape is much richer than when v1 was conceived. Several options that did not exist or were not mature when v1 was designed:

**STMicroelectronics STM32V8** (announced November 2025, OEM availability Q1 2026): Cortex-M85 at 800 MHz with Helium MVE, 4 MB embedded PCM, 1.5 MB ECC SRAM, native gigabit ethernet with TSN, USB HS/FS with PHYs included, 18nm FD-SOI process, 60% performance improvement over H7 in CoreMarks. The natural ST successor for v2 because it maintains continuity with the v1 ecosystem.

**NXP i.MX RT1170 family** (available since 2021): Cortex-M7 at 1 GHz plus secondary Cortex-M4 at 400 MHz, dual native gigabit ethernet with AVB and TSN, up to 2 MB SRAM. Higher clock speed than V8 on the main core; mature ecosystem with extensive third-party support.

**Renesas RA8P1** (available now): Cortex-M85 at 1 GHz plus secondary Cortex-M33 at 250 MHz, native gigabit ethernet, Ethos-U55 NPU for AI/ML acceleration. The NPU is unique among these options and would enable neural amp modeling at quality competitive with commercial products.

The choice between these depends on what v2's compelling driving feature actually turns out to be. The platform should not commit to a specific v2 MCU now; the decision happens when v2 development begins, based on what the platform has learned.

### Version Two Is Not a Promise

The most important framing change is that v2 is not a promise to users. Users buying v1 should not expect v2 in 18 months. The platform commits to v1 as the long-term standard, with firmware updates and additional module variants providing the evolution. If v2 happens, it will be because users are asking for capabilities v1 cannot deliver, not because the calendar demands it.

This framing protects both the platform and the users. The platform does not over-promise. Users do not feel obsolete. The architecture stays stable long enough for the ecosystem to mature around it.



**NXP i.MX RT1170 family** (available since 2021): Cortex-M7 at 1 GHz plus a secondary Cortex-M4 at 400 MHz, dual native gigabit ethernet ports with AVB and TSN, up to 2 MB SRAM, mature ecosystem. Higher clock speed than V8 (1 GHz vs 800 MHz) on the main core. The secondary M4 core could handle separate workloads like network protocols while the M7 focuses on DSP. The mature ecosystem is a real advantage; the chip has been in production for years with extensive third-party support, libraries, and reference designs.

The cost of switching from ST to NXP ecosystem is real (different IDE, different libraries, different vendor support culture) but the technical capabilities are competitive with or better than the V8. Pricing is in the $15-30 range in production volumes.

**Renesas RA8P1** (available now): Cortex-M85 at 1 GHz plus secondary Cortex-M33 at 250 MHz, native gigabit ethernet, Ethos-U55 NPU for AI/ML acceleration. The fastest Cortex-M85 currently shipping. The NPU is unique to this option and could provide hardware acceleration for neural amp modeling well beyond what the Helium MVE on V8 or the M7 on RT1170 can do. Available at $20-28 in small quantities.

The Renesas ecosystem is less familiar to ST developers but the e2 Studio IDE and Flexible Software Package are mature. The NPU advantage is meaningful if neural processing is a key v2 use case.

### System-on-Module Considerations

A SoM (System on Module) approach is worth thinking about for v2 even though it probably does not fit the standard pedalboard form factor. SoMs are small PCBs that integrate the MCU plus supporting circuitry (clock generation, power management, memory, sometimes ethernet PHY) into a replaceable assembly that mates with a carrier board through a standardized connector. The carrier board provides application-specific I/O, connectors, and physical form factor.

The SoM-plus-carrier pattern has substantial design advantages: certified SoMs simplify regulatory compliance, supply chain risk is reduced because the SoM vendor can substitute pin-compatible alternatives when specific parts go end-of-life, development velocity improves because carrier board iteration does not affect the complex high-speed digital portion, and manufacturing flexibility increases because low-volume products use commodity SoMs while higher-volume products integrate components directly.

The MCU SoM ecosystem is mature. Toradex Verdin modules (NXP i.MX 8M Mini, i.MX 8M Plus, i.MX 95) are roughly 55 by 38 mm. Variscite DART modules (25 by 50 mm or 55 by 30 mm). Octavo Systems OSD32MP1 integrates STM32MP1 plus DDR3 RAM plus power management plus oscillators into a single 18 by 18 mm BGA (closer to SiP than traditional SoM). For STM32V8 specifically, ST will likely produce official SoMs and third parties will follow.

### Why SoMs Do Not Fit the Standard Pedal Form Factor

The Hammond 1590B enclosure that Metapedal targets has internal PCB area of roughly 100 by 50 mm after accounting for clearance, mounting, and panel-extending connectors. A 55 by 38 mm SoM occupies a substantial fraction of this area. With the mezzanine connector plus the SoM connector plus the inter-module USB-C ports plus the M8 peripheral ports plus the TRRS jack plus the 9V power jack, the motherboard becomes geometrically constrained quickly. The SoM also adds vertical height (the SoM sits perhaps 5-10 mm above the motherboard), competing with the mezzanine for the enclosure's depth budget.

For standard A and B variants targeting pedalboard use cases, direct MCU integration on the motherboard is the right choice. The form factor and cost constraints both push against the SoM approach for the main product family.

### Where SoMs Might Make Sense

Several specific v2 module categories could justify the SoM approach:

A "Pro" module variant using a higher-end MCU (Renesas RA8P1 with its NPU for neural processing) might justify SoM because the Pro motherboard would benefit from the SoM's integration of complex high-speed routing. The Pro variant is bigger and more expensive anyway, so SoM area and cost overhead are more tolerable. The pro variant could live in a 1U rack enclosure or a larger pedal-format enclosure (Hammond 1590DD or similar) with the headroom to host a SoM comfortably.

A "Compact" module variant wanting to fit in smaller enclosures (Hammond 1590A or even smaller) might use SiP-style integration (OSD32MP1 pattern: MCU plus memory plus power management in one BGA) to save board area. The SiP approach avoids the connector overhead while keeping the integration benefits.

An "Installation" module class sharing components with industrial control systems might use a standard industrial SoM (Toradex Verdin or similar) to leverage industrial-grade certification, extended temperature operation, and supply chain stability that pro AV installations care about. The installation use case is less price-sensitive and more concerned with long-term reliability.

A SiP approach in general becomes more attractive as MCU complexity increases. The STM32V8's 18nm FD-SOI process and the high-pin-count BGA packages for high-performance variants are harder for small PCB manufacturers to handle than the H7. v2 and v3 might find SoM-based motherboards more practical for small-batch manufacturing because the complex high-speed work is encapsulated in the SoM.

### v1 Direct Integration

For v1 specifically, the H7 direct integration is the right answer. The H7 has been around for nearly a decade; small PCB manufacturers know how to handle its packaging and high-speed routing requirements. The platform's BOM is not gigantic. The motherboard area is geometrically constrained but workable. The MCU plus supporting circuitry (clock, power, USB hub for inter-module bus, USB PHY for OTG HS) takes maybe 30-40 percent of the motherboard area; the rest goes to the mezzanine connector, peripheral connectors, power circuitry, and other application-specific components. The direct integration is the cost-optimal and area-optimal choice for v1.

v2 will revisit the SoM question when more complex MCUs become candidates and when the platform has accumulated experience about which variant categories justify the SoM approach versus continued direct integration.

### Capability Menu for Eventual v2

If v2 development eventually begins, several specific capabilities are candidates. These are notes for future reference rather than a roadmap; the actual v2 scope will be determined by what user demand looks like when development begins.

**Native gigabit ethernet** through one of the MCU options that has it (V8, RT1170, RA8P1). Replaces v1's USB-bridged half-gigabit approach with line-rate gigabit, plus TSN support for AES67/AVB/Dante native operation. Most useful for installation-class configurations and serious pro AV applications.

**Native USB 3 SuperSpeed inter-module bandwidth** at 5-10 Gbps versus v1's 480 Mbps. Enables higher channel counts in chains and higher sample rates. Not currently bandwidth-limited in any realistic v1 use case, so this is genuinely "headroom for the future" rather than addressing a current pain point.

**Direct DisplayPort output** for module-driven displays. Most useful if users start asking for "plug HDMI directly from the module" rather than the v1 web-platform approach where displays are external. The web-platform decision means this is less compelling than it originally seemed.

**Neural amp modeling at commercial quality** through the Renesas RA8P1's NPU or comparable hardware. Most useful if users specifically want commercial-quality neural amp models in the Metapedal form factor rather than buying dedicated neural amp products.

**Higher-quality audio paths** like 96 kHz / 32-bit through the chain by default rather than the user-selectable tier approach v1 uses. Most useful if users find tier selection burdensome.

**Wireless capability integrated into standard modules** rather than as peripherals. Most useful if wireless audio quality improves enough to be reliable for live performance, which is not currently the case.

**Routing modules** as multi-port chain hubs for complex topologies. Most useful if v1 demonstrates that hand-built complex topologies (mesh, redundant rings) are something users want.

**More compute** through a successor MCU or additional dedicated DSP chips. Most useful if v1's compute budget becomes the binding constraint for use cases users actually want.

### When v2 Becomes Real

The platform team will know v2 is justified when users start describing specific use cases that v1 cannot serve, multiple users articulate similar unmet needs, and the unmet needs cluster around capabilities that v1's architecture genuinely cannot deliver through software updates or additional modules.

Until that happens, v2 stays as notes for future reference. v1 is the platform.

## The Web Platform Decision: Notes for Future Reference

Earlier drafts of this project thought about designing a custom imperative drawing protocol for the presentation layer, modeled after PostScript, Cairo, Skia, HTML5 Canvas, or OpenVG. That direction was retired in favor of adopting the web platform (HTTP, WebSocket, HTML/CSS/JavaScript, browser rendering on the consumer side) for the presentation protocol. The technical document captures the current architecture.

A few notes worth preserving from the earlier thinking, in case the web-platform decision needs revisiting:

The custom protocol design space was well-mapped. The historical references (PostScript, PDF imaging model, Cairo, Quartz 2D, Direct2D, Skia, HTML5 Canvas, SVG's drawing model, OpenVG) all operate at the same abstraction level and share the same primitives. Differences are in syntactic detail, transformation model, font handling, and breadth of features. The Metapedal protocol could borrow from any of them, or define its own minimal version.

The hard problems in custom protocol design are: path-based versus primitive-based commands (path-based is more flexible, primitive-based is more compact); transformation model (whether to support affine transforms with translation, rotation, scaling, skew); text rendering (essentially impossible to specify well in a protocol because of fonts and Unicode); color model (RGBA is probably sufficient); image inclusion (likely necessary for band logos, channel icons); state management (how the renderer's state persists across commands).

OpenVG was a particularly good reference point because it was a standardized Khronos specification for embedded 2D vector graphics, designed exactly for the kind of devices the Metapedal protocol would be consumed on (mobile-class GPUs, embedded systems). OpenVG never achieved broad commercial success but the specification is mature and the design decisions were thoughtful.

The web platform approach displaces all of this because the browser handles every one of these design questions already: paths through Canvas or SVG, transformations through CSS or the Canvas API, text rendering through the browser's font system, color through CSS color values, images through standard image elements, state through standard application architecture patterns. Adopting the web platform inherits decades of solutions to these problems for free.

If at some future point the web platform approach proves wrong for the Metapedal use case (perhaps because of resource constraints on the consumer side, perhaps because of latency requirements that browsers cannot meet, perhaps because of standardization concerns that browsers introduce), the custom protocol direction is still available and the design space is well-mapped. The notes above are preserved for that potential future use.
