# Metapedal Firmware and Protocol Development Plan

This document lays out a development plan for the Metapedal firmware and the inter-module protocols. It is written with specific awareness of the constraints under which the work will actually happen: a time-constrained software engineer working in fragmented sessions, possibly with a hardware collaborator handling the PCB design, building toward a platform that does not yet have final hardware. The plan favors work that can happen before hardware exists, decomposes into pieces small enough to make progress on in fragmented time, and sequences toward demonstrable results early to sustain motivation.

## Guiding Principles

**Build vertically, not horizontally.** The temptation is to perfect each layer before moving to the next: complete the hardware abstraction, then the audio pipeline, then the features. This delays the first demonstrable result for a long time, which is fatal for a motivation-constrained project. Instead, build a thin vertical slice through all the layers first (just enough to get audio in, run one trivial effect, and get audio out), then thicken the slice over time. The thin slice validates the architecture end-to-end and produces a working module early.

**Maximize pre-hardware work.** The bulk of the firmware logic is hardware-independent: the audio pipeline structure, the routing graph engine, the inter-module protocols, the web platform. These can be developed and tested in simulation on a regular computer long before any hardware exists. The hardware-specific parts (codec drivers, control reading) are a relatively thin layer at the bottom that can wait. The plan front-loads the hardware-independent work because it is both the largest fraction of the effort and the part that does not depend on uncertain hardware timing.

**Use a development board before custom hardware.** When hardware-dependent work does begin, a commodity STM32H7 development board (Nucleo-H743 or similar, $25-50) plus a codec breakout on a breadboard provides a working audio platform. Firmware developed on the dev board ports to the custom Metapedal PCB with minimal changes because it is the same MCU. The custom PCB is not on the critical path for most firmware development.

**Rust throughout.** The firmware is written in Rust. The static typing catches errors at compile time, the memory safety prevents whole classes of embedded bugs, the embedded Rust ecosystem (Embassy, RTIC, the HAL crates) has matured substantially, and the founder's existing Rust expertise means no learning curve. Rust also helps with fragmented-time work: the type system serves as documentation and guard rail, so coming back to code after a gap is less error-prone because the types restore context and the compiler catches mistakes from forgotten details.

**Test in simulation wherever possible.** A simulation harness that models the audio pipeline, multiple modules, and the inter-module bus lets most of the firmware be developed and validated without hardware. Protocol logic in particular can be fully tested in simulation. The plan invests in simulation infrastructure early because it pays for itself across all subsequent work.

## Phase Structure Overview

The work decomposes into phases with clear dependency relationships. Phases are not strictly sequential; later phases can begin before earlier ones are fully complete, and the fragmented-time reality means work proceeds opportunistically. But the dependency structure determines what can be worked on at any given time.

Phase 0: Foundations and simulation infrastructure.
Phase 1: Single-module audio pipeline (the thin vertical slice).
Phase 2: The inter-module protocol core.
Phase 3: The routing graph engine.
Phase 4: The web platform and control plane.
Phase 5: Application-level features.
Phase 6: Hardware bring-up and porting.

Phases 0 through 5 are largely hardware-independent and can happen in simulation. Phase 6 is where the firmware meets real hardware, and it can begin in parallel with the later simulation phases once a development board is available.

## Phase 0: Foundations and Simulation Infrastructure

The first work establishes the development environment, the project structure, and the simulation harness that subsequent work depends on. This phase produces no user-visible functionality but is the foundation everything else builds on.

The project structure establishes the Rust workspace with separate crates for the hardware-independent core (the audio pipeline logic, the protocol logic, the routing engine) and the hardware-specific layer (the HAL integration, the drivers). The separation is what allows the core to be developed and tested in simulation while the hardware layer is stubbed out. The core crate has no dependencies on hardware-specific functionality; it operates on abstract interfaces that the simulation harness and the real hardware both implement.

The simulation harness models the runtime environment the firmware will eventually run on. It provides a simulated audio source and sink (reading from and writing to audio files, or generating test signals), a simulated set of controls (programmable to produce control events), a simulated inter-module bus (connecting multiple simulated module instances), and a simulated clock (for testing time-dependent behavior deterministically). The harness lets the firmware logic run on a regular computer with full debugging, fast iteration, and deterministic reproducibility that hardware cannot match.

The testing infrastructure establishes the patterns for unit tests, integration tests, and the simulation-based system tests. The investment in good testing infrastructure early pays for itself because the fragmented-time reality means the developer frequently returns to code after gaps; comprehensive tests catch regressions that the developer might otherwise introduce from forgetting context.

This phase fits fragmented time well because it is pure software development on a regular computer with no hardware dependency and no need to hold the whole system in mind. Each session can establish one piece of infrastructure.

## Phase 1: Single-Module Audio Pipeline

This phase produces the thin vertical slice: a simulated module that takes audio in, processes it through a trivial effect, and produces audio out. It is the first demonstrable result and validates the core audio architecture.

The audio pipeline implements the block-based processing model. Audio arrives in blocks (sixty-four samples at a time in the default configuration). The pipeline runs the configured DSP on each block and produces output blocks. In simulation, the blocks come from and go to audio files or test signal generators; on hardware, they come from and go to the codec via DMA. The pipeline logic is identical in both cases because it operates on the abstract block interface.

The Faust integration establishes how Faust-generated DSP code plugs into the pipeline. Faust compiles DSP descriptions to C or to Rust; the pipeline calls the generated code on each block. This phase implements the integration with a few trivial effects (gain, a simple filter, a basic delay) to validate that Faust-generated code runs correctly in the pipeline. The effect library grows later; this phase just establishes the integration pattern.

The control-to-parameter mapping establishes how control events (from controls in simulation, eventually from real controls on hardware) drive DSP parameters. A control event arrives; the mapping translates it to a parameter change; the DSP picks up the new parameter value on the next block. This is the foundation of the platform's decoupled control architecture, implemented first in the simplest single-module case.

By the end of this phase, a simulated module processes audio through configurable effects with control-driven parameters, all running on a regular computer. This is genuinely motivating because it is the platform's core function working end-to-end, even if only in simulation and only for one module.

This phase fits fragmented time reasonably well. The block pipeline is a bounded piece of work. The Faust integration is a bounded piece of work. The control mapping is a bounded piece of work. Each can be a focus of several sessions.

## Phase 2: The Inter-Module Protocol Core

This phase is the heart of the platform's distributed architecture and is well-suited to the fragmented-time, pre-hardware situation because it is largely about protocol logic rather than hardware. The protocols are developed and tested entirely in simulation, with the simulated bus connecting multiple simulated module instances.

The protocol core decomposes into several subsystems with their own internal structure.

**Module discovery.** When modules connect, they need to discover each other: each module learns what other modules are present, what each one is (its variant, its capabilities, its mezzanine configuration), and how they are connected (the topology of the chain or star). The discovery protocol handles modules joining and leaving, produces a consistent view of the connected set, and provides the foundation for everything else the protocol does. Discovery is tested in simulation by instantiating multiple simulated modules on the simulated bus and verifying they correctly discover each other across various topologies and join/leave sequences.

**Leader election.** The platform's architecture designates one module as the leader that coordinates chain-wide services (the routing graph, time synchronization, the configuration state). Leader election is a classic distributed systems problem with well-understood solutions; the implementation uses a deterministic election based on module identity plus capability (a module with more resources or a specific designated role wins). The election handles the leader leaving (re-election), network partitions (each partition elects its own leader, with reconciliation when the partition heals), and the various edge cases that distributed consensus involves. This is the most distributed-systems-flavored part of the project and is likely to be motivating for a founder with distributed systems background. It is fully testable in simulation.

**Time synchronization.** Audio routing across modules requires the modules to share a common time base so that audio samples can be aligned. The time synchronization protocol establishes a shared clock across the connected modules, accounting for the propagation delay across the bus. This is similar to PTP (Precision Time Protocol) used in professional audio networks, adapted to the platform's specific bus. Time sync is correctness-critical for audio alignment and is tested in simulation by verifying that simulated modules converge on a shared time base and maintain alignment under various conditions.

**The routing protocol.** Once modules are discovered, a leader is elected, and time is synchronized, the routing protocol carries audio and control signals between modules according to the routing graph. The protocol handles the data plane (the actual audio and control sample transport) and the control plane (the messages that configure what flows where). The routing protocol is where the platform's value lives; it is what lets a control signal on one module drive a parameter on another, and what lets audio flow through the chain according to the user's configuration. Tested in simulation by configuring routing graphs across simulated modules and verifying that audio and control signals arrive where the graph specifies.

**The lease protocol for mezzanine resources.** The mezzanine bus lease protocol (covered in detail in the mezzanine bus specification) manages the allocation of I2S resources to mezzanines. The protocol logic is implemented and tested in this phase, validating that lease requests, grants, denials, and releases work correctly and that contention is handled gracefully. This is largely module-local logic but interacts with the routing protocol because the routing graph determines what leases are needed.

By the end of this phase, multiple simulated modules discover each other, elect a leader, synchronize time, route audio and control signals according to a configured graph, and manage mezzanine resources through leases. This is a substantial validated subsystem, all running in simulation on a regular computer. It is the part of the project most amenable to fragmented-time work because each protocol subsystem is a bounded, logically self-contained piece that can be developed and tested independently.

## Phase 3: The Routing Graph Engine

This phase implements the engine that takes a declarative routing graph (the user's intent) and compiles it to a physical plan (the realized configuration of modules and inter-module connections). This is the "query planner" concept that appears throughout the platform's design.

The routing graph data model defines how routing graphs are represented: nodes (audio sources, audio sinks, effects, control sources, control sinks, analysis nodes), edges (audio connections and control connections between nodes), and the parameters that configure each node. The data model is the contract between the web platform (which produces routing graphs) and the engine (which realizes them).

The compilation logic transforms a routing graph into a physical plan. It assigns nodes to specific modules (based on capability and load), determines what audio and control flows are needed between modules, computes the lease requests needed for mezzanine resources, validates the plan against hardware constraints (DSP budget, bandwidth, latency, lease availability), and produces either a valid physical plan or a diagnostic explaining why the graph cannot be realized.

The validation and diagnostics are a substantial part of the value. When a user's routing graph cannot be realized (too much DSP load on one module, insufficient bandwidth on a cable, conflicting lease requests, excessive latency), the engine produces a specific diagnostic explaining the problem and suggesting remediation, rather than failing silently or mysteriously. This is the intent-versus-implementation separation that the platform applies throughout.

The engine is developed and tested in simulation, using the simulated modules from Phase 2 as the targets that physical plans configure. Test routing graphs are compiled and the resulting physical plans verified for correctness and for appropriate diagnostics on invalid graphs.

This phase fits fragmented time well because the compilation logic is a bounded algorithm and the validation rules are individually testable. Each validation rule (DSP budget, bandwidth, latency, leases) can be a focus of a session.

## Phase 4: The Web Platform and Control Plane

This phase implements the configuration UI that users interact with and the control plane that connects the UI to the modules.

The control plane on the module side exposes the module's state and capabilities to the web platform: what the module is, what it is currently doing, what parameters it has, what the current routing graph is. The control plane handles configuration changes from the web platform (apply this routing graph, change this parameter, save this preset) and reports status back (current levels, lease state, diagnostic conditions).

The presentation protocol delivers the web UI to the user's browser. As covered in the web platform document, this uses a network coprocessor on the module serving HTTP and WebSocket to the user's device. The protocol is implemented and tested first in simulation (the simulated module serves the web platform to a real browser pointed at the simulation), then ported to hardware later.

The web UI itself is the routing graph editor plus the per-module configuration interface plus the preset management. This is web development (HTML, CSS, JavaScript, or a framework) rather than embedded development, and it is a substantial piece of work in its own right. It can be developed against the simulated module's control plane, with a real browser editing routing graphs that the simulation realizes.

This phase has two fairly independent tracks (the embedded control plane and the web UI) that can proceed somewhat separately. The web UI in particular is a different kind of work (frontend development) that might suit different working sessions than the embedded Rust work, which is useful for a fragmented-time developer who can match the available session to the kind of work that fits their current context.

## Phase 5: Application-Level Features

This phase implements the higher-level capabilities that build on the stable lower layers: presets, audio analysis, the streaming router, and the various application features the platform offers.

Presets and the configuration data model implement saving, loading, and managing configurations. This builds on the routing graph engine (a preset is essentially a saved routing graph plus parameter settings) and the control plane (presets are applied through the control plane).

The audio analysis algorithms (envelope follower, transient detector, beat tracker, pitch detector, and the rest of the catalog from the TODO document) are implemented as analysis nodes in the routing graph. Each algorithm is a bounded piece of DSP work that produces control signals. They build on the audio pipeline (they process audio blocks) and the routing graph (their outputs are control signals that flow to parameters).

The streaming router and the various application features build on everything below them. These are implemented as the platform matures and as specific use cases demand them.

This phase is open-ended; it grows as the platform's capabilities expand. The features are individually bounded and can be implemented opportunistically as interest and time allow.

## Phase 6: Hardware Bring-Up and Porting

This phase is where the firmware meets real hardware. It can begin in parallel with the later simulation phases once a development board is available, and it becomes the focus once custom hardware exists.

The hardware abstraction layer implements the abstract interfaces (audio I/O, control reading, bus communication) against the real hardware. The audio I/O implementation drives the codec through the SAI peripheral and the DMA controller. The control reading implementation reads the actual buttons, encoders, and sensors. The bus communication implementation drives the actual USB or ethernet inter-module connections. Because the core firmware operates on the abstract interfaces, the hardware layer is the only part that changes between simulation and hardware.

The development board bring-up gets the firmware running on a commodity STM32H7 dev board plus a codec breakout. This validates the hardware abstraction layer against real silicon and surfaces the inevitable differences between simulation and reality (timing, peripheral quirks, the actual behavior of the codec). The dev board bring-up can happen relatively early, in parallel with the simulation-based development of the higher layers, because it validates the foundational hardware layer that everything else depends on.

The custom hardware bring-up gets the firmware running on the actual Metapedal PCB once it exists. Because the firmware has already been validated on the dev board with the same MCU, the custom hardware bring-up is mostly about the differences between the dev board and the custom board (the specific codec, the specific control hardware, the specific connectors). This is where the hardware collaborator's involvement is most valuable.

The hardware-specific debugging surfaces the issues that only appear on real hardware: timing margins, signal integrity, thermal behavior, power consumption, the analog audio quality. These cannot be fully anticipated in simulation and require iteration on real hardware. The plan budgets time for this iteration because hardware always has surprises.

## Sequencing Strategy for Fragmented Time

The plan front-loads the hardware-independent work (Phases 0 through 5) because it is the largest fraction of the effort and does not depend on uncertain hardware timing. A time-constrained developer working in fragmented sessions can make substantial progress on the simulation-based work without ever touching hardware. The protocols, the routing engine, the web platform, and the application features are all developed and validated in simulation.

The hardware-dependent work (Phase 6) begins opportunistically when a development board is available and the developer has the time and context for hardware work. The dev board bring-up validates the hardware abstraction layer; the custom hardware bring-up waits for the custom PCB to exist, which depends on the hardware collaborator situation.

The vertical-slice approach means a demonstrable result exists early (the Phase 1 single-module audio pipeline in simulation) and the demonstrable capability grows over time (two modules in Phase 2, configurable routing in Phase 3, web-based configuration in Phase 4). Each phase produces something that can be shown, which sustains motivation and provides validation that the architecture works.

The protocol work in Phase 2 is likely to be the most engaging for a founder with distributed systems background, and it is fully doable in simulation, so it can be a focus of early enthusiasm. The web platform work in Phase 4 is a different kind of work (frontend) that suits different sessions. The application features in Phase 5 are individually bounded and can be picked up opportunistically. The plan allows the developer to match the available session to the kind of work that fits their current context and energy.

## What Could Be Demonstrated and When

A useful way to think about progress is in terms of demonstrable milestones, since visible progress sustains a fragmented-time project.

After Phase 1: a simulated module processes audio through effects with control-driven parameters. Demonstrable as a command-line tool that takes an audio file, processes it, and produces an output file, with control events scripted to show parameter changes.

After Phase 2: multiple simulated modules discover each other, elect a leader, synchronize time, and route audio and control signals between them. Demonstrable as a simulation that shows audio flowing through a multi-module chain according to a configured routing.

After Phase 3: routing graphs compile to physical plans with validation and diagnostics. Demonstrable as a tool that takes a routing graph description and shows the resulting physical plan, or the diagnostic explaining why the graph cannot be realized.

After Phase 4: a real browser edits routing graphs and configures modules, with the simulation realizing the configurations. Demonstrable as a working web UI driving the simulated platform, which is close to the actual user experience minus the hardware.

After the dev board bring-up in Phase 6: the firmware runs on real silicon, processing real audio through a real codec. Demonstrable as an actual sound coming out of an actual development board, which is the moment the project becomes tangibly real rather than purely simulated.

After the custom hardware bring-up: the firmware runs on an actual Metapedal module. Demonstrable as a working prototype module, which is the milestone that turns the project from architecture-plus-simulation into a real thing.

The simulation-based milestones (Phases 1 through 4) can all be reached without any hardware, which means substantial demonstrable progress is possible entirely on a regular computer in fragmented sessions. This is the key insight for the time-and-money-constrained situation: most of the firmware can be built and shown to work before any hardware investment, which de-risks the project and provides validation that could attract a hardware collaborator or justify the eventual hardware investment.

## Estimated Effort Shape

Without committing to a schedule (which would be dishonest given the fragmented-time reality), the rough effort shape is worth noting.

Phase 0 is modest: setting up the project structure and simulation harness is a few weeks of part-time work, front-loaded because everything depends on it.

Phase 1 is moderate: the audio pipeline, Faust integration, and control mapping are a few months of part-time work, producing the first demonstrable result.

Phase 2 is substantial: the protocol core (discovery, leader election, time sync, routing, leases) is the largest single piece of the hardware-independent work, several months of part-time effort, but also the most architecturally interesting and the most amenable to fragmented sessions.

Phase 3 is moderate: the routing graph engine is a few months of part-time work, building on the protocol core.

Phase 4 is substantial but parallelizable: the control plane and the web UI together are several months, but the two tracks can proceed somewhat independently and the web UI is a different kind of work.

Phase 5 is open-ended: the application features grow over time, with each feature individually bounded.

Phase 6 is gated by hardware availability: the dev board work can happen relatively early and is moderate; the custom hardware work waits for the PCB and is unpredictable because hardware always surprises.

The total is a multi-year part-time effort to reach a working custom-hardware prototype, with substantial demonstrable progress (the full simulation-based platform) reachable in the first year or so of consistent part-time work. The simulation-based milestones provide the validation and motivation that sustain the longer effort toward hardware.

## The Critical De-Risking Insight

The single most important strategic point in this plan: the bulk of the firmware can be developed and validated in simulation before any hardware exists. This means the project's largest technical risk (does the architecture actually work?) can be retired through simulation-based development that costs only the developer's time, with no hardware investment, no cofounder commitment, and no capital.

A developer working part-time can build the simulated platform (audio pipeline, protocols, routing engine, web UI) and demonstrate that the architecture works end-to-end. This demonstrable simulated platform is then the foundation for everything else: it validates the design, it provides a portfolio piece that demonstrates the work is real, it could attract a hardware collaborator who sees a working simulation and wants to help make it physical, and it justifies the eventual hardware investment because the architecture has been proven.

This sequencing turns the daunting "build a hardware platform" problem into the more tractable "build a simulated platform first, then make it physical" problem. The simulated platform is achievable within the founder's actual constraints (time-limited, money-limited, working alone). The physical platform is the subsequent step that the validated simulation justifies and enables.
