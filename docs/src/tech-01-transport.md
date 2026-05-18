# Metapedal Inter-Module Transport

This document covers the inter-module communication: connectors, cable quality testing, the ethernet transport variant, end-to-end latency analysis, topology choices (chain versus star), the bus protocol and leader election, bandwidth analysis, and USB host mode for external connections.

The audience is engineers building the firmware transport layer and people designing multi-module configurations.

For the on-module mezzanine bus, see `tech-04-mezzanine-bus.md`.

## Connectors: USB-C for Inter-Module, M8 for Peripherals

The inter-module bus carries power, control, and eventually digital audio between modules in a chain. The connector choice for this bus is USB-C, for several strong reasons. USB Power Delivery, which runs over USB-C, gives off-the-shelf power negotiation between a power source and connected modules, including budget tracking and refusal of overloads. The connector has high-speed differential pairs that can carry any protocol the designer wants. USB-C is also the universal connection to host computers and phones, so the same port that connects modules together can connect a module to a laptop for recording or firmware updates or configuration.

USB-C does have known mechanical robustness problems, particularly around side loads on the connector. When a cable is yanked sideways, the moment arm levers the surface-mount connector against its solder pads and can tear them off the PCB. This is the standard failure mode that destroys USB-C connectors in phones, laptops, and other consumer devices. For a pedalboard application, which involves more cable abuse than most, the standard USB-C mechanical design is not robust enough on its own.

The mitigation that resolves this is a combination of techniques. The connector is recessed into the metal enclosure rather than flush-mounted, with a metal shroud that is part of the enclosure surrounding the connector body. When the cable is plugged in, only the cable protrudes from the shroud, while the connector and plug body sit inside the protected pocket. A direct stomp on the connection lands on the metal shroud rather than on the connector. The shroud also mechanically captures the connector body, so any side loads on the cable get resolved against the shroud rather than against the connector's solder joints. The PCB sees essentially no mechanical stress from cable manipulation.

The shroud serves several additional purposes. It provides physical alignment during insertion, guiding the plug into the connector as a funnel does. It provides a natural anchor point for cable strain relief. It can serve as a ground connection point for the cable shield, electrically continuous with the enclosure. And it provides visual hierarchy on the back panel, making the connector locations clearly demarcated.

A further refinement is a TPU grommet that the user slides onto the cable before plugging it in and then pushes into the shroud after the cable is seated. The grommet has interference fit with both the shroud opening and the cable, providing strain relief and lateral cable retention without depending on the connector itself. TPU is the right material because it is tough, abrasion-resistant, chemically resistant to the various liquids that find their way onto pedalboards, and elastic enough to deform under shock loads without taking permanent set. The grommet's failure mode under extreme overload is graceful: it slides out of the shroud rather than damaging the connector. Push it back in and the connection is restored.

The combination of recessed connector in a metal shroud, mechanical capture of the connector body, ruggedized connector parts, and a TPU grommet, gets USB-C robustness for the inter-module bus to a level genuinely competitive with proper bayonet connectors. Not as bulletproof as a custom locking connector, but close enough that the disadvantages of USB-C are reduced to acceptable levels while the advantages, power negotiation and host computer compatibility and cable ubiquity, are all preserved.

For peripheral connections, separate from the inter-module bus, the right connector is the M8 family from the industrial automation world. M8 connectors are circular threaded or bayonet-locking connectors, eight millimeters in diameter, rated for thousands of mating cycles, watertight in their better grades, and well-standardized across multiple manufacturers in the one-to-three-dollar range. They are small enough that several can fit on the back panel of a pedal-sized module. They are clearly different from the USB-C inter-module ports, so the user cannot confuse one for the other. And they are clearly different from quarter-inch audio jacks, so the user cannot confuse a peripheral connection for an audio connection.

The M8 peripheral system uses three standard pinouts, each corresponding to a class of peripheral. The Analog Single pinout, on a three-pin M8, carries power, ground, and a single analog signal back from the peripheral. This is the right pinout for a passive expression pedal or any other simple analog control. The Analog Multi pinout, on a four or five-pin M8, supports peripherals with a few discrete switches or multiple analog signals, like a multi-button footswitch. The Smart Digital pinout, on an eight-pin M8, carries power, ground, and a small digital bus, supporting peripherals with their own microcontrollers that present richer interfaces.

The standardization of these pinouts is what enables an ecosystem of compatible peripherals. Peripheral manufacturers know exactly which pinout to implement for each kind of product. Users know that any peripheral with an Analog Single connector works in any module with an Analog Single port. Cables are interchangeable within each category.

A standard module has one M8 Smart Digital port on its back panel, plus the inter-module USB-C ports, the 9V barrel jack for pedalboard power, the microSD slot, and the audio jacks (which mount on the sides rather than the back to conserve back panel space). The Smart Digital port is the most useful single M8 port type because it handles intelligent peripherals like Bluetooth dongles, expression pedals with their own microcontrollers, and other rich peripherals. Users who need the Analog Single or Analog Multi pinouts, or who need multiple M8 ports of any type, get them from a breakout peripheral rather than from additional ports on the standard module.

The motivation for this is back panel real estate. Three M8 connectors at eight millimeters diameter plus mounting hardware, with cables needing clearance for insertion, take significant panel space. Combined with the 9V barrel, two USB-C ports, the microSD slot, and any audio jacks, the back panel of a pedal-sized module gets crowded fast. Putting all M8 connector types on every module is over-provisioning for the typical user; most users have only one or two peripherals connected at any time, and they do not need three M8 ports each. Concentrating the peripheral ports onto a dedicated breakout serves users who actually need elaborate setups, without burdening every standard module with capacity its typical user does not use.

The breakout peripheral is a separate product, a small enclosure (perhaps the size of a half-pedal) with several M8 ports of each pinout type, that connects to a standard module through the inter-module bus. The breakout draws power from the inter-module bus and presents its connected peripherals to the chain through the same routing graph mechanisms as on-module peripherals. From the routing graph's perspective, a peripheral connected through the breakout is no different from a peripheral connected directly to a module; the breakout is transparent to the routing logic.

The breakout can be smarter than just providing more M8 ports. A peripheral hub module designed for elaborate setups might include several M8 ports of each pinout type, additional inter-module bus ports for connecting multiple chains together, a USB host port for connecting a display or MIDI controller, perhaps even a power distribution function for delivering 9V to other pedals on the board. The hub becomes a real organizational element in the user's pedalboard rather than just an expansion port, similar in spirit to dedicated MIDI switching units like the Voodoo Lab Ground Control or the Disaster Area Designs MIDI controllers that serve this function in premium guitar rigs.

The connector layout for the standard module's back panel becomes: 9V barrel jack, two USB-C inter-module ports, one M8 Smart Digital port, and the microSD slot. That is five connectors on the back panel, which is busy but achievable in a Hammond 1590B-sized enclosure. The side panels carry the audio jacks: the quarter-inch input on one side and the quarter-inch output on the other side for B variants, with the same side-mount convention used by premium pedals like the Strymon Timeline, Eventide H9, and most Empress products. The TRRS jack can also mount on a side panel since it serves headphone monitoring (which expects a side jack on consumer devices) and MIDI (which uses persistent connections that do not need back panel access). Side-mounting the audio jacks is a well-established pedal convention that users understand, and it frees substantial back panel space for the other connectors.

The Smart Digital port is particularly interesting because it enables peripherals that are essentially Metapedal modules without the standard enclosure. A peripheral with a sensor, a microcontroller, and an M8 connector can present itself as a control source for any parameter in the chain, without needing its own metal case or its own OLED or its own USB-C ports. The host module provides power and the connection to the broader chain. The peripheral just contributes its own control data. The barrier to building a new control surface drops significantly, because a hobbyist can prototype on a breadboard, package in a plastic box, terminate with an M8 connector, and have a functional Metapedal peripheral.

The Smart Digital port also enables peripherals that have physical or regulatory requirements at odds with the metal enclosure of the standard module. A Bluetooth dongle for phone-app connectivity is the canonical example. Putting a Bluetooth radio inside a metal box is bad because the box is a Faraday cage that blocks 2.4 GHz signals. Putting the Bluetooth radio in an external peripheral that plugs into a Smart Digital port keeps the radio outside the cage, in its own RF-transparent housing, where the antenna can do its job. The standard module never has to be RF-certified, the cost of Bluetooth is paid only by users who want it, and one Bluetooth peripheral in a chain serves all the modules in the chain through the inter-module bus.

## Cable Quality Testing

USB-C cables vary wildly in their actual capabilities despite looking identical externally. The cheap cables bundled with phone chargers, the cables sold at gas stations, the dollar-store cables, the unbranded online cables: these often support only USB 2.0 high-speed at best, sometimes only USB 2.0 full-speed (12 Mbps), and occasionally fail at high-speed signaling rates entirely. They might or might not carry full 5V at meaningful current. They might or might not have the CC (Configuration Channel) wires that USB-C requires. They often lie about their capabilities through poorly programmed eMarker chips, or have no eMarker at all when they should.

For Metapedal's inter-module bus running USB high-speed at 480 Mbps, a poor cable causes real and confusing failures: intermittent audio dropouts when signal integrity is marginal, chain enumeration at full-speed instead of high-speed when the cable's pairs are too noisy, power delivery failures at higher current draws. The user experience of these failures is genuinely terrible: a chain that worked yesterday does not work today with no obvious indication of what changed, and the user tries replacing modules before discovering it was a cable issue.

The platform should include cable quality testing as a v1 firmware feature. The implementation requires no additional hardware (the H7 already has the relevant USB and voltage-monitoring capabilities), takes a few seconds to run per cable, and produces actionable diagnostic output.

### v1 Cable Tests

Four basic tests cover the common failure modes:

**USB enumeration check**: when the cable is plugged in, what USB speed does the connection negotiate? The platform's USB stack already performs this negotiation; the test just reports the result. A cable that negotiates full-speed (12 Mbps) when it should negotiate high-speed (480 Mbps) is failing; either the cable lacks the high-speed pairs entirely or those pairs are too marginal to enumerate at high-speed. This test is essentially free in implementation terms.

**Bandwidth measurement**: a few seconds of timed data transfer that measures actual throughput. The H7's USB hardware supports this directly. A cable that should be supporting 480 Mbps but only manages 100 Mbps is failing somehow; a cable hitting 400+ Mbps is probably fine. The bandwidth number is concrete and actionable: the user sees the cable's real capability and can decide whether it meets their needs.

**Power delivery measurement**: voltage drop across the cable at known load currents. The platform's existing voltage monitoring on its power rails handles the measurement directly. A cable with high resistance in its power wires shows voltage drop under load; the measurement quantifies the drop. The user gets a specific number ("this cable shows 0.8V drop at 1A load") that tells them whether the cable can support the chain's actual power needs.

**CC line presence check (USB-C specific)**: USB-C requires the CC (Configuration Channel) wires to be properly connected for the cable to support anything beyond USB 2.0. Some cheap cables have only the basic USB 2.0 pairs and skip the CC wires entirely, which makes them work for slow USB 2.0 transfer but fail at faster modes or power delivery negotiation. The platform detects CC line presence by checking whether the expected CC signals arrive when both ends are plugged in. Simple GPIO check; no special hardware required.

### User Experience

The cable test results appear in the web UI as a "Cable Diagnostics" panel. The user selects which cable to test (the platform knows about each cable based on which module connects to which), runs the test, and gets results. Each cable shows its measured throughput, voltage drop, USB speed negotiation result, and an overall quality rating: "Good," "Marginal," or "Replace." For marginal or failing cables, the UI suggests specific actions: "replace with a USB-C cable rated for USB 3.2 or better" or "this cable cannot support the chain's power needs at this load."

The platform also runs lightweight continuous monitoring in the background, with the UI showing real-time cable health. If a cable starts degrading mid-performance (a connector starts losing connection, a wire breaks under repeated flexing), the platform notices and warns the user before complete failure occurs. This is genuinely valuable for live performance scenarios where catastrophic failure mid-song is much worse than a warning a few minutes earlier.

The diagnostic output is actionable in a way that generic "the chain failed to enumerate" error messages are not. The user knows exactly which cable is the problem and exactly what to do about it, rather than trying to diagnose by swapping modules.

### Routing Graph Bandwidth Validation

The cable testing infrastructure becomes substantially more valuable when it feeds into a validation layer between routing graph configuration and graph execution. The platform should not just test cables on demand; it should proactively validate that the configured routing graph is actually deliverable on the connected hardware. If the graph wants to route Tier 2 signals through a chain, the platform should verify that every cable on the path can carry the required bandwidth before the user hits play.

The fundamental insight is that the routing graph specifies what audio flows where, and the cables determine what audio can flow where. A configuration that exceeds cable capacity fails in subtle and frustrating ways: dropouts, glitches, distortion, or just silence. The user does not know whether the problem is a misconfigured module, a software bug, an underpowered cable, or something else. Pre-flight validation catches the problem at configuration time, with a specific diagnostic that points to the actual root cause.

The bandwidth calculation is straightforward. For a flow of N channels at sample rate R and bit depth B, the raw audio bit rate is N × R × B. With USB protocol overhead for isochronous transfers (call it 15%), the required bandwidth on the cable is approximately N × R × B × 1.15.

Some example bandwidths for the platform's quality tiers:

For 8 channels at Tier 2 (96 kHz / 24-bit): raw 18.432 Mbps, required ~21.2 Mbps. Fits comfortably in USB 2.0 high-speed (480 Mbps) but does not fit in USB 2.0 full-speed (12 Mbps).

For 4 channels at Tier 1 (192 kHz / 24-bit): raw 18.432 Mbps, required ~21.2 Mbps. Same bandwidth as 8 channels of Tier 2 because the math is symmetric.

For 16 channels at Tier 3 (48 kHz / 24-bit): raw 18.432 Mbps, required ~21.2 Mbps. Again same bandwidth at the per-data-wire-pair ceiling.

For aggregate flows on a single cable segment (multiple flows being routed through the same cable, in both directions, for the same chain), the bandwidths add. A cable carrying 8 channels of Tier 2 in each direction needs roughly 42 Mbps of total bandwidth, still comfortable on USB high-speed. A multi-performer chain carrying many channels in busy cable segments might use 100+ Mbps of cable bandwidth, still well within USB high-speed capacity but consuming meaningful headroom.

### The Validation Logic

When the user modifies the routing graph (adding a flow, increasing a quality tier, adding a new module to the chain), the platform validates that the configuration is deliverable:

First, compute the aggregate bandwidth requirement on each cable segment based on the configured flows. The graph specifies which module hosts each input and output and which DSP nodes process the flow. The validator walks the graph, identifies which flows cross which cable segments, and sums the per-segment bandwidth requirements.

Second, compare against the measured cable capacity from the cable test results. Each cable has a measured maximum throughput (the bandwidth test result) and a measured signal quality (the USB enumeration result, the voltage drop, the error counters). The validator uses the measured maximum as the cable's capacity ceiling, with some safety margin (perhaps 70-80% utilization) to account for protocol overhead and transient demand.

Third, identify cables where the aggregate flow requirement exceeds the safe capacity. These cables become validation failures that need user attention.

Fourth, propose specific remediations. The platform knows which flows are using which cables; for each over-capacity cable, the platform can suggest reducing quality tier on specific flows, routing flows through different paths if alternatives exist, or replacing the specific cable with one that has more capacity.

### User Experience

The graph editor in the web UI shows the chain visually with cable segments highlighted. When the user adds a flow that would push a cable beyond its capacity, that cable highlights red with a warning tooltip showing the specific problem ("this cable measured 280 Mbps capacity but the configured flows require 350 Mbps, exceeding safe utilization"). The user sees immediately which cable is the problem.

As the user adjusts quality tiers and channel counts in the UI, the bandwidth requirements update in real time. Cable segments turn yellow as their utilization approaches the safety threshold, red as they exceed it. The user sees the consequences of configuration choices before committing to them, rather than discovering problems at performance time.

The platform also provides specific remediation suggestions. If the user has configured 12 channels of Tier 2 audio flowing through a marginal cable, the UI might suggest: "Reduce these flows to Tier 3 to fit on the current cable, or replace cable on port 2 with USB 3.2-rated cable to support Tier 2." The suggestions are concrete and actionable rather than just identifying the problem.

### Continuous Validation

Cable capacity can change over time as cables degrade. The platform's background cable monitoring catches these changes; the routing graph validation uses the latest measured capacity rather than the original test result. A cable that was fine yesterday but is now showing degraded performance prompts a warning before the user starts a performance, not during it.

This connects the cable testing functionality from a manual diagnostic tool into a continuous validation system. Cable tests run periodically (every few minutes, on chain initialization, or whenever the user requests). The results feed the validation engine. The user gets warnings when configurations become problematic for reasons outside their control (cable degradation) as well as when they make configuration changes that exceed cable capacity.

The architectural property worth being explicit about: the platform takes the routing graph as the source of truth for what audio should flow where, takes the cable measurements as the source of truth for what each cable can carry, and validates the former against the latter. The user is responsible for fixing problems (replacing cables, reducing quality, adjusting routing); the platform is responsible for catching problems early and suggesting specific fixes. This is much better than the alternative of "configuration succeeds in software, fails mysteriously at runtime."

This validation also provides genuine value for the platform's positioning. Commercial pedal products do not do this kind of pre-flight validation; they just fail mysteriously when the cables cannot support the configuration. The platform's diagnostic capability becomes a real selling point for serious users who care about reliability.

### v2 Enhancements

Several more sophisticated tests are interesting but defer to v2:

**Bit error rate measurement** sends a pseudorandom pattern through the cable, with the receiver checking each bit against the expected pattern. Good cables have error rates below 10^-12 (essentially zero); marginal cables might be at 10^-6, which causes audible audio glitches. Implementation requires bypassing some of the USB protocol's automatic error recovery, which is feasible but more complex than v1 needs.

**Impedance characterization** through eye diagram analysis: sending signals at various frequencies and measuring signal integrity. Good differential pair impedance (90 ohms for USB) produces a clean eye diagram; impedance discontinuities cause distortion. The H7 cannot do full time-domain reflectometry but could do crude eye diagram analysis at lower speeds.

**Cable mechanical health monitoring** that detects intermittent connection failures by watching for brief enumeration events or USB error counter increments. A cable with degrading internal wiring shows transient failures even when not under load; tracking these failures over time identifies cables that are about to fail catastrophically.

### Audio Cable Testing (Separate Question)

Audio cable testing is a different question with different testable properties. The cables involved are TS and TRS analog cables carrying instrument signals, with failure modes that are fundamentally different from USB digital cables: continuity failures, intermittent contact, microphonic noise, capacitance loss in high frequencies, ground hum from poor shielding. The platform could plausibly do audio cable testing through the codec's ADC and the H7's DSP: send a known test signal out the jack, measure what comes back, and characterize the signal path. This would detect continuity failures, characterize frequency response, measure noise floor, and check for hum patterns. The implementation is more involved than the USB test because it requires generating test signals through the codec and analyzing them in the H7. For v1, the USB cable test is the higher priority because USB cable problems will appear more frequently in the wild. Audio cable testing is interesting v2 territory.

## Ethernet Transport Option

USB-C is the right primary transport for typical Metapedal use cases: short cable runs between adjacent pedals on a pedalboard, high bandwidth for multi-channel audio, ubiquitous availability, and modest cost. But USB has real limitations that are worth acknowledging: cable runs are limited to roughly 5 meters before signal integrity degrades, USB cables are mechanically less robust than the cabling used in professional installations, and USB does not integrate with the existing network infrastructure that pro AV facilities already have in place.

For installation and pro AV use cases, ethernet is a more natural transport. Cat 5e and Cat 6 cables are the most thoroughly engineered cabling system in the world, available at every price point, mechanically robust enough to survive being stepped on and run along walls, and supported by infrastructure (switches, patch panels, network management tools) that pro venues already have. The platform should support ethernet as an alternative transport for module variants positioned for these use cases.

### The H7's Ethernet Capability and the USB-Bridged Approach

The STM32H7's onboard Ethernet MAC is 10/100 Mbps only, not gigabit. This is a real constraint that shapes the platform's ethernet story. The H7 cannot natively reach gigabit speeds through its built-in MAC.

The architectural alternative is USB-bridged gigabit ethernet: a gigabit ethernet PHY connected to the H7 through a USB-to-ethernet bridge chip (like the Microchip LAN7430), with the H7 speaking USB to the bridge and the bridge speaking ethernet to the network. This is the same approach the Raspberry Pi 3 B+ used to get gigabit ethernet on a SoC that did not natively support it. The Pi 3 B+ delivered roughly half-gigabit throughput in practice because USB 2.0 high-speed (480 Mbps nominal, 350-400 Mbps practical after protocol overhead) was the bottleneck between the PHY and the SoC, not the ethernet signaling itself. The Pi Foundation made this compromise deliberately: users got most of the benefits of gigabit ethernet (long cable runs, infrastructure compatibility, autonegotiation with gigabit switches) without redesigning the SoC.

The same approach works on the H7. The H7's OTG HS USB controller drives a gigabit ethernet PHY through a USB-to-ethernet bridge chip. In ethernet-variant modules, the OTG HS controller is reassigned from inter-module USB bus to ethernet PHY interface, with the module's inter-module connection becoming ethernet rather than USB. The throughput ceiling is the USB high-speed ceiling (~350-400 Mbps practical), but this is meaningfully better than the H7's native 100 Mbps and is sufficient for the platform's installation and pro AV use cases.

The Pi 3 B+ precedent matters here because users will already understand the throughput profile. Anyone who has used a Pi 3 B+ knows that gigabit ethernet on it delivers roughly half-gigabit throughput. The same expectation translates to Metapedal: gigabit ethernet hardware with practical throughput of 350-400 Mbps, plus all the other benefits ethernet provides.

The Pi 4 resolved this properly with a redesigned SoC that has native gigabit MAC, and reliably hits 900+ Mbps in real-world throughput. Metapedal v2 will likely take a similar path, either by switching to a more capable MCU or by adding dedicated gigabit MAC silicon. v1 uses the USB-bridged approach and accepts the half-gigabit ceiling.

### Throughput Comparison Across Options

The throughput options for v1 modules:

Native 100 Mbps ethernet through the H7's onboard MAC:
- Practical application bandwidth: ~75-80 Mbps
- At 96 kHz / 24-bit: ~32 channels per direction
- Cheapest implementation (~$5-7 BOM addition)
- Sufficient for small-to-medium installations

USB-bridged gigabit ethernet through the H7's OTG HS:
- Practical application bandwidth: ~350-400 Mbps
- At 96 kHz / 24-bit: ~150-170 channels per direction
- At 192 kHz / 24-bit: ~75-85 channels per direction
- BOM addition ~$8-12 over standard variant
- Sufficient for very large installations including AES67-style audio-over-IP networks

USB-C native (the standard inter-module bus):
- Practical application bandwidth: ~350-400 Mbps
- Same bandwidth as USB-bridged gigabit
- No ethernet infrastructure benefits

The USB-bridged gigabit approach gives the platform substantially more capable ethernet than native 100 Mbps would, at a modest cost premium. For ethernet-variant modules, the USB-bridged approach is probably the right v1 choice because it delivers meaningful bandwidth advantages alongside the infrastructure benefits.

### What v1 Ethernet Provides

With the USB-bridged gigabit approach, ethernet variants provide:

Long cable runs that USB cannot support. USB cables are limited to roughly 5 meters. Cat 5e and Cat 6 cables support 100 meters at full bandwidth. For installations where modules need to be far apart, ethernet is the right choice.

Mechanical durability. RJ45 connectors are designed for thousands of mating cycles with positive latching that resists accidental disconnection. Cat cables are designed for installation abuse.

Network infrastructure integration. Ethernet plugs into the building's network. Compatible with AES67 audio over IP and similar pro audio networking protocols.

Power over Ethernet support. Standard PoE delivers 13W, PoE+ delivers 25W. Sufficient for a Metapedal module. Eliminates separate power runs in installations.

Multi-module connection through standard ethernet switches. A 5-port gigabit switch (around $20 retail) becomes a network hub for 5 modules. The switch handles physical-layer routing.

Substantial bandwidth at half-gigabit practical throughput. Sufficient for high-channel-count multi-tier audio configurations that 100 Mbps could not support.

### The Ethernet Variant Module Architecture

The right approach is to position ethernet as an option on specific module variants rather than as a universal feature. Standard A and B variants for pedalboard use continue to use USB-C exclusively. An ethernet variant of A and B modules uses an RJ45 port (replacing the inter-module USB-C ports) with the supporting USB-to-ethernet bridge chip, gigabit PHY, and magnetics.

The cost of adding USB-bridged gigabit ethernet to a variant is modest. The USB-to-ethernet bridge chip (Microchip LAN7430 or similar, $3-5), the gigabit PHY (often integrated into the bridge chip), the RJ45 magnetics connector ($1-2), and supporting passives ($0.50-1) add roughly $5-8 BOM. The PCB area is comparable to two USB-C ports plus their supporting components. So the ethernet variant has roughly $5-10 more BOM than the standard variant, retailing for perhaps $15-25 more.

Users buy the ethernet variant when they need ethernet's specific benefits: long cable runs, mechanical durability, network integration, PoE, switch-based topology, or the higher practical bandwidth that half-gigabit provides for very high channel counts. Users who do not need these benefits buy standard variants.

### The "Show Up at the Venue with a Switch" Scenario

A representative use case for v1 ethernet variants: a user arrives at a venue with four or five ethernet-variant Metapedal modules plus a $20 gigabit switch. They plug each module's RJ45 port into the switch. The switch handles physical-layer routing. The modules discover each other over the network and establish a chain topology, operating like a USB-connected chain but with ethernet as the underlying transport.

Each module-to-switch link runs at gigabit signaling but caps at the USB-bridged ceiling (~350-400 Mbps practical per link). The switch's gigabit capacity is more than sufficient for the aggregate traffic of 4-5 modules at this throughput. Users can run substantial multi-channel audio through the chain with comfortable headroom.

The platform's bandwidth validation logic knows that each ethernet link caps at ~350-400 Mbps and would warn if the configuration approaches or exceeds it. Most realistic configurations stay well within this ceiling.

### v2 Ethernet Story and MCU Options

v2 will likely bring native gigabit ethernet, with several specific MCU options now available or imminent that support it natively without USB bridging. The MCU landscape for high-performance, gigabit-capable parts has matured substantially in the past few years:

**NXP i.MX RT1170 family**: Cortex-M7 at 1 GHz plus a secondary Cortex-M4 at 400 MHz, dual native gigabit ethernet ports with AVB and TSN, mature ecosystem since 2021. Higher clock speed than the H7 (1 GHz versus 480 MHz). Up to 2 MB SRAM. Direct contender for v2 silicon and available in production volumes now.

**Renesas RA8P1**: Cortex-M85 at 1 GHz plus secondary Cortex-M33 at 250 MHz, native gigabit ethernet, Ethos-U55 NPU for AI/ML acceleration. The Cortex-M85 with Helium MVE provides substantial DSP acceleration over the M7; the NPU could enable neural amp modeling at quality competitive with commercial neural amp products. Available now at ~$20-28 in small quantities.

**STMicroelectronics STM32V8**: Announced November 2025, OEM availability Q1 2026. Cortex-M85 at 800 MHz with Helium MVE, 4 MB embedded PCM non-volatile memory, 1.5 MB ECC SRAM, native gigabit ethernet with TSN, USB HS/FS with PHYs included. Manufactured on 18nm FD-SOI process. 60% performance improvement over the H7 in CoreMarks. The natural ST successor to the H7 for high-performance applications, with the same vendor ecosystem the H7 uses. Pricing not yet disclosed but expected in the $20-40 range based on positioning.

All three options provide native gigabit ethernet without USB bridging, meaningful DSP performance improvements over the H7, and substantial RAM expansion. The choice between them is a matter of pricing, availability timing, ecosystem familiarity, and specific capabilities (the Renesas NPU is unique to that option; the ST V8's PCM memory is unique to that option).

For Metapedal's v2 architecture, the natural choice is probably the STM32V8 because it maintains continuity with ST's ecosystem that v1 already uses. Existing v1 firmware code, libraries, and tools largely port over. The V8 is positioned as a non-pin-compatible successor to the H7 (so v2 motherboards need new PCB layouts) but uses the same STM32Cube development framework and similar peripheral architectures. This minimizes the transition cost for v1-to-v2 development.

The NXP and Renesas options are viable alternatives if pricing or availability favors them, or if specific capabilities (NPU for neural processing on the Renesas, dual gigabit ethernet on the NXP) match the v2 use cases better than the V8's specific feature set.

With native gigabit ethernet on any of these MCUs, v2 ethernet variants reach full 900+ Mbps practical throughput, supporting AES67 / AVB / Dante native operation with proper deterministic timing through TSN. The 2.5 Gbps and 5 Gbps ethernet standards (running on Cat 6 and Cat 6a cables) become feasible. The v2 ethernet variants become the natural choice for serious pro AV installations, professional recording studios, and high-channel-count multi-room setups.

For v1, the USB-bridged half-gigabit ethernet variants on the H7 serve installation and pro use cases that benefit from ethernet's infrastructure properties plus substantial bandwidth (~400 Mbps practical), with honest documentation about the half-gigabit ceiling. Users who need full line-rate gigabit wait for v2.

## End-to-End Latency and the Physical Plan

Latency is one of the platform's harder architectural challenges. Live performance has firm tolerances: 10 milliseconds end-to-end is "you cannot really tell," 20 milliseconds is "starting to feel a bit weird," 30 milliseconds is "noticeable lag, hard to perform with," and 50 milliseconds or more is "you cannot play tightly with this." A multi-module chain accumulates latency from several sources, and the architecture needs to be explicit about where that latency comes from and how it gets managed.

### Latency Sources

The fundamental latency sources in a Metapedal chain are:

**ADC conversion** at the source module's codec. Modern audio codecs use sigma-delta converters with decimation filters that introduce group delay. Typical group delay at 48 kHz is around 20-40 samples, which is 0.4-0.8 ms per conversion. Every audio signal pays this on the way in.

**DAC conversion** at the output module's codec. Similar group delay as ADC, 0.4-0.8 ms per conversion. Every audio signal pays this on the way out.

**DSP buffer period** at each module that processes audio. The buffer size is a fundamental tradeoff. Small buffers (32 samples = 0.67 ms at 48 kHz) give low latency but high CPU overhead from interrupt handling and protocol housekeeping. Large buffers (256 samples = 5.3 ms) give plenty of CPU headroom but add buffer-period delay to latency. A reasonable working buffer size for typical use is 64-128 samples (1.3-2.7 ms per buffer). Each module that touches audio adds one buffer period to total latency.

**Inter-module USB transit** between modules. USB high-speed has 250 microseconds of intrinsic isochronous transfer latency from the protocol itself. With buffer-based audio transfer in the platform's custom firmware, the actual per-hop latency is the protocol minimum plus some buffer scheduling overhead, perhaps 1-2 ms per hop in typical operation. This is substantially less than what OS-level USB audio achieves (where Jack and similar audio servers add 2-4 ms per hop because of OS scheduling), because the platform controls both ends of every link and can optimize the scheduling.

**Algorithm-inherent latency** for specific DSP operations. Look-ahead compressors add their look-ahead window (typically 5-10 ms). FFT-based effects add the FFT window size (typically 256 or 512 samples, 5-11 ms). Convolution reverbs add the FFT processing latency for partitioned convolution (typically 64-256 samples). Pitch shifters using PSOLA add their analysis window latency (often 10-30 ms for natural-sounding results). These are properties of the algorithm itself, not the platform, and cannot be reduced below their algorithmic minimum.

### Sample Latency Budgets

For a 4-module linear chain with light processing at the source module only:

- Source ADC: 0.6 ms
- Source DSP buffer: 2 ms
- USB hop 1→2: 1.5 ms
- USB hop 2→3: 1.5 ms
- USB hop 3→4: 1.5 ms
- Output mix-down DSP buffer: 2 ms
- Output DAC: 0.6 ms
- **Total: 9.7 ms**

This is comfortably under the 10 ms "you cannot tell" threshold and is the baseline the platform should achieve for typical live-performance chains.

For the same chain with heavy DSP processing (FFT-based pitch shift or convolution reverb) at one middle module:

- Source ADC: 0.6 ms
- Source DSP buffer: 2 ms
- USB hop: 1.5 ms
- Heavy DSP module: 2 ms buffer + 10 ms algorithm look-ahead = 12 ms
- USB hop: 1.5 ms
- USB hop: 1.5 ms
- Output DSP buffer: 2 ms
- Output DAC: 0.6 ms
- **Total: 21.7 ms**

The algorithm-inherent latency from the heavy effect is the dominant contributor. Adding 10 ms of look-ahead to the chain raises end-to-end latency from 9.7 ms to 21.7 ms regardless of where the effect is placed. The platform cannot make a look-ahead compressor or FFT-based pitch shifter latency-free; it can only manage where the latency is paid.

### The Physical Plan Concept

The routing graph specifies what audio should flow where (the user's intent). The platform's runtime needs to figure out how to deliver that intent on the actual hardware (the physical plan). This is the same pattern as relational database query planning: declarative SQL specifies what data the user wants, the query planner figures out the most efficient way to compute it.

For Metapedal, the physical plan takes as input the routing graph plus the hardware reality (cable measurements, module DSP budgets, latency requirements per flow, DSP algorithm catalog with cycle costs and inherent latencies) and produces the placement of each DSP node on specific modules, the buffer sizes for each section, the flow routing through specific cables, predicted latency per flow, predicted CPU and RAM utilization per module, warnings if any flow's latency requirements cannot be met, and suggestions for the user to modify the configuration if needed.

The architectural insight is that the routing graph is intent and the physical plan is the realized DSP topology. They are separate concerns with separate optimization criteria. The same routing graph might compile to different physical plans depending on what hardware is available, what other configurations are sharing the chain, and what latency requirements are most important.

### Specific Physical Plan Optimizations

The query planner can apply several specific optimizations:

**Reducing buffer-period traversals.** If a signal needs processing at modules 1, 2, and 4 in a 4-module chain, each module adds buffer-period latency. If the processing can be consolidated to fewer modules (all of it on module 3, for instance), total latency drops by the difference. The planner identifies opportunities to consolidate.

**Avoiding round trips.** A naive routing might send audio from module 1 to module 4 (for processing) and back to module 2 (for something else). Each round trip adds USB hop latency. The planner detects these patterns and either reorders operations or warns the user.

**Allocating to modules with headroom.** A heavy effect on an already-loaded module might require a longer buffer period to fit everything. Moving the heavy effect to a less-loaded module lets that module use a smaller buffer, reducing latency.

**Identifying the critical path.** In a multi-source mix-down scenario, the bottleneck is the longest path from any source to the output. The planner identifies the critical path and targets optimizations specifically at it, rather than wasting effort optimizing paths that already have latency headroom.

**Trading bandwidth for latency.** The planner can decide whether to process audio at the source (using local DSP but transmitting processed audio downstream) versus transmit raw audio and process downstream. For high-quality-tier flows where bandwidth is precious, source-local processing might be better. For low-bandwidth scenarios where the source is overloaded, transmit-then-process might be better.

**Mixed buffer sizes.** Some effects need large buffers (FFT-based) while others can use small buffers (filters, simple gain, distortion). The planner can arrange different buffer sizes for different sections of the chain, with the large-buffer sections being the parts that need that latency anyway, and the small-buffer sections being kept tight to minimize total latency.

**Consolidating shared effects.** If two performers both want their signals processed through the same reverb, the reverb should run once on a single module rather than twice. The planner identifies shared effects and consolidates them.

**Source-local light processing plus downstream heavy processing.** For multi-source mix-down with per-source effects, each performer's light effects run on a module near them; the mix-down module handles the master effects after the mix. This minimizes the latency for each source's processed signal to reach the mix-down while applying master effects to the combined signal where they are conceptually wanted.

### When the Naive Rule Helps and When It Does Not

Your initial intuition about "moving latency-expensive DSP downstream" needs careful application. The naive rule does not always reduce latency.

In a linear chain where audio already flows through every module, moving processing from one module to another does not eliminate the buffer-period at any module that the signal passes through. If the signal already passes through modules 1, 2, 3, and 4, putting heavy processing at module 3 versus module 1 makes no difference to the total buffer-period accumulation; both placements have the signal traversing all four modules.

Where the rule helps is when consolidation or critical-path optimization is possible. Specifically:

If module 1 has too much load to fit the heavy effect at all, moving it elsewhere is necessary, not just an optimization. The planner picks the closest module with capacity to minimize additional buffer traversals.

If module 1 currently has light load and the heavy effect would force a larger buffer period there, moving the heavy effect to a module that can fit it with a smaller buffer reduces the source module's contribution. Net latency saving depends on whether the savings exceed the cost of the additional buffer-period traversal where the heavy effect ends up.

If the heavy effect is one that applies to the master mix (master compressor, master EQ, master reverb), placing it on the mix-down module is conceptually correct and latency-neutral (since the mix-down module's buffer period is already in the budget regardless).

The planner makes these decisions automatically based on the specific configuration. The user expresses intent through the routing graph; the planner figures out the optimal physical realization.

### User-Visible Latency Information

The web UI shows predicted latency per flow as part of the routing graph editor. Each source-to-output path has a calculated end-to-end latency that updates as the user modifies the configuration. Latency over thresholds (10 ms, 20 ms, 30 ms) shows visual warnings. The user can specify per-flow latency requirements ("this guitar flow must be under 12 ms") and the planner enforces them or explains why they cannot be met.

When the planner cannot meet a latency requirement, it suggests specific remediations: reduce the heavy effect's look-ahead window if the algorithm allows, move processing to a different module, eliminate redundant effects, switch to a simpler algorithm with less inherent latency, or add more modules to the chain to distribute the load with shorter critical paths.

This is the same architectural pattern as the bandwidth validation we discussed for cable testing: the platform validates that the user's intent is deliverable on the actual hardware, with specific diagnostics and suggestions when it is not. The latency validator and the bandwidth validator share much of the same infrastructure and produce similar user-facing diagnostic experiences.

## Chain Topology Versus Star Topology

The default architecture we have been describing is a linear chain: modules connect to their neighbors through USB-C cables, with each module communicating with the modules directly adjacent to it. This works well for typical pedalboard use cases where modules are physically arranged in a line and users want simple plug-and-play connections without external hubs.

For some use cases, particularly multi-source mix-down scenarios, a star topology produces better latency characteristics. In a star topology, all modules connect to a central hub rather than to each other directly. Audio flowing between modules traverses two hops (source → hub → destination) regardless of the modules' physical positions, eliminating the latency-inequality problem that chains create for far-from-the-output modules.

### The Latency Comparison

For a 4-module configuration, the difference between chain and star topologies:

Linear chain, signal from module 1 to module 4 (mix-down output): 3 USB hops × 1.5 ms = 4.5 ms inter-module transit. The performer at module 1 has the highest latency in the chain.

Linear chain, signal from module 3 to module 4 (mix-down output): 1 USB hop = 1.5 ms inter-module transit. The performer at module 3 has lower latency than the module 1 performer.

Star topology, signal from any module to mix-down output: 2 USB hops (module → hub → output module) = 3 ms inter-module transit. Every performer has the same latency.

The star saves 1.5 ms for the worst-case performer in a 4-module chain. For 8-module configurations, the chain's worst case is 10.5 ms of transit while the star is still 3 ms; the savings scale with module count.

The deeper benefit is latency equalization. Performers in a band are sensitive to relative latency between members; if the drummer hears their kick at 8 ms latency but the bassist hears the same kick at 12 ms, the rhythm section feels misaligned even though both latencies are individually fine. Star topology makes the latency identical across performers, which is a significant ergonomic improvement for multi-performer chains.

### What the Star Topology Requires

USB is fundamentally host-centric: standard USB 2.0 has exactly one host and a tree of devices, with the host initiating all communication. Peer-to-peer USB does not exist in the standard. For star topology, the platform needs either a multi-host hub (specialty hardware, not commodity) or one module designated as the central hub-controller that all others connect through.

The right architecture for v1 is asymmetric star: one module (typically the A variant, which is already designated as the chain leader role) acts as the USB host, with a standard USB hub plugged into its USB-C port. The other modules (the B variants) connect to the hub and present themselves as USB devices to the A variant host.

The hub is commodity hardware: any standard USB-C hub with USB 2.0 high-speed support works. These are widely available at $15-30 retail. The platform does not need a specialty multi-host hub; the asymmetric star sidesteps that requirement by designating one specific module as the host.

### The B Variant's USB Role

In a star topology, the B variants act as USB devices rather than as hosts. Their USB-C inter-module ports connect to the central hub rather than to neighboring modules. The B variant's firmware acts as a USB Audio Class compliant device for the audio streams plus a custom interface for the inter-module protocol (control messages, configuration, presets, etc).

This is actually slightly simpler firmware-wise than the chain topology, where each module had to negotiate host/device roles with its neighbors through OTG mode. In star, the role is fixed: A variant is host, B variants are devices. The OTG complexity goes away for the B variants.

The B variants can still be used in chain topology when the user prefers (by connecting them to neighboring modules instead of to the central hub). The firmware supports both modes; the topology is determined by how the user physically connects the modules.

### When to Use Which Topology

The choice between chain and star depends on the use case:

**Chain topology is preferred for:** solo pedalboard use cases where the user wants minimal cabling and no external hub; small multi-module configurations (2-3 modules) where the latency difference is negligible; configurations where the modules are physically arranged in a line and chaining matches the geometric layout; users who do not want to manage an external hub as additional gear to transport.

**Star topology is preferred for:** multi-performer chains where latency equalization across performers matters; mix-down scenarios with 4+ modules where the chain's worst-case latency is meaningful; installations where the central hub fits naturally into the space and users do not need to think about it; configurations where modules are physically spread out (different positions on a stage, different rooms in a recording studio) and central connection through a hub fits the geometry better than chaining.

For most pedalboard users, chain topology is the right choice. For most band and installation users, star topology is the right choice. The platform supports both, with documentation explaining when each is preferred.

### The Query Planner and Topology

The query planner takes topology into account when computing the physical plan. The latency calculations differ between chain and star (different transit costs for different paths), and the planner optimizes for whichever topology is in use. A user switching from chain to star or back gets the planner's recommendations updated to reflect the new topology.

The query planner can also identify topology-related optimization opportunities. In a chain topology where the user wants a multi-source mix-down, the planner might suggest moving to star topology with the specific advice: "Your current chain topology gives different latencies to different performers (8 ms for module 3, 11 ms for module 1). Star topology would equalize this to 9 ms for all performers. Consider connecting your modules through a central USB hub instead of in a chain."

This is the kind of intent-versus-implementation separation that the physical plan abstraction enables. The user expresses what they want (multi-source mix-down with equal latency for all performers); the planner figures out the best topology to deliver it; the user makes informed decisions about whether to adopt the suggestion.

### Hybrid Topologies (v2+)

For v1, the platform supports pure chain and pure star topologies. Hybrid topologies (some modules in chain, others hub-connected) are technically feasible but add user confusion and firmware complexity. v2 might support hybrid topologies for users who want maximum flexibility, but v1 keeps the topology choice simple.

### Ethernet Star as the Latency Champion

The ethernet variant of modules we discussed earlier turns out to be the lowest-latency topology option for multi-module configurations, which is worth being explicit about because it changes the platform's positioning for latency-critical use cases.

Ethernet switches are inherently star-topology devices, and ethernet audio protocols are specifically engineered for low-latency networked audio. Published latency numbers for professional networked audio over 100BASE-T are remarkable: AVB networks reach 2 milliseconds maximum across 7 hops, Dante reaches 1 millisecond across 10 hops, and AES67 has minimum latency of 125 microseconds. These numbers are achieved through direct hardware paths, precise time synchronization through PTP (IEEE 1588), and dedicated network stacks.

For Metapedal with custom firmware on both ends of every link, the achievable per-hop latency on ethernet is similar: roughly 100-300 microseconds per switch hop in commodity gigabit switches running 100 Mbps. The architectural advantage is the topology itself: every module connects to the central switch, so any source-to-destination path is two hops (source → switch → destination) regardless of physical position. There is no "chain" creating latency inequality.

### Latency Comparison Across Topologies

For a 4-module configuration with mix-down at one module:

**Linear USB chain, worst-case path:**
- Source ADC: 0.6 ms
- Source DSP buffer: 2 ms
- 3 USB hops: 4.5 ms (1.5 ms each)
- Output DSP buffer: 2 ms
- Output DAC: 0.6 ms
- **Total: ~9.7 ms**

**Asymmetric USB star with hub:**
- Source ADC: 0.6 ms
- Source DSP buffer: 2 ms
- 2 USB hops (through hub): 3 ms
- Output DSP buffer: 2 ms
- Output DAC: 0.6 ms
- **Total: ~8.2 ms**

**Ethernet star with gigabit switch (running 100 Mbps for v1):**
- Source ADC: 0.6 ms
- Source DSP buffer: 2 ms
- Switch transit (source → switch → destination): ~0.5 ms total
- Output DSP buffer: 2 ms
- Output DAC: 0.6 ms
- **Total: ~5.7 ms**

Ethernet wins by ~4 ms over the USB chain and ~2.5 ms over the USB hub-based star. For multi-performer mix-down scenarios where latency equalization matters, ethernet is the latency champion among the v1 topology options.

### Bandwidth Headroom in v1 Use Cases

100 Mbps ethernet sounds like it might be a meaningful constraint compared to USB high-speed's 480 Mbps, but for the realistic v1 customer personas the bandwidth headroom is actually generous. The platform's v1 personas are solo pedalboard users (1-3 modules, low channel count), small bands (3-6 performers, 2-4 channels each), small recording setups, solo electronic musicians, and hobbyists. None of these realistic users approach the 100 Mbps ceiling.

Let me run the numbers. Each performer sends roughly 2-4 channels (a stereo pair for instruments, a single mono channel for vocals, perhaps both for performers playing instruments while singing). At 48 kHz / 24-bit per channel with 15% USB protocol overhead, each channel needs roughly 1.3 Mbps. So:

3 performers × 4 channels: 15.6 Mbps
5 performers × 4 channels: 26 Mbps
6 performers × 4 channels: 31 Mbps

Adding monitor mixes (each performer receives 2 channels of personalized monitor mix as additional traffic):

3 performers, 4 channels send + 2 channels receive each: 23 Mbps
5 performers similarly: 39 Mbps
6 performers similarly: 47 Mbps

All comfortably within 100 Mbps with substantial headroom. Even 6 performers with 6 channels each (sending plus receiving) is well under half the ethernet capacity. The 100 Mbps ceiling becomes a real constraint only for installation-class configurations (large theaters, multi-room venues, broadcast facilities) which are firmly v2+ territory anyway because they also need things like AES67 compatibility and redundancy.

**The architectural insight worth being explicit about:** the platform's transmission bandwidth and recording bandwidth are decoupled. The performance-quality audio flowing between modules is 48 kHz / 24-bit (which is excellent and sufficient for live performance), while each module can independently record at higher quality to its local microSD card. A B variant module can capture its input at 192 kHz / 32-bit float to local storage while transmitting only the 48 kHz / 24-bit version to the rest of the chain. The recording does not cross any inter-module link, so it does not consume any inter-module bandwidth.

This decoupling is a real architectural advantage. The user gets maximum-quality stems for later mixing and mastering without paying any inter-module bandwidth cost for that quality. The platform's recording use case (capturing stems at the source for studio-quality post-production) is essentially free compared to the live performance configuration.

### The Bandwidth Headroom Enables Real Features

The substantial 100 Mbps headroom on ethernet enables capabilities that go beyond just moving audio around:

**Real-time stem distribution.** Each performer's signal can flow not just to the mix-down module but to a dedicated stem-recording module or USB audio interface module sending stems to a laptop DAW. With 100 Mbps headroom, this is comfortable for typical performer counts.

**Click track distribution.** A drummer's click track flows as a dedicated channel to all performers' monitor mixes, with each performer's mix able to include or exclude the click independently.

**Talkback channels.** Performers can have talkback to each other or to the sound engineer through dedicated low-bandwidth channels for spoken communication during quiet moments. Negligible bandwidth (under 1 Mbps per talkback channel).

**Tempo and MIDI distribution.** Tempo information from a drummer's foot-tap flows to other performers' modules so that delay effects, modulation effects, and synth sequences stay in sync with the drummer. Just a few bytes per beat, essentially zero bandwidth.

**Cue audio for performers offstage.** A performer about to come on stage can hear what is happening on stage through their monitor mix before connecting their instruments. Cue audio is just another routing destination.

All of these features fit comfortably in 100 Mbps for any realistic v1 configuration. The platform's bandwidth headroom enables capabilities that traditional analog pedalboards cannot provide and that commercial digital mixers charge significant money for.

### Where USB and Ethernet Each Win

The choice between USB high-speed and ethernet is therefore not about bandwidth (both are plenty for v1 use cases) but about other architectural properties:

**USB chain wins for:**
- Simplicity (no external switch needed)
- Cost (no external hardware to buy)
- Setup speed (just connect cables between modules)
- Solo and small-rig pedalboard configurations
- Cost-sensitive configurations

**Ethernet star wins for:**
- Latency equalization across performers (every performer same latency to mix-down)
- Lower absolute latency in multi-module configurations (~4 ms savings versus USB chain)
- Long cable runs (up to 100 meters versus USB's 5-meter limit)
- Mechanical durability (RJ45 connectors survive stage abuse better than USB-C)
- Network infrastructure integration (plugs into building networks)
- Power over Ethernet support (eliminates separate power runs)
- Physically distributed configurations (modules spread across different positions)

The choice is genuinely use-case-dependent, not bandwidth-dependent. The platform should not apologize for the 100 Mbps ethernet ceiling; the ceiling is well above what v1 users actually need.

### Decision Framework

The user's decision between USB chain, USB asymmetric star, and ethernet star depends on the use case characteristics:

**Solo guitarist with 1-3 modules:** USB chain is right. Simple, cheap, no external hardware. Latency is fine because there are few hops. Bandwidth is not even close to a concern.

**Solo guitarist with 4+ modules:** USB chain is still probably right unless the user does heavy mix-down work. The latency is acceptable and the simplicity is worth it.

**Small band (3-6 performers) with mix-down:** ethernet star is right. The ~4 ms latency improvement over USB chain is meaningful for rhythm sections. 100 Mbps is more than enough for any realistic band size.

**Live performance with physically distributed performers:** ethernet star is right. The cable infrastructure benefits (long runs, durability) combined with latency equalization make ethernet the clear choice. Bandwidth is comfortable for any realistic configuration.

**Recording/installation use case with high channel counts:** ethernet star at 100 Mbps still works for most cases. The few configurations that exceed 100 Mbps (very high channel counts at high quality tiers) are uncommon in v1 use cases. v2 will bring native gigabit for these specific configurations.

**Live touring with quick setup priority:** USB chain or asymmetric star. Ethernet adds an external switch to the gear list, which adds setup time and another point of failure.

The platform supports all of these topologies; the user picks based on what their use case actually needs. The web UI's configuration tool can help with this decision by showing predicted latency and the topology recommendation for the user's specific routing graph, letting them see the tradeoffs concretely before committing.

### The Query Planner Becomes More Valuable

The query planner becomes substantially more useful when the platform supports multiple topology options. For a given routing graph, the planner can compute the latency profile for chain, asymmetric star, and ethernet star topologies, and recommend the topology that best fits the user's stated requirements. The recommendation comes with concrete numbers: "Your configured chain topology gives 11 ms worst-case latency; ethernet star would give 6 ms. The ethernet star requires a $20 gigabit switch and ethernet-variant modules; the additional cost is $120 retail for 3 ethernet-variant modules. Would you like to see the upgrade path?"

This is the same intent-versus-implementation separation working at the topology level. The user expresses what they want; the planner figures out the best topology to deliver it; the user makes informed decisions about cost and effort tradeoffs.

## The Bus Protocol and Leader Election

A chain of Metapedal modules connected over the inter-module bus needs some way to coordinate. Several functions across the chain require a single decision-making module: the master clock for audio synchronization comes from one source, the preset state needs a single authoritative copy that can be reliably replicated, and chain-wide configuration changes need a coordination point. These do not all have to be the same module, but each one needs to be resolved unambiguously, in a way that survives modules being added, removed, or failing during use.

The mechanism for resolving this is a lease-based leader election protocol. One module on the bus holds the leader role at any time. The leader heartbeats periodically to assert its continued presence. As long as the heartbeats continue, the leader keeps the role regardless of what other modules are on the bus. When the heartbeats stop, the remaining modules run an election to choose a new leader. This is a well-established pattern in distributed systems and it has the operational properties that Metapedal needs.

The lease-based approach is critical for user experience reasons. A naive pure-lowest-identifier-wins election would trigger a new leadership transition every time a module with a lower identifier joined the chain. Each transition causes a brief audio glitch as the clock master changes, and various small disruptions as the new leader takes over coordination tasks. This would mean the user could never plug in a new module without potentially disrupting their chain, which is unacceptable. The lease makes leadership sticky: once a module is leader, it stays leader until something happens to it, regardless of what other modules appear on the bus.

The election logic itself uses lowest-identifier-wins among the candidates, but the candidates are evaluated only when an actual election is happening, not continuously. The identifier is a hash of the module's chip serial number, giving a compact but effectively unique identifier per module. STM32 microcontrollers have factory-programmed unique serial numbers accessible to the firmware, which provides the source for these identifiers without requiring any external uniqueness authority. The hash reduces the ninety-six-bit STM32 serial number down to a more compact thirty-two or sixty-four-bit identifier for use in protocol traffic.

Module priority is also part of the election input, taking precedence over identifier. A module with a lower priority value wins over a module with a higher priority value, and identifier comparison applies only within the same priority. The default priority value is the same for all modules, so by default the election is purely identifier-based. The priority becomes meaningful when the user configures a preferred leader through the phone app or on-device interface, lowering that module's priority value below the default. The preferred module then wins any election it participates in, regardless of identifiers. Modules that are running heavy DSP near their capacity can also raise their own priority to discourage being elected leader, since the leader does additional coordination work that the heavily-loaded module might not be able to handle.

The protocol flow on bus initialization is as follows. When modules are first connected and powered, they enter a brief settling period during which they announce their presence and observe announcements from other modules. The settling period lets all modules on the bus become aware of each other before any election runs. After settling, the modules run an initial election. The winner claims the lease and begins heartbeating. The heartbeat message contains the leader's identifier, the lease expiration time, and various status information that the followers need to maintain their model of the chain. From this point forward, the lease holder remains leader as long as it keeps renewing the lease.

When a new module joins later, it observes the heartbeat traffic and learns who the current leader is. It joins as a follower and accepts the existing leader without challenge, regardless of its own identifier. This is what makes the leadership sticky. The new module participates fully in the chain's operations but does not attempt to take over the coordination role from the established leader. Stability is preserved.

When the leader fails or disconnects, the followers stop receiving heartbeats. The protocol uses multiple consecutive missed heartbeats as the failure criterion rather than a single missed one. Three missed heartbeats in a row is a much stronger signal than one missed heartbeat, and it gives the protocol tolerance for transient bus noise without losing the ability to detect real failures within a reasonable time. With heartbeats every five hundred milliseconds and a three-miss threshold, the failure detection takes about two seconds, which is acceptable for the audio synchronization requirements while being tolerant of glitches.

Once the followers conclude the leader is gone, they run a fresh election. Each follower begins announcing its identifier and priority as a claim for the leadership role. The protocol handles the race where multiple modules simultaneously try to claim by having claim messages include the claimant's identifier and having modules accept only the lowest-priority and then lowest-identifier claim they have observed. After a brief settling period, exactly one module holds the lease and the others have accepted it. The new leader begins heartbeating and chain operations resume.

There is a graceful handoff mechanism for planned leadership transfers. The current leader can send a message saying "I am stepping down" which causes the lease to expire immediately and triggers a fresh election. This is useful for several scenarios. The user might reconfigure their priority preferences, and the chain should adopt the new preference without waiting for the current leader to fail. The user might want to unplug the current leader module for some maintenance reason, and the chain should elect a new leader before the original leader is removed. All of these are supported through the explicit handoff.

The handoff also enables clean state transfer between the outgoing and incoming leaders. In an unplanned failure the incoming leader inherits whatever replicated state was already being maintained across the chain, with anything in flight at the moment of failure being lost. In a planned handoff, the outgoing leader can transfer its complete state explicitly, including any pending operations that have not yet been replicated. The user interface should prefer handoffs to forced transitions where possible.

The preset state and other chain-wide configuration are replicated across all modules in real time rather than being held only by the leader. When the leader changes any chain-wide state, the change is propagated to all modules immediately. Each module stores a current copy locally. When the leader fails and a new leader is elected, the new leader does not have to fetch state from anywhere because it already has a current copy. This makes leader failover fast and clean. The replication traffic is modest because chain-wide state changes are infrequent compared to audio, and the state itself is small.

There are several edge cases that the protocol handles. When the leader temporarily loses bus connectivity and then regains it, perhaps because of a brief cable disconnection, the original leader observes the bus state for a few heartbeat intervals before asserting any leadership claim. If during this period it sees another module claiming leadership, the original leader accepts that and becomes a follower. If it sees no claims, it can claim the lease for itself. This handles the reconnection case without causing confusion.

When the bus topology is changing rapidly, perhaps because the user is actively connecting or disconnecting modules in quick succession, the protocol defers leader election decisions until the topology has been stable for some short period. This prevents thrashing where the chain repeatedly elects new leaders during the user's reconfiguration session. The user sees stable behavior even during messy reconfiguration.

The clock master role specifically has additional subtlety because audio synchronization requires not just identifying the master but maintaining phase continuity through transitions. When the master changes, the followers must lock to the new master's clock, and the transition causes a brief glitch as the followers adjust. The mitigation is to have followers track the master's phase continuously and to remember the phase when the master fails, so that the new master can start its clock at the same phase the old master was at when it failed. This makes the transition smoother, though there is still some audible disruption that cannot be entirely avoided in an unplanned failover. Planned handoffs can achieve phase continuity completely if the handoff message includes the current phase information.

There is a separate concept worth distinguishing, which is position in the chain. The leader is one module with a chain-wide coordination role. The position of a module is its place in the signal chain, determined by the topology of how modules are physically connected. Position is local information that each module computes from its own connections, without requiring any negotiation. Position is used for preset addressing, where presets refer to "the first Processor in the chain" or similar by position, so that presets can be portable between similar chains with different physical modules.

The combination of identifier (which uniquely names a module across all possible chains), position (which locates a module within the current chain), priority (which influences leader election), and the lease-based protocol (which provides sticky leadership with clean failover) gives the platform the coordination infrastructure it needs.

One specific chain-wide protocol message worth being explicit about is the identify command, which solves a real usability problem the platform's flexibility creates. The flexibility means modules are visually indistinguishable from their physical appearance: two modules might look identical but be configured to run completely different effects. The identify command is a broadcast message, sent by any module in the chain and forwarded to all others through the streaming router's chain-wide message distribution, that causes every module to display its user-assigned nickname on its OLED simultaneously. The displays show large at-a-glance text optimized for floor-level readability. After a timeout, typically five to ten seconds, the displays return to their normal content. The user gets a snapshot of all module identities mapped to physical positions, which is the fastest way to confirm which module is which.

The identify command can also carry additional information beyond just the nickname. The display during identify mode might show the current preset name, the assigned effect type, the bypass state, and a visual indicator that confirms the module is communicating correctly. This turns identify into a quick chain-wide status check rather than just a name announcement, useful for both routine identification and troubleshooting.

The trigger for the identify command is mapped through the routing graph like any other action. The user assigns it to whatever input makes sense for their setup: a long-press of a footswitch, a button combination on the phone app, a voice command through the microphone input, or a dedicated identify peripheral. The protocol leader from the lease-based election serves as the natural coordinator for the identify command, ensuring all modules display their names at the same time and clear them at the same time.

The multi-performer chain case from version two does not require any changes to this protocol. A larger chain with more modules just runs the same election among the larger participant set, with the same lease-based stickiness applying. The bandleader role from the multi-performer feature is a separate concept layered on top of the protocol-level leader. The protocol leader is a technical role determined by the election. The bandleader is a social role designated by user preference. They can be the same module or different modules, and the architecture handles both cases uniformly.

## Bandwidth Analysis

The bandwidth available on the inter-module bus is more than sufficient for any realistic use case. USB 2.0 high-speed at 480 megabits nominal gives around 300 megabits of useful application throughput after protocol overhead. A single twenty-four-bit stereo audio stream at 96 kilohertz is about five and a half megabits per second with overhead, less than two percent of available bandwidth. Even ten such streams in parallel use less than twenty percent. The wire-level bandwidth is not the limiting factor in any scenario this platform is designed for.

The actual constraints are CPU load, buffer memory, protocol complexity, and the user interface for configuring routings. These all become real engineering problems as channel counts scale up, but bandwidth itself is not.

This means multi-performer chains are bandwidth-feasible even at high channel counts. Four performers with stereo signals at 96 kilohertz is eight stereo streams, around 45 megabits per second of payload, well within capability. Cue audio streams are mono and at lower rates, using a few hundred kilobits per second per stream, which is negligible.

## USB Host Mode and Standard Device Classes

When a Metapedal module is plugged into a host computer via USB-C, it should look like a standard USB audio device and a standard USB HID or MIDI device, with no drivers required, on every major operating system. This is huge for the product story. A single Metapedal module by itself becomes useful gear even for someone who has not bought into the larger ecosystem.

The technical mechanism is USB composite devices. A single USB device presents multiple interfaces simultaneously, and the host operating system enumerates each one independently. A Processor module with audio input and output presents a USB Audio Class interface, giving driver-free audio I/O on macOS, Windows, Linux, iOS, and Android. The same module simultaneously presents a USB MIDI interface for its control surface, so the same physical box shows up as both an audio interface and a control device. The STM32H7 supports composite USB devices natively, with reasonable software stacks available from ST and from open-source projects like TinyUSB.

The control surface presentation is most pragmatically a combination of MIDI and HID. USB Audio Class 2.0 includes MIDI as a standard sub-protocol, which means the module works with every DAW out of the box with no configuration. This gives universal compatibility at the cost of MIDI's resolution limits. HID is more flexible and higher resolution but requires the host application to know how to consume the specific HID descriptor. The module presents both interfaces, and the host software picks whichever it prefers.

The routing options when connected to a host are several. The guitar input can be digitized and sent to the host raw, with the host applying effects in software. The module can apply onboard effects first and send the processed signal to the host. Or the module can send the dry and processed signals simultaneously on separate USB audio channels, letting the user record both. This DI-plus-amp workflow is exactly what high-end audio interfaces support and what recording guitarists value enormously. The specification supports all three modes and lets the user choose.

When the module is part of a chain, the USB role is held by whichever module's USB-C port is connected to the host. Other modules in the chain treat that port as just another inter-module connection. This requires every module's firmware to act as either end of the USB connection depending on context, which is real firmware work but is the right answer for user experience. The user just plugs the laptop into whichever module is convenient, and the chain figures out the rest.

Power becomes a consideration when the chain is host-powered. A laptop USB port may not provide enough power for a chain of modules. USB Power Delivery negotiation allows up to 100 watts but laptops vary in what they actually deliver. The specification should handle insufficient power gracefully, perhaps by powering down modules in defined priority order or allowing an external power supply alongside the USB-C host connection.

Firmware updates use the standard USB DFU protocol that the STM32H7 supports natively. Plug the module into a computer, run an updater application, and the module flashes itself in DFU mode. This is the right way to do firmware updates for this product class and costs essentially nothing because the silicon already supports it.
