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

The mezzanine attaches through a fine-pitch board-to-board connector pair rather than through castellated edge connections, headers, or flex cables. The specific connector is a 50-pin, 0.5 mm pitch board-to-board pair like the Hirose DF12 family or Samtec ERM/ERF series. The connector spans about 25 mm along the motherboard edge and provides a snap-fit mechanical attachment plus the electrical signal path.

The cost per matched connector pair is about $1.80-3 in production quantities, which is real money on the BOM but well worth the benefits.

### Why 50 Pins Rather Than 40

The 40-pin version of this connector family is sufficient for the v1 mezzanine variants we have specified, but the 50-pin version costs essentially the same (a few cents extra in production quantities) and provides 10 additional pins for future capability that the platform cannot easily add later. The connector specification is a permanent contract: once mezzanines and motherboards ship against the 40-pin contract, adding pins requires either a parallel second connector (mechanical complexity) or a hard break in the contract (every existing mezzanine becomes incompatible).

The cost of getting the pin count wrong is enormous and only grows over time. The cost of building in headroom now is approximately zero. The 50-pin connector locks in the platform's mezzanine specification with substantial headroom for future capabilities.

The 10 additional pins enable:
- Twice the GPIO count, giving mezzanine designers substantially more signaling flexibility
- Dedicated word clock input and output for studio sync support
- A second master clock for clock domain B, enabling 44.1 kHz family support alongside 48 kHz family
- Reserved/future expansion pins for protocol extensions the platform cannot currently predict

These capabilities are described in detail in the pin allocation section below.

### Why This Connector Choice

The connector approach is well-established in modern electronics. Smartphone and tablet manufacturers use fine-pitch board-to-board connectors throughout their products because they enable modular construction where subassemblies can be tested, replaced, and upgraded independently. The connectors are designed for high-density signal routing with controlled impedance, repeated mating cycles (typically 50-100 cycles rated), and substantial mechanical retention force.

Cable-based daughter boards have problems: the cable acts as an antenna for noise pickup, the connector at each end adds reliability concerns, and impedance discontinuities at cable transitions degrade signal integrity for higher-speed signals like I2S audio. Castellated edge connections are essentially zero-cost in connector terms but require permanent solder attachment, making the assembly unrepairable. The board-to-board connector approach gives signal integrity comparable to castellated edges through controlled-impedance fine-pitch contacts, while keeping the assembly separable for repair, upgrade, and prototyping.

**The separability is genuinely valuable:**

If a mezzanine fails, the user (or a repair shop) can unmate the connector, replace the mezzanine, and remate. With castellated construction the entire assembly would have to be reworked.

A motherboard manufacturer can produce motherboards once and ship them to a different facility for mezzanine attachment, decoupling the manufacturing supply chain.

A hobbyist with a soldering iron can solder the connector to a custom mezzanine they designed, mate it with a standard motherboard, and have a working module without aligning castellations precisely during soldering.

A designer iterating on a mezzanine can swap prototypes against the same motherboard during development.

## The 50-Pin Allocation

The 50 pins are allocated to give B mezzanines all the resources they need with comfortable headroom for current capabilities and substantial headroom for future expansion. The allocation:

**Power and ground (13 pins):**
- 10 pins of ground distributed every 4-5 signal pins along the connector length for the grounding topology
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
- 8 general-purpose I/O pins for status indicators, mode switches, jack-presence detection, mezzanine-specific control, and any signaling that mezzanine designers find useful. The generous GPIO allocation reflects the platform's commitment to mezzanine designer flexibility; nobody has ever complained about having too many GPIO pins.

**SPI bus (4 pins):**
- 1 pin SCK, 1 pin MOSI, 1 pin MISO, 1 pin chip-select for high-speed peripherals on the mezzanine

**Spare I2C bus (2 pins):**
- 1 pin SCL, 1 pin SDA for additional I2C peripherals on the mezzanine without sharing the codec's bus

**Reserved for future expansion (3 pins):**
- 3 pins explicitly held in reserve with no defined v1 function

These pins are insurance against future needs the platform cannot currently predict. As the platform evolves over years and the third-party ecosystem builds new categories of mezzanines, the reserved pins are available for protocol extensions, new signal types, or capabilities that emerge from real user needs. Reserving pins now is essentially free; needing them later when the connector contract is locked is enormously expensive.

Examples of what the reserved pins might eventually carry:
- A third clock domain (BCK_C, WS_C) for configurations needing three independent rate domains
- Dedicated SPDIF or AES3 digital audio interfaces for pro audio interoperability
- Differential clock pairs for the highest-quality digital audio
- High-speed serial buses for future bandwidth needs
- Boot-time identification flags before the I2C EEPROM is read
- Any other capability that emerges as the platform evolves

The platform commits not to use these pins in v1. Third-party mezzanines should not connect to them. Future protocol versions will assign specific functions to these pins as needed, with backward compatibility considerations preserving the platform's commitment to v1 mezzanines continuing to work.

**Total: 50 pins.**

This allocation gives the platform substantial headroom while delivering all current capabilities cleanly. The two-clock-domain capability with independent MCLKs enables multi-rate configurations including 44.1 kHz interoperability. The word clock pins enable pro audio sync integration. The reserved pins are insurance against future needs.

The cost of the 50-pin connector relative to the 40-pin connector is a few cents per pair in production quantities. The cost of getting the pin count wrong and needing to break the contract later is enormous. Spending the additional pins now is the right architectural decision.

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

The platform's standard mezzanines:

**B1: One combo jack.** A single combo jack (quarter-inch instrument plus XLR microphone) plus the codec, basic analog circuitry, and the connector to the motherboard. Approximately 40 by 60 millimeters. The mezzanine extends the motherboard's capabilities to professional audio I/O without the complexity of multi-channel configurations.

**B2: Two combo jacks.** Two combo jacks plus the codec (in a dual-channel variant), the analog circuitry for both channels, and the connector. Approximately 50 by 70 millimeters. The dual-channel configuration is what audio interfaces typically offer; the B2 is functionally equivalent to a Focusrite Scarlett 2i2 or similar.

**BX: B1 or B2 plus fat DSP.** The standard B1 or B2 audio components plus an Analog Devices SHARC ADSP-21489 chip alongside the codec, providing approximately 2.7 GFLOPS of single-precision floating-point performance for heavy DSP algorithms beyond what the H7 alone can handle. Approximately 60 by 90 millimeters to accommodate the additional silicon.

**B16: Sixteen channels.** Multi-channel audio capture using TDM on the primary I2S data lines plus additional GPIO for channel-specific status indicators. Approximately 80 by 100 millimeters with appropriate connectors for the multi-channel I/O. Used in recording and installation applications.

**B32: Thirty-two channels.** Higher-channel-count multitrack using all four I2S data lines simultaneously. Approximately 100 by 120 millimeters. Used in large recording and installation applications.

Each variant has sub-variants for whether microphone preamp and phantom power components are populated. So a B1 might come in instrument-only and microphone-capable variants, and a B2 might come in instrument-only, single-microphone, and dual-microphone variants. The mezzanine itself captures all this product variation, with the motherboard being identical underneath.

### The BX Variant Specifically

BX is the platform's answer for users who specifically want substantially more DSP capability than the H7 alone provides, packaged in the same single-box form factor as standard B variants.

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

Beyond the single-mezzanine variants the platform ships, the bus specification supports splitter mezzanines that turn the single 50-pin connector into a bus capable of hosting two sub-mezzanines.

A splitter mezzanine is a small board that plugs into the motherboard's 50-pin connector and re-exposes two downstream connectors. Sub-mezzanines (audio codec, fat DSP, CV/gate, MIDI, specialty I/O, prototyping breakouts) plug into the splitter's downstream connectors. The motherboard's firmware discovers the configuration through the identification EEPROM protocol and routes signals appropriately.

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

Every mezzanine has an EEPROM or small I2C device at a well-known I2C address that reports the mezzanine's identity to the motherboard on boot.

### EEPROM Contents

The EEPROM holds a structured identity record:

**Header (16 bytes):** Magic number identifying this as a Metapedal mezzanine EEPROM (4 bytes), specification version this EEPROM conforms to (2 bytes), checksum of the remaining data (2 bytes), reserved (8 bytes).

**Vendor information (32 bytes):** Vendor name as null-terminated string (16 bytes), vendor ID assigned by the platform's vendor registry (4 bytes), website URL (12 bytes).

**Product information (48 bytes):** Product name as null-terminated string (24 bytes), product ID assigned by vendor (4 bytes), product version (4 bytes), production date (4 bytes), serial number (12 bytes).

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

**Extension fields (32 bytes):** Optional fields for future protocol extensions.

Total EEPROM size: 256 bytes minimum, with room for larger EEPROMs to hold extended capability information.

### Discovery Sequence

On motherboard boot, the H7 firmware performs mezzanine discovery:

1. Probe the well-known mezzanine EEPROM address on the codec control I2C bus.
2. If found, read the header and verify the magic number and checksum.
3. If valid, read the vendor and product information for logging and the web UI.
4. Read the capability descriptor to understand what sub-functions exist on this mezzanine.
5. For each sub-function, register its capabilities with the motherboard's mezzanine manager.
6. If the mezzanine is a splitter (the capability descriptor lists splitter_passthrough sub-function with downstream mezzanine slots), recursively discover the sub-mezzanines plugged into the splitter's downstream connectors.

The discovery is hierarchical: a splitter mezzanine's EEPROM reports its existence; the splitter then exposes the sub-mezzanines through additional I2C addresses at well-known offsets, and the motherboard discovers each sub-mezzanine the same way it discovered the first mezzanine.

### Hot-Plug Behavior

For v1, hot-plug is not supported. The motherboard discovers mezzanines on boot. Adding or removing mezzanines while powered may produce undefined behavior; the user should power down before changing mezzanines. The system should detect a change in mezzanine configuration on next boot and configure firmware appropriately for the new setup.

v2 might support hot-plug if user demand justifies the additional firmware complexity. The bus protocol does not preclude it; the firmware just does not handle the lifecycle events for v1.

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
- The mezzanine attaches through the 50-pin board-to-board connector mechanically and electrically.
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

**Prototyping mezzanines** with breadboard or header-based access to the 50-pin signals for hobbyist experimentation.

**Wireless mezzanines** with Bluetooth, Wi-Fi, or RF capabilities for cable-free configurations.

**Compliance-test mezzanines** that exercise specific aspects of the bus protocol for development and debugging.

The platform builds none of these directly. The community builds whatever the community wants, knowing that conforming to the specification means their mezzanines work with any Metapedal motherboard.

This is the same architectural pattern that made Eurorack synthesis successful (where the standard is just connector dimensions and signal conventions), USB peripherals successful (where the standard is just bus protocol and class definitions), and Raspberry Pi HATs successful (where the standard is just connector pinout). The mezzanine bus specification follows the same approach, with the platform's contract enabling third-party innovation indefinitely.

The platform's commitment is to maintain this specification as a stable interface over the platform's lifetime. v1 mezzanines work with v1 motherboards; v2 motherboards (if they ever exist) maintain backward compatibility with v1 mezzanines. The ecosystem invests in the specification knowing the investment persists.
