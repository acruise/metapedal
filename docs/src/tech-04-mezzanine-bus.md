# Metapedal Mezzanine Bus Specification

This document specifies the mezzanine bus that connects the Metapedal motherboard to mezzanine boards carrying audio codecs, fat DSP silicon, specialty I/O, and other expansion hardware. The specification covers physical interface, pin allocation, identification protocol, lease-based resource allocation, multi-mezzanine configurations through splitter mezzanines, and the guidance third parties need to build conforming mezzanines.

The mezzanine bus is the architectural commitment that makes Metapedal extensible. The platform ships specific mezzanine variants as v1 products (B1, B2, BX, plus various specialty mezzanines), but the bus specification is open: anyone can design and produce conforming mezzanines that work with any Metapedal motherboard. The platform's commitment is to maintain this contract over time.

## The Architectural Concept

A Metapedal motherboard is fundamentally a digital signal processing platform: a Cortex-M7 microcontroller (STM32H7 family in v1), USB inter-module communication, power management, support circuitry for connecting to peripherals, and the basic infrastructure that every Metapedal module needs. A motherboard alone is a complete A variant module, with simple onboard audio components driving the TRRS jack for headphone output and microphone input.

A mezzanine is a smaller PCB that attaches to the motherboard through a board-to-board connector and provides specialized capabilities: an audio codec with quarter-inch or combo jacks for B variants, a fat DSP chip for BX, CV/gate interfaces for modular synth integration, MIDI DIN connectors for legacy gear, specialty input impedance handling, or whatever else the mezzanine designer chooses. The motherboard provides the core compute and communication; the mezzanine provides the specialized I/O.

This architectural separation has several substantive benefits:

**Manufacturing simplicity.** One motherboard design serves every variant. The mezzanine determines what the module does; the motherboard is identical underneath.

**Repairability.** A failed mezzanine can be replaced without replacing the motherboard. A failed motherboard can be replaced without losing the mezzanine. The platform's total cost of ownership benefits substantially.

**Upgradability.** A user with an A variant can later install a B mezzanine to upgrade the module's capabilities. A user with a B1 can swap to a BX to add fat DSP. The investment in the motherboard persists across upgrades.

**Third-party innovation.** Anyone can design a conforming mezzanine and have it work with any Metapedal motherboard. The platform's open ecosystem benefits from this freedom.

**Specialization without proliferation.** Specialty use cases (XLR connectors, tube preamps, exotic codecs, wireless audio) get their own mezzanines without requiring entire new motherboard designs. The motherboard stays simple and standard while the mezzanines explore the long tail of use cases.

## The Board-to-Board Connector

The mezzanine attaches through a fine-pitch board-to-board connector pair rather than through castellated edge connections, headers, or flex cables. The specific connector is an 80-pin, 0.4 mm pitch dual-row board-to-board pair like the Hirose DF40 family or Samtec QSH/QTH series. The connector spans about 25 mm along the motherboard edge (similar to a 50-pin single-row connector's linear footprint, because the dual-row arrangement packs roughly twice the pin density per linear millimeter) and provides a snap-fit mechanical attachment plus the electrical signal path.

The cost per matched connector pair is about $2.50-3.50 in production quantities, which is real money on the BOM but well worth the benefits. The cost is comparable to a 50-pin single-row connector at production scale; the additional pin count is essentially free.

### Why 80 Pins With Dual-Row Construction

The reasoning that justifies extra connector headroom is the same that drove the platform from 40 pins to 50 pins originally, but applied with more confidence after sustained design attention has surfaced more candidate uses for reserved pins.

The connector specification is a permanent contract. Once mezzanines and motherboards ship against a given pin count, adding pins requires either a parallel second connector (mechanical complexity, layout disruption) or a hard break in the contract (every existing mezzanine becomes incompatible). Both options are bad. The cost of getting the pin count wrong is enormous and only grows over time. The cost of building in headroom now is approximately zero.

The dual-row connector geometry gives roughly twice the pin density per linear millimeter compared to a single-row connector at the same pitch. The 80-pin dual-row connector takes about the same linear footprint as a 50-pin single-row connector. The mechanical envelope is essentially preserved; the PCB layout requires routing on both sides of the connector but this is standard high-density PCB design.

The 80-pin allocation gives substantial headroom for current capabilities plus 30 pins of reserved expansion capacity. Specific candidate uses for the reserved pins that have emerged through design discussion:

A third clock domain (BCK_C, WS_C, MCLK_C) for configurations needing three independent rate domains across mezzanines.

Differential pairs for the highest-quality digital audio signals. Differential signaling gives substantially better jitter and noise immunity; the additional pins would carry the complementary halves of differential clocks and data.

Additional I2S data lines beyond the four currently allocated, for very high channel count multi-channel mezzanines.

Specialized digital interfaces. SPDIF, AES3, MADI for pro-audio interoperability would each take a few pins; the platform could carry these as optional capabilities on appropriate mezzanines.

Hot-plug detection and graceful peripheral lifecycle signaling for a future bus revision that wants to support hot-swap.

Active power management protocols, current monitoring signals, thermal feedback, additional regulated rails.

Future protocol extensions we cannot anticipate now. This is the most important category because it is the unknown unknown; the platform's lifetime might span decades, and the capabilities that will matter in 2040 are not knowable in 2026.



### Why This Connector Choice

The connector approach is well-established in modern electronics. Smartphone and tablet manufacturers use fine-pitch board-to-board connectors throughout their products because they enable modular construction where subassemblies can be tested, replaced, and upgraded independently. The connectors are designed for high-density signal routing with controlled impedance, repeated mating cycles (typically 50-100 cycles rated), and substantial mechanical retention force.

Cable-based daughter boards have problems: the cable acts as an antenna for noise pickup, the connector at each end adds reliability concerns, and impedance discontinuities at cable transitions degrade signal integrity for higher-speed signals like I2S audio. Castellated edge connections are essentially zero-cost in connector terms but require permanent solder attachment, making the assembly unrepairable. The board-to-board connector approach gives signal integrity comparable to castellated edges through controlled-impedance fine-pitch contacts, while keeping the assembly separable for repair, upgrade, and prototyping.

**The separability is genuinely valuable:**

If a mezzanine fails, the user (or a repair shop) can unmate the connector, replace the mezzanine, and remate. With castellated construction the entire assembly would have to be reworked.

A motherboard manufacturer can produce motherboards once and ship them to a different facility for mezzanine attachment, decoupling the manufacturing supply chain.

A hobbyist with a soldering iron can solder the connector to a custom mezzanine they designed, mate it with a standard motherboard, and have a working module without aligning castellations precisely during soldering.

A designer iterating on a mezzanine can swap prototypes against the same motherboard during development.

## The 80-Pin Allocation

The 80 pins are allocated to give B mezzanines all the resources they need with comfortable headroom for current capabilities and substantial headroom for future expansion. The allocation:

**Power and ground (16 pins):**
- 13 pins of ground distributed every 4-5 signal pins along the connector length for the grounding topology, with extra ground pins around high-speed signals
- 1 pin for 3.3V digital (codec digital side, mezzanine digital logic)
- 1 pin for 3.3V analog (codec analog side, analog front-end)
- 1 pin for 5V analog (codec analog with higher dynamic range, phantom power boost converter input)

**Digital audio (10 pins):**
- 1 pin for BCK_A (clock domain A bit clock)
- 1 pin for WS_A (clock domain A word select / LRCK)
- 1 pin for BCK_B (clock domain B bit clock, for two-domain configurations)
- 1 pin for WS_B (clock domain B word select, for two-domain configurations)
- 1 pin for MCLK_A (master clock for domain A, typically 256-1024× the sample rate)
- 1 pin for MCLK_B (separate master clock for domain B, enables 44.1 kHz family support on domain B independently of domain A)
- 4 pins for I2S data: SDI, SDO, SDI2, SDO2 (each independently assignable to either clock domain through the lease protocol)

**Word clock for studio synchronization (2 pins):**
- 1 pin for WCLK_IN (external word clock input from studio sync sources)
- 1 pin for WCLK_OUT (word clock distribution output to external gear)

These pins support pro audio installation scenarios where the Metapedal module needs to lock to an external word clock master, or where the module provides word clock master for downstream gear. Without dedicated pins for word clock, this kind of studio integration would require external sync converters and would degrade the platform's pro audio integration story.

**Codec control (4 pins):**
- 2 pins for I2C (SCL and SDA) for codec configuration
- 1 pin for codec reset
- 1 pin for codec interrupt (status events)

**Analog I/O (4 pins):**
- 2 pins for analog input from the mezzanine to the H7's ADC (CV input, voltage monitoring, control inputs)
- 2 pins for analog output from the H7's DAC to the mezzanine (CV output, control voltage generation)

**GPIO (8 pins):**
- 8 general-purpose I/O pins for status indicators, mode switches, jack-presence detection, mezzanine-specific control, passive identification, and any signaling that mezzanine designers find useful. These pins serve double duty: passive mezzanines that do not use specific GPIO pins for their actual function can tie them to ground in a platform-specified pattern to encode their class. See the identification protocol section for details.

**SPI bus (4 pins):**
- 1 pin SCK, 1 pin MOSI, 1 pin MISO, 1 pin chip-select for high-speed peripherals on the mezzanine

**Spare I2C bus (2 pins):**
- 1 pin SCL, 1 pin SDA for additional I2C peripherals on the mezzanine without sharing the codec's bus

**Reserved for future expansion (30 pins):**
- 30 pins explicitly held in reserve with no defined v1 function

This is substantial headroom and is the major payoff of going to an 80-pin connector. The reserved pins are insurance against future needs the platform cannot currently predict. As the platform evolves over years and the third-party ecosystem builds new categories of mezzanines, the reserved pins are available for protocol extensions, new signal types, or capabilities that emerge from real user needs. Reserving pins now is essentially free; needing them later when the connector contract is locked is enormously expensive.

Examples of what the reserved pins might eventually carry:
- A third clock domain (BCK_C, WS_C, MCLK_C) for configurations needing three independent rate domains across mezzanines
- Differential pairs for the highest-quality digital audio, complementary halves of differential clocks and data lines
- Additional I2S data lines beyond the four currently allocated, for very high channel count multi-channel mezzanines
- Dedicated SPDIF, AES3, or MADI interfaces for pro audio interoperability
- Hot-plug detection and graceful peripheral lifecycle signaling for a future bus revision
- Active power management protocols, current monitoring, thermal feedback signals
- Any other capability that emerges as the platform evolves over the coming decades

The platform commits not to use these pins in v1. Third-party mezzanines should not connect to them. Future protocol versions will assign specific functions to these pins as needed, with backward compatibility considerations preserving the platform's commitment to v1 mezzanines continuing to work.

**Total: 80 pins.**

This allocation gives the platform deep architectural headroom while delivering all current capabilities cleanly. The two-clock-domain capability with independent MCLKs enables multi-rate configurations including 44.1 kHz interoperability. The word clock pins enable pro audio sync integration. The GPIO pins serve double duty as passive identification carriers when not otherwise used. The 30 reserved pins are substantial insurance against future protocol evolution that the platform cannot currently anticipate.

The cost of the 80-pin dual-row connector relative to a 50-pin single-row connector is essentially negligible at production volumes. The cost of getting the pin count wrong and needing to break the contract later is enormous. Spending the additional pins now is the right architectural decision; the reasoning that drove 40-to-50 pins drives 50-to-80 even more strongly because we have surfaced more candidate uses for reserved pins through sustained design attention.

### What Is Deliberately Not on the Mezzanine Bus

The mezzanine bus is deliberately focused on the relationship between the motherboard's digital processing and the mezzanine's audio capabilities. Several signals that might initially seem natural to include are deliberately excluded:

**Ethernet RMII signals.** Ethernet, when present, lives on its own dedicated daughter board with a small dedicated connector to the motherboard, rather than being routed through the mezzanine bus. This keeps the mezzanine bus focused on audio concerns and prevents every mezzanine designer from having to think about ethernet pin allocation even when their mezzanine has nothing to do with networking. See the daughter board pattern in the core technical document.

**USB-C inter-module signals.** USB-C connectors live on their own daughter boards near the enclosure exits. The mezzanine bus does not carry the USB-C differential pairs or configuration channel signals.

**TRRS jack signals.** The TRRS jack lives on its own daughter board near the enclosure exit. The mezzanine bus does not carry the TRRS signals.

**9V power input.** The 9V input lives on its own daughter board near where the barrel jack exits the enclosure. The mezzanine bus carries only the regulated rails the mezzanine actually needs.

These exclusions are deliberate and protective of the mezzanine bus's internal coherence. The bus does what it does well by staying focused on what mezzanines actually need to do their job; other concerns are handled through other mechanisms (daughter boards for physical interfaces, dedicated buses for high-speed networking) that are better suited to those specific concerns.

### The Significance of Four I2S Data Lines

Most mezzanines need only one or two of the four I2S data lines. A simple stereo codec uses SDI for input and SDO for output. Mono codecs use just one of the two.

The two additional data lines (SDI2 and SDO2) exist for two purposes:

**High-channel-count TDM.** Modern codecs supporting many channels through TDM (16 slots per data line at 48 kHz tier 3) use all four data lines together with the shared clocks to carry 64 simultaneous channels. The B16 and B32 mezzanines in the platform's variant family use the secondary data lines for this purpose.

**Multi-mezzanine configurations.** A splitter mezzanine can route the primary I2S to one sub-mezzanine (typically the audio codec) and the secondary I2S to another sub-mezzanine (typically a fat DSP or specialty processor). Both sub-mezzanines operate simultaneously through their separate I2S paths, sharing only the clocks and ground.

The lease-based protocol described later in this document lets mezzanines dynamically claim subsets of these four data lines based on their actual needs.

### Shared Clocking Across the Bus

A meaningful architectural commitment of the mezzanine bus is that v1 supports two independent I2S clock domains, each with its own BCK and WS, sharing a single high-frequency MCLK. Mezzanines and sub-mezzanines on the bus operate within one domain at a time; mezzanines in different domains can operate at different sample rates simultaneously.

The hardware commitment is real in v1 (the motherboard drives two clock domains and the connector exposes BCK_A/WS_A and BCK_B/WS_B as separate pins). Firmware support is phased: v1 firmware fully supports single-domain configurations (which covers all platform-shipped variants); two-domain configurations get firmware support in v1.x when third-party demand or specific use cases justify the additional complexity.

This gives the platform forward compatibility. Anyone designing a mezzanine that needs two-domain capability can know the hardware supports it; the firmware will be ready when there is demonstrated demand.

**The pin allocation reflects this commitment:**

- BCK_A, WS_A for clock domain A
- BCK_B, WS_B for clock domain B (separately routable)
- MCLK shared across both domains (high-frequency reference from which both domains derive their rates)

This costs two additional pins compared to a single-domain bus. The pins come from reducing the GPIO allocation from 6 to 4, since mezzanines have multiple alternative signaling paths (the SPI bus, the spare I2C bus, the codec interrupt line) for status indication.

### The Shared MCLK Math

A single high-frequency MCLK can serve both domains operating at different rates because all the platform's tier rates (48, 96, 192 kHz) divide cleanly from a common high-frequency reference. A 49.152 MHz MCLK (which is 256 × 192 kHz, 512 × 96 kHz, and 1024 × 48 kHz) serves all three rates simultaneously. Codecs internally divide MCLK to generate their local timing; modern codecs all support this kind of MCLK division.

The 44.1 kHz family does not divide cleanly from this MCLK frequency. v1 deliberately does not support 44.1 kHz to keep the shared-MCLK architecture clean. Users wanting 44.1 kHz interoperability use external sample-rate conversion.

### Audio Quality Benefits of Domain-Local Shared Clocks

Within each domain, all consumers run from the same BCK and WS. There is no jitter from clock synchronization issues within a domain. Combining audio streams from consumers in the same domain involves no phase drift because they share the underlying timing.

Across domains, audio streams need sample-rate conversion at the domain boundary if combined. The conversion happens once, at a well-defined boundary, rather than being smeared throughout the chain. The motherboard's H7 (or in BX configurations, the SHARC) handles the conversion using high-quality sample-rate converter algorithms.

### Use Cases for Two-Domain Configurations

The configurations that genuinely benefit from two clock domains:

**Multi-rate single-module configurations.** A module hosts mezzanines for different purposes that genuinely want different rates. A guitarist's codec at 48 kHz on domain A; a vocalist's codec at 96 kHz on domain B; the same module serves both performers without forcing either to compromise on rate.

**External word-clock synchronization.** A studio installation needs the module to lock domain A to external word clock for synchronization with the rest of the studio; domain B operates at the module's internal rate for processing audio that does not need external sync.

**Mixed-rate splitter configurations.** A splitter mezzanine has a standard codec on slot 1 and a high-channel-count multitrack interface on slot 2. The codec attaches to domain A at 48 kHz; the multitrack attaches to domain B at 192 kHz. Each operates at its preferred rate.

**Different DSP rate requirements.** A configuration where the audio I/O codec wants one rate but the fat DSP wants a different rate for algorithm-specific reasons. With two domains, each operates at its preferred rate; the H7 bridges them with rate conversion.

For v1, these use cases are niche. Most users will configure their modules at one tier and operate everyone on the bus at that rate. But the hardware capability is there for users who need it.

### Why Integer Multiples Do Not Eliminate the Need

A reasonable intuition is that if different consumers want sample rates that are integer multiples of each other (48 kHz and 96 kHz, for instance), they could share a single clock domain running at the higher rate. The slow consumer just processes every other sample. This technically works but has real costs that two clock domains avoid.

The slow consumer must run its physical I2S interface at the higher rate. The slow consumer must internally process only every other sample. The slow consumer's silicon and firmware become harder to design.

With two clock domains, the slow consumer just runs at its preferred rate on its domain. Simpler silicon and firmware. The rate conversion (if needed for cross-domain audio flow) happens once at the H7's bridge between domains.

### The Lease Protocol Includes Sample Rate and Domain

When a mezzanine requests an I2S lease, the request specifies:
- Which data lines (any subset of SDI, SDO, SDI2, SDO2)
- Which clock domain (A or B)
- The sample rate (48, 96, or 192 kHz)

The motherboard's lease manager validates:
- The requested data lines are not held by conflicting leases in the same domain
- The requested clock domain is available (either it is unused or other leases are already at the requested rate)
- The requested rate matches the domain's current rate, or the domain can be reconfigured to the requested rate

If a domain is already in use at a different rate, the new lease can be denied with a specific diagnostic: "Cannot grant lease at 96 kHz on domain A; existing leases on domain A are at 48 kHz. Try domain B if available, or reconfigure existing domain A leases." The user resolves through the routing graph configuration.

### Multi-Rate Configurations Across Modules

For configurations where a user needs more than two distinct rates simultaneously, the answer is still to use multiple modules. Two clock domains per module gives substantial flexibility; configurations needing three or more rates use multiple modules with inter-module rate conversion at the boundaries.

This matches how professional audio works generally. Studios commonly have different rate domains for different parts of the signal chain. The platform follows the established pattern with two-domain modules as the unit of rate-coherent processing.

## The B Variant Mezzanine Family

The platform's standard mezzanines, distinguished by how many combo jacks they expose:

**A: No combo jacks.** The A motherboard has no combo jack and uses only the TRRS for any audio I/O. The TRRS handles modest line-level signals (headphone output, consumer-microphone input, line-level connections to powered monitors or interfaces) but does not handle the demanding signal types that combo jacks accept. A variants serve users doing routing, control, or synthesis where the combo-jack signal quality is not needed.

**B1: One combo jack.** A single combo jack (quarter-inch instrument plus XLR microphone, with the appropriate impedance matching, gain ranges, and optional phantom power) plus the codec, analog circuitry, and the connector to the motherboard. Approximately 40 by 60 millimeters. The TRRS is still present on every B1 module for additional line-level I/O.

**B2: Two combo jacks.** Two combo jacks plus a stereo-capable codec, the analog circuitry for both channels, and the connector. Approximately 50 by 70 millimeters. The TRRS is still present.

**BX1 and BX2: B1 or B2 plus fat DSP.** The standard B1 or B2 audio components plus an Analog Devices SHARC ADSP-21489 chip alongside the codec, providing approximately 2.7 GFLOPS of single-precision floating-point performance for heavy DSP algorithms beyond what the H7 alone can handle. Approximately 60 by 90 millimeters to accommodate the additional silicon.

**B16: Sixteen channels.** Multi-channel audio capture using TDM on the primary I2S data lines plus additional GPIO for channel-specific status indicators. Approximately 80 by 100 millimeters with appropriate connectors for the multi-channel I/O. Used in recording and installation applications.

**B32: Thirty-two channels.** Higher-channel-count multitrack using all four I2S data lines simultaneously. Approximately 100 by 120 millimeters. Used in large recording and installation applications.

### Combo Jack Flexibility: Input or Output Per Preset

A key property of the B variant combo jacks is that each jack can act as either an input or an output, with the direction configured per preset rather than fixed by the hardware. This is implemented by appropriate switching and analog conditioning in the mezzanine's circuitry: the combo jack connects to both the codec's ADC path (for input use) and the codec's DAC path (for output use), with switching that selects which is active. The routing graph specifies what each combo jack does in the current preset; the user can change the configuration without recabling.

The permutation space for a B2 mezzanine is therefore: both jacks as inputs (stereo recording or two simultaneous instrument inputs), both jacks as outputs (stereo output to amps or two parallel signal chains), or one of each (the typical "audio flows through the module from input to output" effect-pedal configuration). All three configurations work because each jack is independently configurable; the user picks per preset based on what the routing graph needs.

A B1 mezzanine has a similar choice on its single combo jack: the jack is either an input (the typical guitar-pedal use case) or an output (driving a power amp from a synth voice running on the module, for example). Whichever direction the combo jack is not serving, the TRRS supplements with line-level I/O in the other direction. A B1 used as "guitar pedal" has the combo jack as input and the TRRS as output to headphones or to a powered speaker. A B1 used as "synth voice generator" has the combo jack as output to a power amp and the TRRS as a control input or auxiliary I/O.

### What the Combo Jack Provides That TRRS Does Not

The combo jack is the high-quality electrical interface that handles signal types the TRRS cannot. Specifically:

The combo jack handles instrument-level signals from guitar pickups, with high input impedance (typically 1 megohm or higher) that preserves the natural frequency response of the pickup. Plugging a guitar into a TRRS would load the pickup with the TRRS's low input impedance, dulling the tone and rolling off the high frequencies. The combo jack does this properly.

The combo jack handles microphone-level signals from professional microphones, including phantom-power-required condensers. The combo jack provides 48V phantom power on its XLR pins when the user enables it, plus low-noise preamplification with substantial gain (typically 60+ dB) for the tiny signals from dynamic microphones. The TRRS handles only consumer headset microphones with their built-in bias voltage and modest preamp needs.

The combo jack handles amplifier-level signals when used as output. Driving a guitar power amp or a PA system requires the higher output drive capability and signal level handling that the combo jack supports. The TRRS would clip immediately at amp-input levels.

The TRRS remains useful for what it does well: headphone output (driving low-impedance headphones cleanly), modest line-level signals at consumer reference levels, and consumer-microphone input. The combination of combo jack for the demanding signals plus TRRS for the easy signals lets each interface do what it does best.

Each variant has sub-variants for whether microphone preamp and phantom power components are populated. So a B1 might come in instrument-only and microphone-capable variants, and a B2 might come in instrument-only, single-microphone, and dual-microphone variants. The mezzanine itself captures all this product variation, with the motherboard being identical underneath.

### The BX Variant Specifically

BX is the platform's answer for users who specifically want substantially more DSP capability than the H7 alone provides, packaged in the same single-box form factor as standard B variants. BX comes in BX1 (one combo jack plus the fat DSP) and BX2 (two combo jacks plus the fat DSP) variants, matching the B1 and B2 variants with the addition of the SHARC chip.

The fat DSP chip choice for BX is the Analog Devices SHARC family (ADSP-21489 or similar). SHARC is the established audio DSP technology with a mature development ecosystem refined across decades of professional audio products. The platform inherits a mature toolchain rather than betting on emerging technology.

**The BX cost picture:**
- Standard B1 or B2 mezzanine components (codec, analog circuitry, jacks): ~$15-20 BOM
- SHARC ADSP-21489 chip: ~$25
- Additional power circuitry for the SHARC: ~$3
- I2S routing and switching: ~$3
- Larger PCB to accommodate the SHARC: ~$2 incremental
- **BX mezzanine BOM total: ~$48-53**

A complete BX module (motherboard + BX mezzanine + enclosure) retails around $140-170, compared to ~$80-95 for the standard B1 or B2. The BX premium of $60-90 reflects substantial additional DSP capability.

**What BX enables that standard B variants cannot:**

Neural amp modeling at quality competitive with commercial neural amp products. Large convolution reverbs with multi-second impulse responses. Complex granular synthesis with many simultaneous grains. Heavy spectral processing (frequency-domain effects, vocoders, spectral pitch correction). Multi-effect chains that would otherwise require multiple modules. Audio analysis algorithms (chord detection, beat tracking) running alongside audio processing.

BX is positioned for users who specifically need these heavier algorithms and want them in a single-module package. Users who do not need this capability buy standard B1 or B2 and pay nothing extra for DSP they would not use.

## Splitter Mezzanines

Beyond the single-mezzanine variants the platform ships, the bus specification supports splitter mezzanines that turn the single 80-pin connector into a bus capable of hosting two sub-mezzanines.

A splitter mezzanine is a small board that plugs into the motherboard's 80-pin connector and re-exposes two downstream connectors. Sub-mezzanines (audio codec, fat DSP, CV/gate, MIDI, specialty I/O, prototyping breakouts) plug into the splitter's downstream connectors. The motherboard's firmware discovers the configuration through the identification EEPROM protocol and routes signals appropriately.

The technical basis for splitter mezzanines:

Inherently bus-like signals (I2C, SPI) pass through to multiple sub-mezzanines naturally because these protocols already support multiple devices.

Inherently point-to-point signals (I2S audio) get routed to specific sub-mezzanines through the lease protocol described later. The four I2S data lines (SDI, SDO, SDI2, SDO2) are independently leasable resources; different sub-mezzanines claim different lines.

Power and ground distribute to both sub-mezzanines from the splitter.

GPIO allocates per-sub-mezzanine as needed, with the splitter's identification EEPROM declaring which GPIOs go to which sub-mezzanine slot.

### Form Factor Considerations

A splitter plus two sub-mezzanines is a taller stack than a single mezzanine. The total height is roughly 24-36 mm above the motherboard, depending on the specific mezzanines and splitter design. This exceeds the available vertical clearance in a standard Hammond 1590B enclosure (31 mm internal height after accounting for motherboard mounting).

The solutions:

**Taller enclosures.** Hammond 1590BB (51 mm internal) or 1590DD (50 mm internal) provide enough clearance. Standard pedal-format aluminum enclosures, just taller than the basic option. Users wanting splitter configurations buy modules in the taller enclosure.

**Horizontal splitter geometry.** A non-standard splitter that holds the sub-mezzanines side-by-side rather than stacked. Requires a custom enclosure or a wider 1590-series option. More complex to design but stays in the standard pedal-format aesthetic.

**Separated daughter board geometry.** The splitter has connectors on cables, allowing one sub-mezzanine to live in the main enclosure and the other to live in a small auxiliary enclosure connected by short cable. Preserves the standard 1590B for the main module while enabling the expansion.

For most users, the taller enclosure approach is simplest. The platform's enclosure recommendations include the appropriate sizes for splitter configurations.

## The Lease-Based I2S Resource Protocol

The four I2S data lines (SDI, SDO, SDI2, SDO2) are dynamically allocated through a lease protocol. Any mezzanine can request zero to four of the lines based on what it actually needs for the current configuration. This is meaningfully more flexible than static allocation, which would require enumerating every valid combination of mezzanines and their line assignments.

### The Resource Model

A simple stereo audio codec needs SDI and SDO (2 lines). A pure synthesis mezzanine that only generates audio needs SDO (1 line). A fat DSP that needs separated input and output paths claims SDI and SDO2 (2 lines, specifically separated to avoid contention with a codec's primary I/O). A passive specialty I/O mezzanine that does not use digital audio claims zero lines. A high-channel-count multitrack interface might claim all four lines.

### Lease Lifecycle

**Requested.** A mezzanine asks the motherboard's lease manager for specific lines. The request specifies which lines (any subset of SDI, SDO, SDI2, SDO2), the duration (until released or fixed time), and the priority (mandatory for the mezzanine to function or opportunistic for optimization).

**Granted.** The lease manager checks availability against other active leases. If the requested lines are available, the lease is granted and the mezzanine starts using them.

**Denied.** If higher-priority leases hold conflicting lines, the request is denied. The mezzanine can try alternative line combinations, reduce its functionality, or signal an error through the control interface.

**Active.** The lease is in use; audio is flowing on the leased lines.

**Released.** The mezzanine explicitly gives up the lease. The lines become available for other claimants.

**Expired.** Fixed-duration leases without renewal are automatically reclaimed. This is a safety mechanism for handling crashed or misbehaving mezzanines.

### Protocol Messages

The lease protocol uses I2C messages on the codec control bus:

**LEASE_REQUEST** (mezzanine → motherboard): "I am sub-function X on mezzanine Y; I need lines [SDI, SDO, SDI2, SDO2]; priority is [mandatory, opportunistic]; duration is [until_release, fixed_duration_ms]."

**LEASE_GRANT** (motherboard → mezzanine): "Lease granted; you may use the specified lines starting now."

**LEASE_DENIED** (motherboard → mezzanine): "Lease denied; the requested lines are held by higher-priority claims [list of conflicting claimants]."

**LEASE_RELEASE** (mezzanine → motherboard): "I am releasing the lease on lines [SDI, SDO, SDI2, SDO2]."

**LEASE_RENEW** (mezzanine → motherboard): "I am renewing my fixed-duration lease for another [duration] milliseconds."

**LEASE_EXPIRED** (motherboard → mezzanine): "Your fixed-duration lease has expired; the lines are now available for other claimants."

The motherboard maintains the lease table; the platform's web UI can expose this for diagnostic purposes.

### Mapping to Routing Graph Topology

The user's routing graph specifies audio flows declaratively. The platform's runtime computes a physical plan from the routing graph, including which mezzanines need which I2S lines for which flows. The runtime issues lease requests on behalf of the mezzanines; the motherboard arbitrates; the resulting allocations realize the routing graph.

When the routing graph changes (the user loads a different preset, modifies an active configuration), some leases get released (mezzanines no longer carrying audio flows) and new leases get requested (mezzanines with new flows). The transition is graceful: leases release as their flows stop, new leases activate as their flows start.

### Concrete Examples

**Example 1: Standard B1 module in a chain.** The B1 codec mezzanine requests SDI and SDO for stereo I/O. Granted. SDI2 and SDO2 remain available but unused. Two lines in use, no contention.

**Example 2: BX module (codec + SHARC on same mezzanine).** The codec sub-function requests SDI and SDO. Granted. The SHARC sub-function requests SDI2 and SDO2. Granted. All four lines in use, on the same mezzanine, no contention. The H7 reads codec input from SDI, writes audio to SDO2 (sent to SHARC for processing), reads SHARC's processed audio from SDI2, writes final output to SDO (sent back through codec).

**Example 3: User loads a preset that only uses the codec.** In the BX module above, the new preset has the routing graph entirely on the H7. The codec keeps its SDI/SDO lease; the SHARC sub-function never requests its leases. SDI2 and SDO2 stay available but unused. The SHARC chip is powered down to save energy. When the user loads a different preset that uses the SHARC, the leases are requested and the SHARC powers up. Dynamic power management based on lease activity.

**Example 4: Splitter mezzanine with B1 codec plus separate fat DSP sub-mezzanine.** The splitter mezzanine reports as a passive routing element that does not claim lines. The B1 sub-mezzanine claims SDI and SDO. The fat DSP sub-mezzanine claims SDI2 and SDO2. Same result as BX in terms of line allocation, but achieved through separate sub-mezzanines plus a splitter.

**Example 5: Contention.** A user has a BX module with the SHARC active, but also tries to use a custom sub-mezzanine through a stacked splitter configuration. Both the SHARC and the custom mezzanine want SDI2/SDO2 with mandatory priority. The motherboard cannot grant both.

The arbitration rule is order-of-arrival with priority hints. The first mandatory-priority claim wins; subsequent conflicting mandatory claims are denied with a diagnostic listing the conflicting claimant. The platform's runtime exposes this conflict to the user with a specific message: "Your configuration requests both the BX fat DSP and a custom mezzanine; both need SDI2/SDO2 lines. Choose which should have priority in this configuration."

The user can resolve the conflict through the routing graph: a configuration that needs the SHARC explicitly does not use the custom mezzanine, and vice versa. Or the user can specify per-preset priority that overrides order-of-arrival.

### Why Lease-Based Allocation Is Better Than Static

Static allocation would require the platform's specification to enumerate every valid mezzanine combination and their line assignments, which scales poorly. With four I2S lines and an open-ended set of mezzanine categories, the combinatorial space is large and the specification becomes brittle: adding new mezzanine types requires updating the enumeration.

Leases let the actual configuration determine the actual allocation. Each mezzanine declares what it needs; the motherboard arbitrates. The specification is the protocol, not the enumeration. New mezzanine categories slot into the existing protocol without changing the specification.

Lease-based allocation handles several scenarios cleanly that static allocation does not:

**The BX-versus-splitter case uniformly.** Whether the fat DSP is on the same mezzanine as the codec or on a separate sub-mezzanine through a splitter, the lease protocol gives the same result. The motherboard does not need different firmware paths.

**Dynamic reconfiguration cleanly.** Users changing presets get smooth transitions because leases release and acquire as flows change. No global reconfiguration needed.

**Power management.** Mezzanines holding no leases can power down. The BX SHARC powers up only when actually used.

**Future mezzanine types.** A custom mezzanine needing only one I2S line, or three, or zero, fits into the lease protocol the same way standard mezzanines do.

**Graceful contention.** When mezzanines compete for resources, the lease protocol provides clear diagnostic output that the platform's UI can show to users.

## The Identification Protocol

The platform supports two tiers of mezzanine identification, layered to match the cost and complexity of the mezzanine itself. The first tier uses purely passive components and serves mezzanines with no active electronics. The second tier uses an I2C EEPROM and serves mezzanines that need richer identification or that have active programmable elements requiring firmware. Every mezzanine implements at least the first tier; mezzanines that benefit from the second tier implement both.

### Tier One: Passive Identification Through Tie-to-Ground GPIO Patterns

Mezzanines signal their class to the motherboard through which GPIO pins they tie to ground. The platform does not dedicate a specific identification pin; instead, the existing 8 GPIO pins serve double duty as both general-purpose signaling pins (when actively used by the mezzanine) and identification carriers (when not used by the mezzanine).

The mechanics are simple. The motherboard's GPIO inputs have internal pull-ups enabled (this is essentially free on the H7; the pull-ups are configurable on every GPIO). When no mezzanine is connected, all GPIO pins read high. When a passive mezzanine is connected, the mezzanine ties one or more pins to ground through PCB routing (no components needed; the pins are simply routed to the ground plane on the mezzanine PCB). The motherboard reads the resulting pattern and decodes the mezzanine class from the platform's class registry.

This tier serves an important category of mezzanines: those with no active programmable components at all. Many useful mezzanines fall into this category: a simple passthrough breakout that exposes the motherboard's signals on external jacks; an XLR mezzanine with passive impedance matching and protection but no codec of its own; a MIDI DIN breakout that exposes the platform's UART through the 5-pin DIN connector that legacy MIDI gear uses; a CV/gate breakout for modular synth integration with basic voltage scaling and protection; a pure analog send/return loop mezzanine that routes audio between the motherboard's codec and external analog pedals. These mezzanines have essentially nothing to identify beyond "I am a member of class X with these basic capabilities," and an EEPROM with its supporting circuitry would be a meaningful fraction of their BOM.

The passive identification approach using tie-to-ground patterns lets the mezzanine designer add identification for literally zero component cost. Identification is purely a PCB routing decision: the designer routes one or more of the GPIO pins to the mezzanine's ground plane in the pattern that encodes the mezzanine's class. No resistors are needed. No special analog input pin is needed. The motherboard's GPIO reads at boot detect the pattern.

The encoding capability is generous. With 8 GPIO pins available, the platform can theoretically encode 256 distinct patterns. In practice, the platform reserves several pin-patterns for the no-mezzanine state and for active-mezzanine signaling, and the registry uses the remaining patterns for passive mezzanine classes:

- All pins floating high (no pull-down anywhere): no mezzanine present
- Pin 7 tied to ground, all others floating: active mezzanine, proceed to tier-two identification through EEPROM
- Other specific patterns (covered by the platform's class registry): platform-defined standard passive mezzanines and registered third-party passive classes

The exact patterns for each class are specified in the platform's mezzanine compliance documentation. The platform's registry has room for dozens of distinct passive mezzanine classes, with plenty of headroom for future expansion as the third-party ecosystem grows.

This tier requires no power-up sequencing of complex peripherals, no I2C transactions, no analog readings, and no waiting periods. The motherboard reads the GPIO pins within microseconds of power-up and immediately knows what category of mezzanine is connected. If the result is the no-mezzanine pattern, the motherboard skips mezzanine setup entirely. If the result is a passive class pattern, the motherboard configures itself for that class using built-in knowledge of the class's signal routing and capabilities. If the result is the active-mezzanine pattern, the motherboard proceeds to tier-two identification.

The architectural elegance is that the GPIO pins do not need to be dedicated to identification. They are general-purpose pins that mezzanines can use for actual signaling when they have signaling needs. Passive mezzanines that have no signaling needs tie unused pins to ground for identification. Active mezzanines tie the active-signaling pattern to ground for identification, then use the other GPIO pins normally for their actual signaling purposes once the motherboard has read the identification pattern at boot.

### Tier Two: EEPROM Identification

Mezzanines with active components or that need richer identification implement tier-two identification through an I2C EEPROM. The EEPROM lives at a well-known I2C address and contains a structured identity record. This tier is what the rest of the identification protocol describes; tier-one passive identification simply tells the motherboard whether to read this tier or not.

Mezzanines that implement tier two also implement tier one, with the identification resistor set to code 31 (active mezzanine). This tells the motherboard "do not configure me as a passive class; instead, run the full EEPROM identification protocol." The cost of implementing tier one on a tier-two mezzanine is one additional resistor, which is negligible compared to the EEPROM and supporting circuitry.

### EEPROM Contents

The EEPROM holds a structured identity record:

**Header (16 bytes):** Magic number identifying this as a Metapedal mezzanine EEPROM (4 bytes), specification version this EEPROM conforms to (2 bytes), checksum of the remaining data (2 bytes), reserved (8 bytes).

**Vendor information (32 bytes):** Vendor name as null-terminated string (16 bytes), vendor ID assigned by the platform's vendor registry (4 bytes), website URL (12 bytes).

**Product information (56 bytes):** Product name as null-terminated string (24 bytes), product ID assigned by vendor (4 bytes), product version (4 bytes), hardware revision (4 bytes), production date (4 bytes), serial number (16 bytes).

**Capability descriptor (128 bytes):** Number of sub-functions (1 byte), then a sub-function record for each (variable length per sub-function). A sub-function record includes:
- Sub-function type (audio_codec, fat_dsp, cv_gate, midi, prototype, wireless, splitter_passthrough, etc.)
- I2S line requirements (which subsets of SDI, SDO, SDI2, SDO2 this sub-function might claim)
- I2C address range used by this sub-function
- SPI chip-select if used
- GPIO pin assignments
- Power requirements (current draw on each rail)
- Audio I/O channel counts in each direction
- Sample rate support (which tiers this sub-function supports)
- Control parameter descriptors (what controls the sub-function exposes through the web UI)

**Extension fields (24 bytes):** Optional fields for future protocol extensions.

Total EEPROM size: 256 bytes minimum, with room for larger EEPROMs to hold extended capability information.

### The Serial Number and Per-Instance Identity

The serial number deserves specific discussion because it enables a meaningful piece of user-facing functionality that the platform commits to supporting.

The serial number is a 16-byte field allocated to give each physical mezzanine a globally unique identity. The first 4 bytes encode the vendor ID (the same vendor ID that appears elsewhere in the record); the remaining 12 bytes are a vendor-allocated sequence that distinguishes between physical instances of the same product. This gives each vendor effectively unlimited address space for any plausible production volume (2^96 distinct values per vendor, which is more than enough for any production timeline that humans will ever care about).

The serial number is written to the EEPROM during manufacturing. Each board gets a unique value at production time, either programmed before the EEPROM is soldered or written as part of the board's final test procedure. The platform's mezzanine compliance program specifies the manufacturing requirements: every shipping mezzanine has a unique serial number; the serial number is recorded by the vendor for warranty and traceability purposes; vendors must commit to never reusing serial numbers across production runs.

The serial number is strongly recommended for every tier-two mezzanine but technically optional. A mezzanine without a serial number (with the field set to all zeros, or to a "no serial assigned" sentinel value) still works in the platform but loses the per-instance tracking features described below. The platform supports both cases gracefully but encourages vendors to include serial numbers because the user experience is meaningfully better with them.

Tier-one passive mezzanines do not have serial numbers because they have no EEPROM. This is acceptable because tier-one mezzanines are cheap and generic; users typically do not build per-instance configurations for them. A passive footswitch breakout is interchangeable; if it breaks, the user replaces it with another and the user experience is not meaningfully degraded by the lack of serial number tracking.

### What the Serial Number Enables

The serial number gives the platform a stable handle to attach per-instance metadata and configuration to. Several user-facing features depend on this stable handle:

**Persistent nicknames.** Users name their mezzanines through the web platform UI. Names like "Vocal Air BX" and "Drive BX" and "Practice Room Mezzanine" attach to the physical mezzanine through its serial number. When the mezzanine appears on any module in any chain the user owns, the platform looks up its serial number and uses the user's nickname in the UI. The nickname follows the mezzanine; it is not tied to a particular module or slot.

**Per-instance configuration.** User presets associate configuration with serial numbers rather than just with mezzanine types. A preset says "the BX with serial X gets the hall reverb configuration; the BX with serial Y gets the tube distortion configuration." When the user later loads the preset, the platform finds the BX with serial X anywhere in the chain and applies the hall reverb; finds serial Y and applies the distortion. The configurations follow the physical mezzanines rather than being tied to slot positions.

**Move detection.** When a serial number appears on a different module or chain than it was previously on, the platform notices the move and applies the appropriate previously-saved configuration automatically. The user moves their Vocal Air BX from the vocal chain to the instrument chain temporarily; the BX brings its configuration with it. The platform asks for confirmation on ambiguous cases (the user might want a different configuration in the new context), but the default behavior preserves the user's expressed intent.

**Replacement detection.** If a mezzanine fails and the user replaces it with another of the same type, the platform sees a new serial number where the old one was. The platform can ask the user: "this looks like a replacement for the Vocal Air BX. Do you want to transfer the Vocal Air BX configuration and nickname to this new board?" The user can accept and inherit the old setup on the new hardware, or decline and start fresh. The replacement workflow becomes a guided experience rather than a configuration loss event.

**Cross-chain consistency.** Users with multiple Metapedal chains (a stage rig, a studio rig, a practice rig) get consistent per-instance behavior across chains. A mezzanine moved between chains carries its serial-number-keyed configuration and nickname with it (assuming the user's preset library is shared across chains, which the platform supports through cloud-stored configuration or USB-based export-import).

**Warranty and support.** Vendors can look up specific units by serial number for warranty service, recall management, and customer support. A user contacting the vendor about a defective unit can provide the serial number; the vendor knows exactly which production batch the unit came from and can match it against any known issues.

### Configuration Storage Implications

The motherboard's preset library structure incorporates the serial number tracking. The data model is roughly:

Each user's preset library maps serial numbers to per-mezzanine configuration. The preset itself is a routing graph plus a set of (serial-number, configuration) pairs.

When the preset loads, the motherboard scans the current chain configuration for each serial number in the preset. For each found serial number, the corresponding configuration is applied to that specific mezzanine. For serial numbers not found in the current chain, the preset notes them as "missing" and the user is informed that some of the preset's expected hardware is not currently present.

The motherboard also maintains a separate "hardware registry" that records every serial number it has ever seen, along with the user's nickname for that hardware and any per-mezzanine metadata. The hardware registry grows over time as users acquire and use new mezzanines. The registry is part of the user's stored configuration and persists across firmware updates.

When an active mezzanine connects (whether at boot or via hot-plug on a future hot-plug-capable bus revision), the motherboard reads its EEPROM, extracts the serial number, and looks it up in the hardware registry. If the serial number is known, the user's nickname and metadata for that mezzanine are immediately available. If the serial number is new, the motherboard adds it to the registry with default values (the product name as the nickname) and the user can rename it through the web platform UI.

### Discovery Sequence

On motherboard boot, the H7 firmware performs mezzanine discovery in two stages:

**Stage one: passive identification through GPIO pin pattern.**

1. Enable internal pull-ups on the 8 GPIO pins of the mezzanine bus.
2. Read the GPIO pin pattern (which pins read low because the mezzanine ties them to ground, which pins read high because they float to the pull-up).
3. If all pins read high (no pull-downs anywhere), no mezzanine is present; skip mezzanine setup entirely. The motherboard runs in A-variant configuration using its onboard audio.
4. If the pattern matches a registered passive class, configure the motherboard for that class using built-in knowledge of the class's signal routing and capabilities; skip stage two.
5. If the pattern matches the active-mezzanine signal (pin 7 tied to ground, others floating), proceed to stage two.

**Stage two: EEPROM identification.**

6. Probe the well-known mezzanine EEPROM address on the codec control I2C bus.
7. Read the header and verify the magic number and checksum.
8. Read the vendor and product information for logging and the web UI.
9. Read the capability descriptor to understand what sub-functions exist on this mezzanine.
10. For each sub-function, register its capabilities with the motherboard's mezzanine manager.
11. If the mezzanine is a splitter (the capability descriptor lists splitter_passthrough sub-function with downstream mezzanine slots), recursively discover the sub-mezzanines plugged into the splitter's downstream connectors.

The discovery is hierarchical at stage two: a splitter mezzanine's EEPROM reports its existence; the splitter then exposes the sub-mezzanines through additional I2C addresses at well-known offsets, and the motherboard discovers each sub-mezzanine the same way. Splitter mezzanines are always tier-two (they use the active-mezzanine pattern) because they have active routing logic that requires firmware.

After tier-two identification completes, the motherboard reconfigures the GPIO pins for their actual signaling roles as specified in the active mezzanine's capability descriptor. The pins are now used by the mezzanine for whatever signaling its firmware needs; their identification role was complete after the initial read.

### Hot-Plug Behavior

For v1, hot-plug is not supported. The motherboard discovers mezzanines on boot. Adding or removing mezzanines while powered may produce undefined behavior; the user should power down before changing mezzanines. The system should detect a change in mezzanine configuration on next boot and configure firmware appropriately for the new setup.

v2 might support hot-plug if user demand justifies the additional firmware complexity. The bus protocol does not preclude it; the firmware just does not handle the lifecycle events for v1.

## Firmware Provisioning: Mezzanines Are Stateless At Rest

A core architectural commitment of the mezzanine bus is that mezzanines do not store their own operational firmware. The motherboard holds firmware images for every mezzanine type it knows about, identifies the mezzanine at boot through the identification protocol, and pushes the appropriate firmware down to the mezzanine before normal operation begins. The mezzanine is stateless at rest; the motherboard brings it to life each time the system powers up.

The only persistent state on a mezzanine is the identification EEPROM, which holds metadata (vendor ID, product ID, version, capability descriptor) rather than executable firmware. Everything that runs on programmable elements of the mezzanine (a SHARC DSP, an FPGA, a microcontroller, whatever the mezzanine designer chose) is loaded freshly each boot from the motherboard's firmware repository.

### Why This Design

Several substantive benefits fall out of this approach.

**Lower mezzanine BOM cost.** Flash memory is not free; even a small external flash chip costs $0.30 to $1 in moderate volume, plus the PCB area, plus the supporting passives. Removing storage from every mezzanine saves real money across the platform's lifetime, particularly as the third-party ecosystem grows and many mezzanine designs exist. The savings compound for low-volume specialty mezzanines where the flash chip might be a noticeable fraction of the total BOM.

**Coherent firmware fleet without per-mezzanine update logistics.** With per-mezzanine flash, the platform faces a fleet management problem: a mezzanine purchased two years ago has an older firmware version; one purchased last week has a newer version; the user might own either; the motherboard has to handle both; the user has to think about firmware updates per mezzanine. With motherboard-managed firmware, the user updates motherboard firmware once and every mezzanine type's latest firmware propagates automatically. Bug fixes reach every mezzanine in the field on next boot. There is no fleet to manage because there is no per-mezzanine state to be out of date.

**Better development workflow.** During mezzanine development, designers iterate on firmware by rebuilding the motherboard image that contains it. No separate flashing step for the mezzanine. No risk of bricking a mezzanine by pushing bad firmware to its onboard storage. Experimental firmware that crashes or misbehaves does not persist; power cycle and try again. This dramatically reduces the friction of firmware iteration, which matters for third-party designers without dedicated mezzanine programming infrastructure.

**Improved security posture.** A mezzanine with persistent storage is a place where malicious firmware could live and be hard to detect. A stateless mezzanine cannot harbor persistent malware because nothing on it persists across power cycles. The mezzanine's behavior on any boot is completely determined by the firmware the motherboard pushes, which the user has chosen by installing motherboard firmware. The attack surface for persistent compromise is concentrated at the motherboard rather than distributed across every mezzanine.

**Cleaner third-party ecosystem contract.** A third-party mezzanine designer writes firmware once, submits it to the platform's firmware repository, and the motherboard ships it to any user with that mezzanine. The third party does not handle distribution, version compatibility, end-user flashing tools, or any of the logistics of getting firmware onto user hardware. The platform's centralized firmware distribution does this work, freeing third-party designers to focus on the mezzanine itself.

### The Bootstrap Sequence

Walking through how a module boots with this scheme:

The motherboard powers up and initializes the H7 and basic peripherals. Before powering up the mezzanine bus rails, it reads the GPIO pin pattern on the mezzanine bus to learn what category of mezzanine is connected. The GPIO pins have internal pull-ups enabled; a mezzanine's tie-to-ground pattern shows up as specific pins reading low while others read high.

If the GPIO pattern is all-high (no pull-downs), no mezzanine is connected. The motherboard configures itself for standalone A-variant operation and skips mezzanine setup entirely. The bootstrap is complete.

If the pattern matches a registered passive mezzanine class, the motherboard powers up the bus rails and configures itself for that class using built-in knowledge of the class's signal routing. No EEPROM read is needed; no firmware push is needed; the bootstrap is essentially complete after the passive class configuration. This is the fast path for ultra-simple mezzanines.

If the pattern matches the active-mezzanine signal, the motherboard powers up the bus rails fully and proceeds with tier-two identification. It probes the identification EEPROM over the codec control I2C bus, reads the structured identity record, and now knows what specific mezzanine is connected and what firmware it needs.

The motherboard looks up the appropriate firmware image for this mezzanine type in its internal firmware repository. The repository contains firmware images for every known mezzanine type, indexed by vendor ID, product ID, and version.

If the mezzanine has programmable elements that need firmware, the motherboard initiates the firmware push. The motherboard puts the mezzanine's programmable chip into bootloader mode through a dedicated GPIO or through a specific command sequence, then streams the firmware image over SPI. The chip's bootloader receives the firmware, validates it through a checksum or signature, and starts execution.

For mezzanines with no programmable elements but rich enough capabilities to warrant EEPROM identification (a B1 variant where the audio codec is configured directly over I2C with no separate firmware, but the EEPROM provides detailed capability information that affects routing), the firmware push step is skipped. The motherboard proceeds directly to configuring the codec's registers through I2C using the capability information from the EEPROM.

Once the mezzanine is ready (its firmware loaded and running, or its codec configured), it indicates readiness to the motherboard through a ready signal. The motherboard then configures the mezzanine for the user's current routing graph through normal control messages. Audio flows.

### Bus Signals Required

The bus specification supports this protocol through signals already present in the 80-pin allocation:

**SPI bus** for firmware push. The four SPI pins (SCK, MOSI, MISO, CS) provide the high-speed channel for streaming firmware to programmable mezzanine elements. The same SPI bus serves general high-speed peripheral communication during normal operation; firmware push uses it during boot.

**Boot mode GPIO.** One of the eight GPIO pins is designated as the boot mode signal that the motherboard asserts during firmware loading. The mezzanine's programmable chip uses this signal to determine whether to enter bootloader mode at reset. After firmware loads, the motherboard releases the signal and the chip transitions to running the loaded firmware.

**Ready signal.** One of the GPIO pins (or the codec interrupt line, repurposed at boot) serves as the ready signal from the mezzanine to the motherboard. The motherboard waits for this signal before sending normal control messages, ensuring the mezzanine has completed firmware loading and is operational.

**I2C** for identification and configuration. The codec control I2C bus carries the identification EEPROM read at boot and the configuration messages during normal operation.

No new pins are required beyond what the 80-pin specification already allocates. The firmware provisioning is a protocol layered on top of existing physical signals.

### Firmware Storage on the Motherboard

The motherboard holds mezzanine firmware images through one of two storage paths:

**Internal flash.** The motherboard's flash (on-chip plus external SPI/QSPI flash) holds firmware for the standard mezzanine types the platform ships with. This is fast (no removable media latency) and is always present. The storage is finite, limiting how many mezzanine types can be supported, but for the standard variant family this is adequate.

**microSD card.** The microSD card holds a directory of firmware images for any mezzanine the user has installed. Storage is effectively unlimited; multi-gigabyte cards are cheap. The latency is slightly worse than internal flash but for a one-time-at-boot firmware load the difference is negligible.

The platform's recommended approach combines both. Standard mezzanine firmware (A, B1, B2, BX, B16, B32) lives in internal flash for guaranteed availability. Third-party mezzanine firmware lives on microSD when present, with internal flash falling back if microSD is absent or does not contain a needed image. This gives fast and reliable boot for the common cases while providing extensibility for the long tail of specialty mezzanines.

The web platform's firmware management interface lets users download firmware images for mezzanines they own and write the images to the microSD card. New third-party mezzanines become usable as soon as their firmware is added to the user's microSD.

### Version Handling

The identification EEPROM reports a mezzanine version. The motherboard's firmware repository contains firmware compatible with various mezzanine versions. The motherboard picks the right firmware for the mezzanine's reported version.

This handles hardware revisions cleanly. A B1 v1.0 and a B1 v1.1 with a corrected component value can use different firmware versions if needed; the EEPROM reports the revision, the motherboard selects the matching firmware, and users with older mezzanines continue to work without needing hardware updates.

It also handles firmware evolution. A bug in BX firmware gets fixed in version 2.1; the motherboard's firmware update ships with version 2.1; every BX mezzanine in the field runs the fixed firmware on next boot, with no per-mezzanine user action. The bug went away when the user updated their motherboard.

### Comparison With How This Is Usually Done

Most platforms with mezzanine-like accessories store firmware on the accessory itself: Eurorack modules have their firmware burned in, USB devices have onboard flash, Raspberry Pi HATs with onboard MCUs have their own firmware. Users manage firmware updates per accessory, and inconsistent firmware versions across an ecosystem are common.

The Metapedal approach is closer to how Linux loads firmware for network cards, wireless chips, and GPU initialization sequences: the kernel ships with firmware images, devices are stateless, and the kernel pushes firmware at boot. This is a well-established pattern in computer systems, novel only because most embedded audio gear does not work this way. The platform inherits the benefits of centralized firmware management that the Linux ecosystem has demonstrated over decades.

### Implications for Mezzanine Designers

Third-party mezzanine designers building against this specification:

Should not include flash memory for firmware storage on their mezzanine. The mezzanine BOM should not budget for firmware storage.

Must include the identification EEPROM. This is the one piece of persistent state required.

Must expose a boot path for their programmable chip if the mezzanine has one. The chip needs a documented way to enter bootloader mode in response to the boot mode signal from the motherboard and to receive firmware over SPI. Most modern MCUs, DSPs, and FPGAs support this naturally; the designer just needs to wire the chip's boot pins to the bus signals appropriately.

Must submit firmware to the platform's firmware repository for distribution. The platform handles getting the firmware to users; the designer focuses on writing good firmware.

Should design firmware that loads quickly. The total boot time of the module is dominated by the slowest mezzanine's firmware load time, so designers should keep their firmware images reasonably sized and their boot sequences efficient.

## Configuration vs Firmware: Two Different Things

A meaningful architectural commitment is the clean separation between firmware and configuration. These are two different categories of content that should not be conflated, evolve at different rates, and serve different purposes:

**Firmware** is the code that runs on a mezzanine's programmable element. It is large (kilobytes to megabytes), it is binary, it is specific to a particular chip plus a particular hardware revision, and it changes when the mezzanine designer fixes bugs or adds features. Different firmware versions are not interchangeable; they target specific chip-and-board combinations.

**Configuration** is the data that tells already-running firmware what to do. It is small (bytes to kilobytes), it is structured, and it evolves much more slowly than firmware. The same configuration schema can persist across many firmware revisions because the configuration describes user intent rather than implementation details. A user's "this BX should run a hall reverb with these parameters" is the same intent whether the underlying SHARC firmware is version 1.0 or version 2.5.

Baking configuration into firmware would be a mistake. Every firmware update would risk resetting user configurations. Every user-specific tweak would require rebuilding firmware. The firmware update path would become entangled with the configuration management path. The platform would lose the ability to evolve firmware independently of user setup. The architecture deliberately separates these concerns: firmware is content the platform distributes; configuration is content the user owns and the motherboard manages.

### Mezzanine Identification: Vendor, Device, Hardware Revision

The platform adopts the USB-style identification model: every mezzanine has a triple (vendor ID, device ID, hardware revision) that uniquely identifies what hardware it is.

**Vendor ID** identifies who made the mezzanine. The platform's registry assigns these to participating vendors. The platform's own products have specific reserved vendor IDs. Third-party vendors get IDs through a registration process with the platform's mezzanine compliance program. Vendor IDs are stable across a vendor's product lifetime.

**Device ID** identifies the specific mezzanine product within a vendor's lineup. A vendor might produce a BX, a CV/gate mezzanine, and a specialty XLR mezzanine; each gets its own device ID under the vendor's ID space. Device IDs are stable for a given product.

**Hardware revision** identifies the specific hardware version of that product. A vendor's first batch of a BX might be hardware revision 1.0; a later batch with a corrected resistor value is 1.1; a batch with a different SHARC variant is 2.0. The hardware revision lets firmware target specific hardware variants without forcing the platform to treat them as entirely different products.

The triple (vendor ID, device ID, hardware revision) uniquely identifies any specific mezzanine that exists, anywhere in the ecosystem, at any time. This is enough to look up the right firmware, the right configuration schema, and the right behavior expectations. The triple is reported by the mezzanine's identification EEPROM as part of the tier-two identification described earlier.

### Firmware Metadata Headers

Firmware images carry their compatibility metadata as a structured header. The header includes:

The target vendor ID, device ID, and hardware revision range. A given firmware version targets a specific vendor/device combination plus a range of hardware revisions (often just one, sometimes a span like "this firmware works on hardware revisions 1.0 through 1.3").

The firmware version itself, independent of hardware revision. Firmware v1.0, v1.1, v2.0 for hardware revision 1.0 are different firmware versions of the same product.

A minimum required configuration schema version that the firmware can handle. If the user's stored configuration uses an older schema than the firmware supports, the platform applies a migration; if it uses a newer schema than the firmware supports, the platform knows firmware needs updating.

A maximum supported configuration schema version. If a future schema extends beyond what current firmware understands, the firmware ignores unknown fields gracefully (protobuf semantics) but flags that newer firmware would understand more.

A platform protocol version that the firmware was built against. As the mezzanine bus protocol evolves, firmware declares which version it implements, so the motherboard uses the right protocol version when talking to it.

A signature or checksum for integrity verification before loading.

When the motherboard reads a mezzanine's identification (vendor ID, device ID, hardware revision through the EEPROM), it looks in its firmware repository for the latest compatible firmware. The firmware's header confirms the match before loading.

### Configuration Schemas: Protobuf with Disciplined Evolution

Each mezzanine type has a configuration schema defined as a protobuf message. Protobuf is the right choice for several reasons:

**Wire format compatibility is built in.** Protobuf's design makes forward and backward compatibility a first-class concern. Unknown fields are silently ignored. Default values fill in missing optional fields. Adding new fields does not break old readers; removing fields is discouraged but handled gracefully. This is exactly the discipline that long-lived configuration needs.

**Schema language enforces evolution discipline.** Once a field number is assigned a meaning, it keeps that meaning forever. New fields get new numbers. Deprecated fields are marked but not removed. The discipline is enforced by the schema language itself rather than relying on developer memory or external documentation.

**Language-independent.** The motherboard firmware (Rust), the web platform (JavaScript), and third-party mezzanine code (often C or C++) all work with the same protobuf-defined messages. The schema becomes the contract between all components.

**Mature embedded tooling.** Nanopb (for C/C++) and prost (for Rust) provide efficient code generation for embedded targets. Generated parsers are compact and fast; the runtime overhead is minimal. The H7's resources are comfortably sufficient for protobuf handling at the motherboard level.

The schemas are hierarchical. A base configuration schema all mezzanines share captures common fields (which I2S leases to hold, which clock domain to use, basic routing flags). Each specific mezzanine type extends the base with its own fields. A BX schema adds SHARC algorithm selection and parameters. A CV/gate schema adds voltage range and scaling parameters. A passthrough schema might have no additional fields beyond the base.

Schemas are versioned independently of firmware. The platform's evolution might add new fields to the base schema as new capabilities become available; existing mezzanines do not need firmware updates to ignore those new fields. A schema version of v1 might define the original fields; v2 adds new fields; v3 adds more. Firmware declares which schema versions it understands; the motherboard ensures configuration sent to a mezzanine uses a schema version the firmware supports.

### Where Configuration Lives

The motherboard holds the canonical configuration for every mezzanine and every preset. The mezzanines themselves do not store configuration; like firmware, configuration is pushed from the motherboard each time it is needed.

The web platform's UI edits configuration through the protobuf schemas. When a user changes a parameter, the web platform updates the protobuf message and sends it to the motherboard. The motherboard applies the change to the running mezzanine through control messages over I2C. The mezzanine receives the new parameter values; it does not parse the protobuf itself. The motherboard does the protobuf handling on behalf of all mezzanines, which keeps the per-mezzanine resource cost low.

When a user saves a preset, the protobuf-serialized configuration is stored on the motherboard (in internal flash for current preset state, on microSD for the user's preset library). When a preset loads, the motherboard reads the stored configuration, validates it against the current firmware's schema requirements, and applies it to the relevant mezzanines.

### Cross-Revision Compatibility

The architecture supports several compatibility scenarios that matter for users:

**Firmware update preserves configuration.** A user updates motherboard firmware (which includes updated mezzanine firmware for all known mezzanine types). Their existing configurations continue to work because the configuration schema evolves backward-compatibly. A preset saved with firmware v1.0 still loads correctly under firmware v2.0.

**Hardware revision change preserves most configuration.** A user buys a new BX hardware revision to replace an older one. Their preset library mostly transfers; parameters that exist on both hardware versions apply; parameters that exist only on the new hardware get firmware-generated defaults; parameters that existed only on the old hardware are ignored. The user's investment in building preset libraries persists across hardware refreshes.

**Vendor change does not preserve configuration.** A user replacing a vendor A's mezzanine with vendor B's similar mezzanine cannot expect configuration to transfer; the vendor IDs differ; the schemas are independent. This is the right behavior because the products are genuinely different even if they serve similar functional roles.

**New mezzanine gets sensible defaults.** A brand new mezzanine connecting to a platform with no saved configuration receives firmware-generated defaults. The user's first interaction with the mezzanine is through reasonable starting parameters, not through unconfigured zero values.

### Default Configuration Generation

Firmware generates default configuration. This is the right separation: the firmware knows what reasonable defaults look like for its specific mezzanine type and current firmware version. When a mezzanine first connects, or when a user explicitly resets to defaults, the firmware produces a default configuration message that the motherboard stores.

Defaults can evolve with firmware. A new firmware version might recommend different default values based on user feedback or improved understanding. The schema stays the same; the values within it can change. Users on the latest firmware get the latest defaults; users with custom configurations keep their customizations.

### Implications for Third-Party Mezzanine Designers

A third-party mezzanine designer participating in the ecosystem:

Registers their vendor ID with the platform's mezzanine compliance program.

Assigns device IDs to their products within their vendor ID space.

Tags hardware revisions explicitly; the EEPROM reports the revision; firmware targets specific revisions.

Defines a protobuf schema for the mezzanine's configuration, following the platform's schema conventions (base schema extension, disciplined field numbering, evolution rules). Submits the .proto file to the platform's mezzanine registry alongside firmware.

Implements firmware that handles the configuration schema for the current schema version and ignores unknown fields gracefully. Firmware should also include a default configuration generator that the motherboard can request.

Updates firmware over time as needed for bug fixes and features; the schema stays mostly stable, with new fields added only when truly new functionality requires them. Old configurations continue to work across firmware updates.

This discipline produces an ecosystem where user investments in configuration persist across firmware updates, hardware refreshes, and the natural evolution of the platform. The platform's value compounds because user-created content (presets, configurations, customizations) does not get reset by maintenance activities.

## Mezzanine Mechanical Specification

The mezzanine mechanical envelope:

**Standard B mezzanine dimensions:**
- B1: 40 by 60 mm
- B2: 50 by 70 mm
- BX: 60 by 90 mm
- B16: 80 by 100 mm
- B32: 100 by 120 mm

**Maximum mezzanine envelope** for any mezzanine that fits in a standard Hammond 1590B enclosure: 100 by 60 mm with up to 12 mm height above the motherboard.

**Splitter mezzanine envelope:** the splitter itself plus two sub-mezzanines must fit in the chosen enclosure. For Hammond 1590BB (51 mm internal height), the stack can be up to ~30 mm tall. For Hammond 1590DD (50 mm internal height), similarly ~30 mm tall.

**Mounting:**
- The mezzanine attaches through the 80-pin board-to-board connector mechanically and electrically.
- Two M2 screws into threaded standoffs on the motherboard provide additional mechanical retention. Standoff positions are specified in the platform's mechanical drawing files.
- Connector alignment posts on the mezzanine engage with corresponding holes on the motherboard before the pins make contact, ensuring proper alignment during mating.
- PCB thickness: 1.6 mm standard for FR-4, Tg ≥ 130°C for thermal reliability.

**Stress isolation:**
- The mezzanine's perimeter PCB region (the ground plane around the connector) extends beyond the connector footprint to provide additional copper area for stress distribution.
- Traces routing away from the connector run perpendicular to the connector edge for the first 5 mm minimum, reducing stress concentration on individual traces.
- A "stress-relief neck" between the connector and the rest of the mezzanine prevents flex in the mezzanine from translating to the connector area.

These mechanical details are specified in the platform's mechanical drawing files, which third-party mezzanine designers should reference when designing conforming mezzanines.

## Power Distribution

The motherboard provides three power rails to the mezzanine:

**3.3V digital:** for codec digital sides, mezzanine logic, EEPROM, and other digital components. Current capacity: up to 500 mA per mezzanine. Regulated from the motherboard's main 3.3V digital rail with appropriate filtering.

**3.3V analog:** for codec analog sides, op-amps, and other analog circuits. Current capacity: up to 300 mA per mezzanine. Regulated from the motherboard's separate 3.3V analog rail through an LDO for noise rejection.

**5V analog:** for codec analog sides with higher dynamic range, phantom power boost converters, and other circuits needing 5V. Current capacity: up to 250 mA per mezzanine.

Current limits are enforced by the motherboard's power management. If a mezzanine draws more than its allocated current, the affected rail is shut down with a fault indication. The mezzanine's EEPROM should declare its actual current requirements; the motherboard can budget power based on those declarations.

For multi-mezzanine configurations through a splitter, the total current draw across all sub-mezzanines must not exceed the motherboard's per-rail capacity. The splitter mezzanine's EEPROM declares the combined current requirements for both sub-mezzanines, allowing the motherboard to verify the total is within budget.

## I2C Address Space Allocation

Multiple mezzanines and sub-mezzanines share the codec control I2C bus. To prevent address conflicts, the platform reserves specific I2C address ranges for different mezzanine categories:

**0x00-0x0F:** Reserved for motherboard internal use.

**0x10-0x1F:** Audio codec sub-functions. Codecs from common vendors (Cirrus Logic, ESS, Texas Instruments, Wolfson, AKM) typically use addresses in this range.

**0x20-0x2F:** Fat DSP and processing sub-functions. SHARC, FPGA, NPU, and other DSP silicon should respond in this range.

**0x30-0x3F:** Specialty I/O sub-functions. CV/gate interfaces, MIDI converters, XLR connectors with controllable preamps, and other specialty I/O.

**0x40-0x4F:** Wireless and network sub-functions. Bluetooth modules, Wi-Fi modules, and similar.

**0x50-0x5F:** Mezzanine identification EEPROMs. Each mezzanine's identification EEPROM lives in this range; the specific address is determined by the mezzanine's position in the connector hierarchy (motherboard-attached mezzanine uses 0x50, first sub-mezzanine on a splitter uses 0x51, second sub-mezzanine uses 0x52, and so on).

**0x60-0x6F:** Reserved for future protocol extensions.

**0x70-0x7F:** General-purpose peripheral devices (sensors, displays, prototyping breakouts). Third-party mezzanines with custom peripheral chips use this range.

Mezzanines that need multiple I2C addresses for their sub-functions (a codec plus an additional ADC, for example) request a range of addresses through their EEPROM declaration. The motherboard verifies that the requested addresses are available and not in conflict with other mezzanines.

## Compliance Testing

Third-party mezzanines should pass compliance tests to verify they work correctly with Metapedal motherboards. The platform provides:

**A reference mezzanine design** (open-source PCB design files for a simple B1-equivalent mezzanine) that third parties can use as a starting point for their own designs.

**A test suite** that runs on a Metapedal motherboard and verifies the mezzanine's behavior across the bus protocol: EEPROM identification works correctly, I2C address conflicts are resolved properly, I2S lease requests are handled correctly, audio routing works as expected, power consumption matches the EEPROM declarations.

**A certification process** for third-party mezzanines that have passed the test suite. Certified mezzanines can carry the "Metapedal Mezzanine Compatible" label and are listed in the platform's mezzanine directory.

Compliance is voluntary but encouraged. Non-certified mezzanines may work fine with Metapedal motherboards, but the platform does not guarantee compatibility for them. Users buying mezzanines should look for the certification label if they want assurance of compatibility.

## Guidance for Third-Party Mezzanine Designers

For anyone designing a conforming mezzanine, the platform recommends:

**Start from the reference design.** The reference mezzanine PCB design includes the connector footprint, mounting holes, power filtering, EEPROM placement, and the basic layout patterns the platform has refined. Starting from this baseline saves design effort and avoids common pitfalls.

**Read the EEPROM specification carefully.** The capability descriptor format determines what the motherboard learns about the mezzanine. Declaring capabilities accurately ensures the motherboard configures itself correctly; declaring inaccurately can produce subtle bugs.

**Lease the I2S lines you actually need.** Do not claim lines you do not need. Other mezzanines may need them. Use opportunistic priority for leases that are nice-to-have but not essential for the mezzanine's function.

**Respect the I2C address space allocation.** Use addresses in the range appropriate for your mezzanine's category. Avoid using addresses outside your category's range; doing so risks conflicts with current or future mezzanines from other vendors.

**Test against the platform's compliance suite.** Even if you do not pursue formal certification, running your mezzanine against the test suite catches most compatibility issues before users encounter them.

**Document your mezzanine well.** Users buying third-party mezzanines benefit from clear documentation of what the mezzanine does, how it integrates with the platform, and what limitations exist. The platform's open ecosystem benefits from a culture of good documentation.

**Consider open-sourcing your design.** Open-source mezzanine designs benefit the platform's ecosystem broadly. Many of the Metapedal motherboard's components and patterns came from open-source designs; mezzanine designers contributing back to the ecosystem strengthen the whole platform.

## What This Specification Enables

The mezzanine bus specification enables the platform's expandability story. Beyond the platform's own variant family (A, B1, B2, BX, B16, B32, plus various peripheral and controller variants), third parties can build:

**Specialty fat DSP mezzanines** with different silicon than BX uses: Renesas NPU for neural processing, FPGA for custom algorithms, additional SHARC for more DSP, or whatever else makes sense for specific use cases.

**CV/gate mezzanines** for modular synth integration, with CV inputs and outputs at appropriate voltage levels and impedances, plus gate/trigger inputs and outputs.

**XLR connector mezzanines** for users wanting professional balanced audio without the combo-jack TRS overhead.

**MIDI mezzanines** with 5-pin DIN MIDI in and out, for users with legacy MIDI gear that does not work over USB or TRRS.

**Specialty input impedance mezzanines** for vintage microphones, piezo pickups, magnetic phono cartridges, and other sources with non-standard impedance characteristics.

**Tube preamp mezzanines** for users wanting analog warmth before the digital conversion.

**Prototyping mezzanines** with breadboard or header-based access to the 80-pin signals for hobbyist experimentation.

**Wireless mezzanines** with Bluetooth, Wi-Fi, or RF capabilities for cable-free configurations.

**Compliance-test mezzanines** that exercise specific aspects of the bus protocol for development and debugging.

The platform builds none of these directly. The community builds whatever the community wants, knowing that conforming to the specification means their mezzanines work with any Metapedal motherboard.

This is the same architectural pattern that made Eurorack synthesis successful (where the standard is just connector dimensions and signal conventions), USB peripherals successful (where the standard is just bus protocol and class definitions), and Raspberry Pi HATs successful (where the standard is just connector pinout). The mezzanine bus specification follows the same approach, with the platform's contract enabling third-party innovation indefinitely.

The platform's commitment is to maintain this specification as a stable interface over the platform's lifetime. v1 mezzanines work with v1 motherboards; v2 motherboards (if they ever exist) maintain backward compatibility with v1 mezzanines. The ecosystem invests in the specification knowing the investment persists.
