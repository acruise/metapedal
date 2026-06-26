# Metapedal Daughter Card Specification

This document specifies the daughter card architecture that mechanically and electrically isolates the platform's external connectors from the motherboard and mezzanine. The specification covers combo jack daughter cards for B variant audio I/O, USB-C daughter cards for inter-module communication, M8 peripheral daughter cards for accessories, and the motherboard-direct architecture for the TRRS jack. The specification is intended to support both the platform's own daughter card products and third-party daughter cards that conform to the contract.

The daughter card architecture follows a passive design principle: each daughter card hosts only the external connector and minimal supporting electronics, with all active circuitry (analog buffers, mode switches, microphone preamps, phantom power generation, signal conditioning) living on the main boards where it has full access to power, ground, and control buses. The daughter cards become genuinely sacrificial mechanical layers, cheap to manufacture and cheap to replace, while the precision active electronics remain in the protected interior of the enclosure.

## Architectural Principles

The platform has a three-layer architecture for external I/O, with each layer optimized for its role in the system.

The motherboard and mezzanine form the precision processing tier. These are PCBs with the platform's active electronics: the H7 microcontroller on the motherboard, the audio codec and associated analog circuitry on the B variant mezzanines, the SHARC DSP on BX variant mezzanines. These boards live in the protected interior of the enclosure with no direct connection to external cables, and they are coupled to each other through a rigid board-to-board connector that does not need to absorb mechanical stress.

The daughter cards form the sacrificial mechanical tier. These are small passive PCBs that host the external connectors. Each daughter card carries the connector itself, minimal supporting passives (ESD protection, decoupling capacitors), and a cable transition. For combo jack daughter cards specifically, a single JFET-input op-amp buffer handles high-impedance instrument signal integrity. The daughter cards experience the mechanical stress that external cables impose on the connectors, and they are designed to be field-replaceable so that connector damage can be repaired without affecting other components.

The flexible interconnects between daughter cards and the main boards form the absorbent tier. Shielded multi-conductor cables for analog signals, flexible flat cables with controlled impedance for high-speed digital signals. These interconnects absorb mechanical movement without transmitting bending stress to the precision boards, and they are themselves field-replaceable so that cable damage can be repaired independently of the daughter cards.

External cables connecting to the platform sit outside the system entirely. Their flexibility absorbs the user's mechanical input, and any abuse that exceeds the cable's tolerance translates to abuse at the daughter card layer, where it is absorbed by the daughter card's mechanical mounting or by the flexible interconnect cable.

This hierarchy ensures that the parts of the platform users invest in for the long term, the motherboard and mezzanine, are protected by sacrificial layers that are inexpensive to replace. The total cost of ownership over the platform's lifetime benefits substantially from this architectural commitment, because the most stress-prone components are also the cheapest to swap.

## The Connector Types Covered

The platform has four classes of external connectors, each with different mechanical and electrical characteristics. Three of them use the daughter card architecture; the TRRS jack remains motherboard-direct due to its simpler role.

The combo jacks on B variant modules carry analog audio between the platform and external instruments, microphones, and audio gear. These are mechanically large connectors (16 to 25 millimeters wide each), they have multiple electrical contacts (XLR plus TRS plus switching contacts), and they support multiple configurable modes through the mezzanine's analog circuitry. Combo jacks live on daughter cards that are connected to the mezzanine via shielded multi-conductor cable.

The USB-C ports on every module carry inter-module communication and provide host computer connectivity. USB-C connectors are mechanically fragile in their standard surface-mount configuration, with the smartphone industry's well-known failure mode of connectors being torn off motherboards by lateral cable stress. USB-C ports live on daughter cards connected to the motherboard via flexible flat cable, with the cable absorbing mechanical stress that would otherwise reach the precision motherboard.

The M8 peripheral ports on each module carry connections to expression pedals, Bluetooth dongles, sensor peripherals, and other accessories. M8 connectors are smaller than combo jacks and have substantial inherent mechanical robustness through their threaded retention, but they still benefit from the daughter card approach for consistency with the platform's architectural pattern and for field replaceability. M8 ports live on daughter cards connected to the motherboard via shielded multi-conductor cable.

The TRRS jack on every motherboard carries headphone monitoring output and microphone input. The TRRS connector itself is small (about 7 millimeters diameter) and mechanically modest, and the use case (headphone monitoring during operation, not high-stress cable manipulation) does not create substantial abuse potential. The TRRS jack remains motherboard-direct, with the simplified switching architecture documented in its own section later in this specification.

## The Interconnect Technology Selection

Daughter cards connect to the main boards through flexible interconnects whose technology is chosen based on the signals they carry.

For analog audio signals on combo jack daughter cards, the appropriate interconnect is a shielded multi-conductor cable. Audio signals at line level are robust enough that conventional shielded multi-conductor construction handles them well. Microphone signals at low level benefit from the differential transmission inherent in the XLR architecture, with the cable preserving the balanced pair through to the microphone preamp on the mezzanine. The cable carries the raw electrical signals from the combo jack contacts to the active circuitry on the mezzanine, with the JFET buffer on the daughter card converting high-impedance instrument signals to low-impedance before the cable transition.

For analog signals on M8 peripheral daughter cards, the appropriate interconnect is similarly a shielded multi-conductor cable. The conductor count varies with the M8 variant (3-pin Analog Single, 4 or 5-pin Analog Multi, 8-pin Smart Digital), and the cable is selected to match.

For high-speed digital signals on USB-C daughter cards, the appropriate interconnect is a flexible flat cable with controlled impedance design. The USB 2.0 high-speed differential pair requires 90 ohm differential impedance, which a properly designed FFC maintains across the cable run. FFC connectors are widely used in mobile devices for exactly this kind of application.

The connector at each end of the interconnect is selected based on the interconnect technology. Multi-conductor cables use latching wire-to-board connectors (JST-VH series, Molex MicroFit, or equivalent) for vibration resistance and reliable retention. FFCs use standard FFC connectors with positive latching mechanisms.

The platform commits to specific connector parts in its reference designs, with second-source equivalents documented for supply chain resilience. Third parties building daughter cards can use the reference parts or any pin-compatible equivalent.

## On the Codec's Internal DSP

A reasonable question arises when reading the rest of this specification: modern audio codecs include internal DSP capabilities (mixer matrices, EQ, filters, level controls, automatic level control, soft mute, and various other features), so why does the platform implement equivalent functionality in software on the H7 (and SHARC on BX variants) rather than using what the codec already provides?

The answer is a deliberate architectural commitment to codec independence, with supply chain resilience and software-defined feature consistency as the underlying motivations. The codec is the analog-to-digital and digital-to-analog transducer between the outside world and the platform's signal processing. The processing itself happens on programmable silicon where the platform controls the behavior. Building features on top of codec-specific DSP creates a hard dependency on that specific codec, which conflicts with the platform's commitments around long-term durability and supply chain flexibility.

### Supply Chain Resilience

The audio codec market is dominated by a small number of vendors (AKM, Cirrus Logic, Texas Instruments, ESS, Wolfson, and a few others). Each vendor has experienced supply disruptions over the past decade, most prominently AKM's fab fire in 2020 that disrupted audio codec availability for nearly two years.

If the platform builds features on the AK4619's internal mixer or routing matrix, and AKM experiences another supply disruption, the platform cannot substitute a TI, Cirrus, or ESS codec without rewriting firmware and potentially changing user-facing behavior. If the platform treats the codec as a generic transducer, any pin-compatible or layout-adaptable 4-in/4-out codec from any vendor drops in without firmware changes. The platform's resilience to supply chain disruption is substantial; user experience is preserved across codec substitutions.

### Vendor-Specific Feature Variation

Audio codec DSP features vary widely across vendors. AKM's codecs have their internal mixer architecture; Cirrus has different mixer functionality with different control conventions; TI has its own approach; ESS has yet another. There is no industry-standard codec DSP feature set.

The features are also typically limited compared to what the platform implements in its own DSP code. The H7 has substantial processing capability for audio DSP; the SHARC on BX variants has far more. Software DSP running on this silicon comfortably outperforms what a codec's internal DSP typically provides.

### What the Platform Uses from the Codec

The platform configures the codec through I2C for several universal features that every major audio codec supports: sample rate configuration, bit depth configuration, TDM mode configuration, power-down and channel enable control, mute control, input gain (PGA) control, output volume control, and optionally the DC blocking high-pass filter where available. These are all configuration of transducer behavior rather than DSP processing.

### What the Platform Does Not Use from the Codec

The platform deliberately does not use codec features that vary by vendor: internal mixer or routing matrix, internal EQ beyond the anti-aliasing filter, automatic level control, hardware compression or limiting beyond simple peak detection, level metering beyond raw sample reading, sidechain processing or ducking, or any other codec-specific DSP feature. The platform's software DSP suite provides equivalent or superior capability for all of these, with consistent behavior across codec substitutions.

### The User-Facing Implication

Users see the platform's DSP features as platform features, not codec features. When the user sees "input gain" in the web UI for a combo jack, the gain is implemented through the codec's I2C-controlled PGA on whatever codec is installed. When the user sees "input EQ" or "input compressor," those are implemented in H7 software regardless of which codec sits below them.

This means the platform's user-facing capability set is stable across codec substitutions. The platform's identity lives in its software, not in its codec choice. The codec's internal DSP is real and works correctly for the use cases it serves, but the platform chooses not to use it because doing so would compromise codec independence, supply chain resilience, and feature consistency across the platform's lifetime.

## The Combo Jack Daughter Card

The combo jack daughter card hosts a single combo jack with minimal supporting electronics, providing the platform with one audio I/O point on a B variant module. Multiple combo jack daughter cards are installed in B variants to provide the variant's full I/O capability: one for B1, two for B2, three for B3, four for B4.

### Mechanical Specification

The daughter card PCB is approximately 25 millimeters wide by 30 millimeters tall. The combo jack mounts to the back panel of the module's enclosure through its panel-mount hardware (a hex nut on the jack's threaded mounting collar), with the daughter card extending inward from the back panel.

The daughter card's primary mechanical support comes from the combo jack itself. The jack is rigidly attached to the back panel, and the daughter card's PCB is rigidly attached to the jack through the jack's solder lugs (which are also the electrical connections). Lateral forces on the jack from cable abuse are transmitted to the daughter card, where the cable absorbs them before they reach the mezzanine.

A secondary mechanical attachment is provided through a single M2 screw at the opposite end of the daughter card from the jack. This screw threads into a standoff mounted to the enclosure's chassis, preventing the daughter card from rotating around the jack's axis under torsional loads. The standoff position is specified in the platform's mechanical drawing files.

The cable exits the daughter card through a wire-to-board connector at one edge, with adequate strain relief provided through a cable tie or molded strain relief feature. The cable has sufficient length to reach the mezzanine without strain, with enough slack to absorb the daughter card's mechanical movement under cable abuse.

The daughter card's PCB is 1.6 millimeter thickness FR-4 with Tg greater than or equal to 130 degrees Celsius for thermal reliability. The combo jack's solder lugs are large pads designed to accommodate the jack's mechanical loads without lifting from the PCB during normal use.

### Electrical Architecture

The combo jack provides three groups of electrical contacts: the XLR contacts (pin 1 ground, pin 2 hot, pin 3 cold) for balanced microphone input, the TRS contacts (tip, ring, sleeve) for 1/4 inch instrument or line audio, and the switching contacts that detect which type of plug is inserted (optional and may be omitted in cost-reduced variants).

The daughter card routes these contacts through minimal local electronics to the cable connector. The only active component is a single JFET-input op-amp configured as a high-impedance buffer for the tip contact. This buffer converts the high-impedance signal from instrument-level sources (guitars, basses, piezo pickups, keyboards) to a low-impedance signal before the cable transition, preserving signal integrity over the cable run to the mezzanine.

The buffer is a low-noise audio op-amp (such as the OPA1641 in SOT-23 package or equivalent) with a JFET input stage providing input impedance greater than 1 megohm. The buffer is powered from a low-voltage rail (typically 3.3 volts) supplied through the cable from the mezzanine. The buffer's output drives the cable conductor for the instrument signal.

The ring contact and the XLR contacts route through small ESD protection diodes (for safety against electrostatic discharge during cable insertion) and directly to their respective cable conductors. No active buffering is needed for these contacts because they are either at line level (ring for stereo line input) or balanced (XLR for microphone input), which are robust enough to traverse the cable without local buffering.

The sleeve contact routes directly to the cable's ground conductor. The cable's overall shield is drained to this ground at the daughter card end, with the connection broken at the mezzanine end to prevent ground loops.

No analog switches, no mode selection circuitry, no microphone preamp, no phantom power generation lives on the daughter card. All of this active circuitry is on the mezzanine, where it has access to the platform's power, control, and audio buses.

### Daughter Card BOM

The combo jack daughter card BOM is intentionally minimal:

The Neutrik NCJ6FA-V combo jack (or equivalent): approximately $3.

The JFET-input op-amp for the instrument buffer: approximately $0.50.

ESD protection diodes for all signal conductors: approximately $0.50.

Decoupling capacitors and other passives: approximately $0.50.

A 10-pin wire-to-board connector (JST-VH or equivalent) for the cable: approximately $0.50.

The small PCB and its assembly: approximately $1.

Total daughter card BOM: approximately $5 to $6 in production quantities. This is a substantial reduction from the active daughter board design's $12.50 BOM, with the savings translating directly to lower replacement cost for users and simpler manufacturing for the platform.

### The Six Configurable Modes (on the Mezzanine)

The combo jack supports six configurable modes through the analog switch matrix and active electronics on the mezzanine. The daughter card is fixed and passive; the mode selection happens on the mezzanine through I2C-controlled analog switches that route the cable signals to the codec's ADC and DAC channels appropriately.

Mono instrument input mode routes the buffered tip signal to one codec ADC channel. The ring contact is left floating or grounded. This serves guitars, basses, keyboards, and other unbalanced instrument-level sources.

Stereo line input mode routes the tip and ring contacts to two codec ADC channels through line-level buffers on the mezzanine. The instrument buffer on the daughter card sees a line-level signal in this mode and passes it through with unity gain. This serves stereo line-level sources like synthesizers and drum machines.

Balanced XLR microphone input mode routes the XLR hot and cold contacts to a microphone preamp on the mezzanine, with the preamp's differential output feeding two codec ADC channels (or one channel after differential-to-single-ended conversion in the preamp). Phantom power is optionally enabled through the mezzanine's phantom power generation and routed through the cable to the XLR pins.

Mono line output mode routes one codec DAC channel through a line driver on the mezzanine to the tip contact (via the cable), with the ring contact grounded or floating.

Stereo line output mode routes two codec DAC channels through line drivers on the mezzanine to the tip and ring contacts. This supports passive Y-cable adapters for fanning to two mono cables.

Insert send and return mode routes one codec DAC channel through a line driver to the tip (as the send) and one codec ADC channel through a line-level buffer from the ring (as the return). Both directions are active simultaneously through the same jack with the same connector.

The mode selection is captured in the routing graph configuration and applied through the mezzanine's I2C-controlled analog switches when the preset becomes active. Switching time is well under a millisecond from the I2C write to the analog paths being settled in their new configuration.

### Passive Cable Compatibility

The combo jack uses standard audio wiring conventions on the TRS contacts: tip is the primary signal, ring is the secondary signal (or insert return in insert mode), sleeve is ground. No DC bias or unusual voltages are present on any contact during normal operation. Output signals are AC-coupled through DC blocking capacitors on the mezzanine to prevent DC offset from reaching connected gear.

These conventions ensure that all common passive audio cables and adapters work with the combo jack as expected. Standard TS instrument cables, standard TRS cables, standard XLR microphone cables, passive 1/4 inch TRS to two TS Y-dongles for stereo I/O fan-out or insert send-and-return integration, passive 1/4 inch TRS to 3.5 millimeter TRS adapters for consumer headphones, and passive XLR to 1/4 inch adapters all work as expected.

The platform commits to maintaining these conventions across all combo jack daughter card variants and across all future platform revisions.

### The Combo Jack Cable Specification

The cable between the combo jack daughter card and the mezzanine carries the raw analog signals plus power for the daughter card's buffer plus the phantom power supply for microphone-capable variants. The cable has 8 signal conductors plus an overall shield:

The tip signal from the daughter card's instrument buffer output (1 conductor).

The ring signal from the daughter card (1 conductor).

The XLR hot signal from the daughter card (1 conductor).

The XLR cold signal from the daughter card (1 conductor).

The shared sleeve and XLR ground (1 conductor, with the cable's shield bonded to this conductor at the daughter card end).

The 48 volt phantom power supply from the mezzanine to the daughter card (1 conductor).

The low-voltage power rail for the daughter card's buffer (1 conductor, typically 3.3 volts).

The buffer power return (1 conductor, may be combined with the audio ground or kept separate based on noise considerations).

Total: 8 conductors plus the overall cable shield.

The cable construction uses shielded multi-conductor pro audio cable in the appropriate gauge. The 48 volt phantom conductor is physically separated from the audio signal conductors within the cable jacket, with adequate insulation rating for the voltage. The audio signal conductors are configured as a star-quad or shielded twisted pair group for proper differential signal handling on the XLR pair.

The cable is approximately 100 to 150 millimeters long, providing enough slack for the daughter card to flex under mechanical stress without straining the cable connections. Strain relief features at each connector prevent the wires from being pulled out of the connector by mechanical stress.

The connector at each end is a 10-pin JST-VH series connector (3.96 millimeter pitch) with locking retention, or an equivalent latching connector. The pinout is fully specified in the platform's reference design files.

### The Active Electronics on the Mezzanine

The mezzanine for B variants hosts all the active electronics that the previous active-daughter-board architecture distributed across multiple daughter boards. For a B4 variant with four combo jacks, the mezzanine includes:

Four sets of analog input buffers (line-level buffers for the ring contacts, balanced-to-single-ended converters for the XLR pairs).

Four microphone preamps (for microphone-capable mezzanines), each providing 0 to 36 decibels of selectable gain.

Four sets of analog switches implementing the mode selection for each jack.

Four phantom power generation circuits, or one shared boost converter with switched output to each microphone-capable jack.

Four sets of output drivers feeding the DAC outputs to the cables.

The audio codec (the AK4619 reference part or equivalent) handling the conversion between digital audio and analog.

The mezzanine connector to the motherboard for digital audio (I2S/TDM), control (I2C), and power.

Four cable connectors for the combo jack daughter card cables.

This makes the mezzanine PCB more complex than in the previous architecture, but the design pattern is consistent across the B variant family. The same mezzanine schematic (with appropriate component population) supports B1 through B4 variants. The variant SKU determines how many cable connector positions are populated and how many combo jack daughter cards are shipped with the module.

### Variant Configuration

The B variant family uses a single mezzanine PCB design with population varying by variant:

B1: one combo jack cable connector populated, one combo jack daughter card shipped with the module.

B2: two combo jack cable connectors populated, two combo jack daughter cards shipped.

B3: three combo jack cable connectors populated, three combo jack daughter cards shipped.

B4: four combo jack cable connectors populated, four combo jack daughter cards shipped.

The PCB itself is the same across all four variants; the bill of materials varies. The enclosure size scales with the number of jacks on the back panel, with the standard Hammond 1590B enclosure serving B1, and progressively larger enclosures (1590BB, 1590D, or equivalents) serving higher-numbered variants.

The variant designation determines:

How many cable connector positions on the mezzanine are populated.

How many input buffers, preamps, mode switches, phantom power circuits, and output drivers are populated on the mezzanine.

How many combo jack daughter cards are shipped with the module.

What enclosure is used.

The codec on the mezzanine is always the 4-in/4-out part, with unused channels powered down for variants that do not use all four.

## The USB-C Inter-Module Daughter Card

The USB-C inter-module daughter card hosts a single USB-C receptacle with appropriate mechanical reinforcement and minimal supporting electronics. Each module has multiple USB-C ports (typically two, one upstream and one downstream for daisy-chain configurations), so each module has multiple USB-C daughter cards.

### Mechanical Specification

The USB-C daughter card is small, approximately 15 millimeters by 20 millimeters, with the USB-C receptacle at one edge and the FFC connector at the opposite edge. The receptacle is a robust panel-mount USB-C variant with side-tab solder pads and additional through-hole stakes for mechanical support, exceeding the mechanical specifications of a standard surface-mount USB-C receptacle.

The receptacle is mounted recessed into the enclosure's back panel, behind a metal shroud that is part of the enclosure casting. The shroud surrounds the receptacle body and protects it from direct mechanical impact. The shroud also provides physical alignment during cable insertion, guiding the plug into the receptacle. The shroud is electrically continuous with the chassis ground for ESD protection.

A TPU grommet slides onto the cable and pushes into the shroud after the cable is plugged in, providing strain relief and lateral cable retention. The grommet is the sacrificial mechanical element under extreme cable abuse: it slides out of the shroud rather than damaging the connector. The grommet can be replaced by the user if it gets damaged.

The daughter card mounts to the back panel through the receptacle's mechanical hardware (the panel mount), with a secondary M1.6 screw at the opposite end providing torsional support. The daughter card sits perpendicular to the back panel, extending inward from the receptacle to the FFC connector position.

The FFC exits the daughter card through a low-insertion-force latching FFC connector, with the FFC routed inside the enclosure to the corresponding connector on the motherboard. The FFC has appropriate strain relief at each end and adequate slack for the daughter card to flex without stressing the FFC.

### Electrical Architecture

The USB-C daughter card is passive. It includes the USB-C receptacle, ESD protection diodes on the high-speed data pair (D+ and D-), and the FFC connector. No active electronics live on the daughter card; all USB signal handling, power delivery negotiation, and protocol management happens on the motherboard.

The FFC carries the following signals between the daughter card and the motherboard:

VBUS power, with two parallel conductors for current capacity (up to the USB Power Delivery negotiated current).

Ground, with two parallel conductors for current return and signal shielding.

D positive and D negative, the USB 2.0 high-speed differential pair, routed as a controlled-impedance differential pair within the FFC.

CC1 and CC2, the USB-C configuration channels for power delivery negotiation, orientation detection, and accessory mode signaling.

SBU1 and SBU2, the sideband pins, routed for future expansion even though they are typically unused for inter-module bus purposes.

Total: 10 conductors on the FFC, organized for appropriate signal integrity with the differential pair positioned for proper controlled impedance.

### FFC Specification

The FFC between the USB-C daughter card and the motherboard is specified for controlled impedance and adequate shielding. The reference FFC has 10 conductors at 0.5 millimeter pitch, with the differential pair positioned for proper signal integrity. Total cable thickness is approximately 0.3 millimeters, allowing the FFC to flex easily without imposing stress on the connectors. The cable includes appropriate shield layer or ground conductor allocation for impedance control on the differential pair.

The FFC length is approximately 50 to 100 millimeters depending on the daughter card position relative to the motherboard. The connectors at each end are standard 0.5 millimeter pitch FFC connectors with latching retention, available from multiple vendors.

## The M8 Peripheral Daughter Cards

The M8 peripheral connectors come in three variants based on the platform's peripheral specification: Analog Single (3-pin), Analog Multi (4 or 5-pin), and Smart Digital (8-pin). Each variant has its own daughter card design, but all three follow the same passive architecture.

### M8 Connector Background

The M8 connector family comes from industrial automation, where it is widely used for sensor and actuator connections. The connectors are 8 millimeter diameter circular threaded connectors with metric M8 thread for the screw-retention mechanism. They are rated for thousands of mating cycles, are watertight in their better grades, and are well-standardized across multiple manufacturers in the one to three dollar range.

The platform uses M8 connectors for peripherals because they are clearly different from the USB-C inter-module connectors and the quarter-inch audio jacks (preventing user confusion about which cable goes where), and they have appropriate mechanical robustness for the peripheral use case. The threaded retention also provides inherent mechanical decoupling: lateral force on the cable transfers to the panel via the M8 nut rather than to the daughter card's solder joints.

### Analog Single M8 Daughter Card

The Analog Single variant uses a 3-pin M8 carrying power, ground, and one analog signal. The daughter card hosts the M8 receptacle with minimal supporting electronics: ESD protection on the signal line, decoupling capacitors on the power conductor, and a cable connector.

The cable to the motherboard carries:

The analog signal (1 conductor).

Power supply (1 conductor, typically 3.3 or 5 volts depending on the specific peripheral capabilities).

Ground (1 conductor).

Total: 3 conductors. A small 3-pin or 4-pin JST-XH connector at 2.5 millimeter pitch is appropriate for the cable termination.

### Analog Multi M8 Daughter Card

The Analog Multi variant uses a 4 or 5-pin M8 carrying power, ground, and 2 or 3 analog signals. The daughter card hosts the M8 receptacle with ESD protection on each signal line.

The cable to the motherboard has 4 or 5 conductors depending on the M8 variant: the analog signals (2 or 3 conductors), power (1 conductor), and ground (1 conductor).

### Smart Digital M8 Daughter Card

The Smart Digital variant uses an 8-pin M8 carrying power, ground, and a small digital bus. The daughter card hosts the M8 receptacle with ESD protection on all signal lines and possibly level shifting if the peripheral operates at different logic levels than the platform's 3.3 volt standard.

The cable to the motherboard has up to 8 conductors carrying the digital bus signals plus power and ground. For Smart Digital variants using I2C as the digital bus, the cable carries SCL, SDA, optional interrupt, optional reset, power, and ground (6 conductors total). For variants using higher-speed buses like SPI or custom serial protocols, additional conductors are needed and FFC may be substituted for wire harness based on signal integrity requirements.

### Active Electronics on the Motherboard

The motherboard hosts the active electronics for M8 peripheral interfacing:

Signal conditioning amplifiers or buffers for analog peripheral inputs.

Digital bus controllers for Smart Digital peripherals (I2C controllers, SPI controllers, etc.).

Power supply switches to enable or disable peripheral power per port.

Identification logic to discover what peripheral is connected to each port (through the peripheral's own identification protocol on the digital bus).

The motherboard's PCB has space allocated for these active electronics adjacent to the cable connector positions for the M8 daughter cards. The pattern is the same as the mezzanine for combo jacks: active electronics on the main board, passive connector cards at the panel.

## The TRRS Motherboard-Direct Architecture

The TRRS jack remains motherboard-direct in the platform's v1 architecture, with the simplified switching arrangement that takes advantage of the four-in-four-out codec on B variants.

### Why TRRS Stays Motherboard-Direct

The TRRS connector is mechanically small and the use case does not generate substantial mechanical stress. Headphones plug in and stay plugged in for extended periods; users do not typically yank headphone cables violently or stomp on them. The mechanical decoupling benefit of a daughter card is much smaller for TRRS than for the other connector types.

The TRRS jack is also used for casual purposes (headphone monitoring, basic microphone capture) rather than for performance-critical I/O. A user with a damaged TRRS jack can still use the platform's full audio capabilities through the combo jacks and the USB-C ports. The TRRS jack's failure is an inconvenience rather than a show-stopping event.

For these reasons, the platform keeps the TRRS jack on the motherboard directly, with the cost savings being applied to other parts of the platform where mechanical decoupling matters more.

### The Output Architecture

The TRRS jack's audio output (the left and right output channels on the standard CTIA pinout) is driven by a dedicated cheap audio DAC chip that is always populated on the motherboard. This DAC is locked at 48 kilohertz, 24-bit operation regardless of the rest of the platform's configured sample rate.

The cheap audio DAC chip is one of the Cirrus Logic CS4344 family, Texas Instruments PCM5102, or ESS Sabre entry-level parts. All of these deliver audio-grade conversion well above what any headphone or in-ear monitor can reproduce. The cost is approximately $0.50 to $1.00 in production quantities; this is paid on every motherboard as a fixed BOM cost.

The H7's audio peripheral feeds the cheap DAC through a dedicated I2S channel at 48 kilohertz, 24-bit. When the main audio path operates at higher rates (96 kilohertz or 192 kilohertz for tier 2 or tier 1 modes), the H7 performs sample rate conversion from the main path's rate down to 48 kilohertz before feeding the headphone DAC. The conversion is computationally trivial and the artifacts are well below the threshold of audibility on any headphone.

This hard-linked output approach has several substantial benefits: consistent behavior across variants, no switching in the audio output path, predictable timing, and simplified mezzanine signal routing.

### The Input Architecture

The TRRS jack's microphone input contact is connected through a single analog switch to one of two paths: the motherboard's microphone preamp circuit followed by the H7's built-in 16-bit ADC, or the mezzanine connector for routing to one of the codec's ADC channels.

The analog switch is a single-pole-double-throw CMOS analog switch in a small package. The switch is controlled by a GPIO from the H7. The switch position determines which ADC receives the microphone signal.

On A variant modules (no mezzanine installed), the switch is always in the H7 ADC position because there is no codec available to switch to.

On B variant modules (with a codec mezzanine installed), the switch position is controlled by the routing graph configuration. Users can choose to route the TRRS microphone to the H7 ADC (preserving codec channels for combo jack work, accepting lower-quality microphone capture) or to the codec ADC (sacrificing one codec input channel for higher-quality microphone capture).

The choice is captured in the routing graph and applied through the analog switch when the preset becomes active.

## Field Repairability

The daughter card architecture supports a layered repair workflow that allows users to diagnose and fix damage at the appropriate layer without affecting other components.

### The Repair Workflow

A user with a damaged module first identifies the symptom: a specific connector not working, audio dropouts on a specific input, a specific jack producing noise, no output from headphone monitoring, no inter-module communication on a specific port. The symptom typically points to a specific layer where the failure resides.

If a specific connector has failed mechanically, the user swaps the affected daughter card. The daughter card is held in by the connector's mounting hardware plus one or two small screws; the swap takes about five minutes with basic tools. The daughter card is essentially universal across positions (any combo jack daughter card works in any combo jack position because they are passive and identical), so spares are interchangeable.

If a daughter card is working but its cable shows broken conductors or damaged connectors, the user swaps the cable. Replacement cables are stocked as spare parts and ship with appropriate connectors pre-attached.

If a daughter card and cable are both fine but the corresponding active electronics on the mezzanine or motherboard are not working, the user swaps the affected board. The mezzanine swap requires opening the enclosure but does not involve any soldering or PCB rework. The motherboard swap requires the most disassembly but is still a screwdriver-only operation.

Each layer is independently diagnostic and independently replaceable. The platform's repair documentation includes diagnostic flowcharts that help users identify which layer is the source of a failure, and replacement parts are stocked for each layer.

### Spare Parts Inventory

The platform's recommended spare parts inventory for a typical user includes:

One or two spare combo jack daughter cards (cheap enough at $5-6 retail to stock multiple).

One spare combo jack daughter card cable.

One spare USB-C daughter card.

One spare USB-C daughter card FFC.

One spare TPU grommet for USB-C cable retention.

One spare M8 daughter card of each variant the user has installed.

One spare M8 daughter card cable for each variant.

The full kit costs approximately $25 to $40 retail and provides comprehensive coverage for the most common failure modes. The lower individual daughter card costs (compared to the active-daughter-board architecture) make stocking spares more economical.

### The Economic Picture

The total cost of ownership for a Metapedal module is dramatically lower than for an integrated pedal because each layer is independently replaceable. A user whose module experiences a damaged combo jack pays approximately $6 for a replacement daughter card, compared to perhaps $100 or more for a complete pedal replacement in the integrated alternative.

The cost per daughter card replacement is genuinely small. Users can stock spares without significant expense, and service partners can offer fixed-price repairs that include both labor and parts at approachable rates.

## Cost Analysis Across the B Variant Family

The passive daughter card architecture results in the following approximate BOM costs.

### Per-Component BOM

Combo jack daughter card: approximately $5-6 (jack, buffer op-amp, ESD protection, passives, PCB, connector).

Combo jack cable: approximately $3 (shielded multi-conductor with connectors).

USB-C daughter card: approximately $4 (receptacle, ESD protection, PCB, FFC connector).

USB-C FFC: approximately $1.50.

M8 daughter card (any variant): approximately $3-4 (M8 receptacle, ESD protection, PCB, connector).

M8 cable: approximately $2.

### Mezzanine Active Electronics Per Combo Jack

The mezzanine grows to accommodate the active electronics per jack. The per-jack active electronics cost on the mezzanine is approximately:

Analog input buffers and line drivers: ~$1.

Mode-switching analog switches: ~$1.

Microphone preamp (for microphone-capable jacks): ~$1.50.

Phantom power generation (shared across multiple jacks, but the switching circuit per jack): ~$0.50.

Connector for the daughter card cable: ~$0.50.

Per-jack mezzanine electronics: approximately $4-5.

### Per-Variant BOM Impact

The total daughter-card-and-mezzanine BOM impact per variant (compared to a hypothetical integrated approach):

A variant: 2 USB-C daughter cards plus FFCs (~$11), 3 M8 daughter cards plus cables (~$18). Plus the M8 active electronics on the motherboard (~$6). Total: approximately $35.

B1 variant: same USB-C and M8 elements as A (~$35), plus 1 combo jack daughter card plus cable (~$9), plus per-jack mezzanine electronics (~$5). Total: approximately $49.

B2 variant: same USB-C and M8 (~$35), plus 2 combo jack daughter cards and cables (~$18), plus per-jack mezzanine electronics (~$10). Total: approximately $63.

B4 variant: same USB-C and M8 (~$35), plus 4 combo jack daughter cards and cables (~$36), plus per-jack mezzanine electronics (~$20). Total: approximately $91.

BX variants follow the same pattern as the corresponding B variants plus the SHARC and supporting circuitry.

These numbers are reasonable for the platform's positioning, and they preserve the field-repairability benefits that justify the daughter card architecture. The per-daughter-card replacement cost is now in the $4-9 range (including cable), which is genuinely cheap for users to stock as spares or replace as needed.

### The Manufacturing Picture

The daughter card architecture creates a small set of distinct PCB designs:

Motherboard (one design across the entire product family).

Mezzanine PCBs (one design per B variant family, derived from a single master schematic).

Combo jack daughter card (one design, identical across all B variants and all positions).

USB-C daughter card (one design).

M8 daughter cards (three designs, one per M8 variant).

Cables (one design per daughter card type).

Total distinct PCB designs: approximately seven. Each design is simpler than its active-daughter-board predecessor (fewer components, passive layout, smaller footprint), which simplifies manufacturing and testing.

The spare parts inventory is well-defined and predictable. The daughter cards are interchangeable across positions, simplifying inventory management.

## The Third-Party Daughter Card Ecosystem

The platform commits to maintaining the daughter card specification as a stable interface, with backward compatibility for daughter cards across platform revisions. Third parties can invest in daughter card designs knowing the investment persists.

The passive architecture makes third-party daughter cards genuinely easy to design: there is no firmware to write, no identification protocol to implement, no active electronics to validate against the platform's signal characteristics. A vendor wanting to build a specialty combo jack daughter card just needs to host their chosen connector on a small PCB and terminate the platform's specified cable interface.

Examples of third-party daughter cards that could exist within the v1 ecosystem:

Specialty combo jack daughter cards with premium connectors: gold-plated XLR contacts, audiophile-grade combo jack variants, locking 1/4 inch jacks (Neutrik silentPlug or similar).

Alternative connector daughter cards: XLR-only without the 1/4 inch jack for users who only need balanced microphone input, 1/4 inch-only for users who only need instrument or line audio.

DIN MIDI daughter cards: 5-pin DIN connectors for legacy MIDI gear, replacing the M8 Smart Digital option with traditional MIDI hardware.

RCA daughter cards: consumer-audio RCA connectors for users who need to interface with consumer gear.

XLR output daughter cards: balanced XLR outputs for connecting to professional audio destinations without adapters.

BNC daughter cards: word-clock synchronization for studio installations.

Wireless daughter cards: integrated Bluetooth or RF audio modules.

The platform builds the standard combo jack, USB-C, and M8 daughter cards. The community builds whatever the community wants, knowing that conforming to the daughter card specification means their hardware works with any Metapedal motherboard and mezzanine combination.

## Compliance Testing

Third-party daughter cards should pass compliance tests to verify they work correctly with Metapedal motherboards and mezzanines. The compliance picture is simpler than for the previous active-daughter-board architecture because there are no firmware compatibility concerns.

### The Reference Daughter Card Designs

The platform publishes open-source PCB design files for reference implementations of each daughter card type:

A reference combo jack daughter card with the standard Neutrik combo jack, the JFET buffer for instrument input integrity, the standard ESD protection, and the standard cable connector.

A reference USB-C daughter card with the robust USB-C receptacle, the shroud-compatible mounting features, ESD protection, and the standard FFC connector.

Reference M8 daughter cards in all three variants, each with the appropriate receptacle and ESD protection.

Third parties can use these reference designs as starting points for their own daughter cards.

### The Test Suite

The platform provides a test suite that runs on a Metapedal motherboard and verifies a daughter card's behavior. The test suite covers signal routing (the daughter card correctly routes connector contacts to cable conductors), mechanical compatibility (the daughter card fits the platform's enclosure designs and mounts correctly), and audio quality (noise floor and frequency response measurements within specified tolerances).

A daughter card that passes the test suite is verified to work correctly with the platform's motherboards and mezzanines.

### Certification Process

The platform offers a voluntary certification process for third-party daughter cards. Certified daughter cards have passed the compliance test suite, have been reviewed for documentation quality, carry the "Metapedal Daughter Card Compatible" label, and are listed in the platform's daughter card directory.

Certification is voluntary; non-certified daughter cards may work fine with Metapedal motherboards but lack the assurance of formal verification.

## What This Specification Commits the Platform To

The daughter card specification creates several long-term commitments that the platform must honor across its lifetime.

The cable connector pinouts for each daughter card type are stable. The platform cannot change the pinouts without breaking compatibility with existing third-party daughter cards.

The FFC pinout for USB-C daughter cards is stable for the same reasons.

The mechanical envelope for each daughter card type is stable. Future daughter card variants must fit within the documented mechanical dimensions, mount through the documented features, and conform to the documented enclosure interface.

The platform commits to maintaining these interfaces over the platform's lifetime. v1 daughter cards work with v1 motherboards and mezzanines; v2 motherboards (if they ever exist) maintain backward compatibility with v1 daughter cards. The ecosystem invests in the specification knowing the investment persists.

The platform does not commit to maintaining specific component choices: the JFET buffer op-amp on the combo jack daughter card could be substituted for a different audio-grade op-amp; the ESD protection diodes can be substituted for equivalent parts; the connectors can be substituted for pin-compatible alternatives. These are implementation choices within the contract rather than the contract itself.

## Summary

The daughter card architecture is the platform's mechanical decoupling layer, with a passive design that isolates the precision motherboard and mezzanine from the mechanical abuse that external cables impose on connectors. The architecture covers combo jack daughter cards for B variant audio I/O (with a single JFET buffer for instrument signal integrity), USB-C daughter cards for inter-module communication (fully passive), M8 peripheral daughter cards for accessories (fully passive in all three variants), and the simplified motherboard-direct architecture for the TRRS jack.

The daughter cards connect to the main boards through flexible interconnects (shielded multi-conductor cables for analog signals, FFCs for high-speed digital signals) that absorb mechanical stress. The interconnects are field-replaceable, providing a sacrificial layer that fails before the more expensive components.

The active electronics that the previous active-daughter-board architecture distributed across daughter boards now live on the main boards: the mezzanine handles combo jack active circuitry (input buffers, mode switches, microphone preamps, phantom power generation), and the motherboard handles USB-C signal conditioning and M8 peripheral interfacing. The codec on the mezzanine remains the 4-in/4-out reference part with all DSP handled in software per the codec independence principle documented earlier in this specification.

The passive daughter card architecture dramatically reduces the cost of individual daughter card replacement (now in the $4-9 range including cable, instead of $15) while preserving the field-repairability and mechanical isolation benefits that justify the daughter card approach in the first place. The total system BOM impact is moderate but acceptable, and the platform's positioning as a long-term modular investment is genuinely supported by the architecture.

The architecture supports a layered repair workflow where each component is independently diagnosable and replaceable, dramatically reducing the total cost of ownership compared to integrated alternatives. The platform's spare parts inventory is well-defined, manufacturing is standardized within each daughter card type, and the third-party ecosystem can innovate on daughter card designs within the platform's stable contract.

The TRRS architecture remains motherboard-direct with hard-linked output through the always-populated cheap audio DAC chip at a fixed 48 kHz/24-bit rate, and switched microphone input between the H7's built-in ADC and the codec's ADC when a B variant mezzanine is installed.

The platform commits to the daughter card specification as a stable interface for the platform's lifetime, enabling third-party innovation and user investment in long-term hardware compatibility.
