# Metapedal Web Platform and Configuration

This document covers the firmware control plane and web-based configuration: the presentation protocol that delivers a configuration UI to any browser, the presets and configuration data model that persists user setups, and the streaming router that connects DSP blocks within a module.

The audience is engineers building the firmware control plane and the web UI.

For the DSP blocks that the streaming router connects, see `tech-03-dsp.md`.

## The Presentation Protocol: Web Technology Over a Network Coprocessor

The data channel between the rig and external presentation consumers (touchscreen mixers, graphics-accelerator breakouts, future display-capable modules, audience visualizations) needs a well-defined protocol. The architecturally cleanest answer is to use the web platform: HTTP and WebSocket for the protocol, HTML/CSS/JavaScript for the UI, and a browser on the consumer device for rendering. The rig hosts a small web server, the web server serves a Metapedal UI as a web page, and any device with a browser can load that page and get a fully functional interface.

This is the architectural pattern that Palm's webOS demonstrated at scale on consumer devices in the late 2000s, and that countless commercial IoT devices, network routers, smart appliances, and some pro audio equipment use today. The phone's UI is a web page rendered by a local WebKit instance; system services expose APIs over local sockets; apps are essentially web apps with extended privileges. The architecture is clean, the development workflow is good, and the user experience is responsive enough for serious interactive use.

For Metapedal, this approach replaces designing a custom protocol with adopting the web's well-established protocols. The rig provides services through HTTP REST endpoints for parameter changes and scene management, and WebSocket streams for real-time data like level meters and spectrum information. The UI is a web app that consumes these services and renders accordingly using HTML/CSS/JavaScript through the consumer's browser rendering engine. The browser handles all the graphics work, including the imperative drawing that earlier protocol sketches were trying to specify from scratch.

### Why Web Technology Over a Custom Protocol

Several properties of the web platform make it genuinely better for this use case than a custom protocol would be.

The mature ecosystem advantage is huge. Web development has decades of accumulated tools, libraries, frameworks, patterns, and community knowledge. Building the Metapedal UI as a web page lets contributors use whatever they are comfortable with: React, Vue, Svelte, vanilla JavaScript, Web Components, Canvas-based visualization libraries, WebGL for fancier graphics. The platform does not need to ship a custom rendering engine, a custom protocol library, or platform-specific client apps for every consumer device. Browsers exist on every device the platform might want to support (laptops, tablets, phones, embedded touchscreens, USB-C touchscreen monitors with built-in computers), and they render web content consistently enough to be useful.

The remote-versus-local symmetry is clean. The same web page works whether the user is sitting at the rig with a local touchscreen or sitting at front-of-house with a tablet on the local WiFi network or sitting in their bedroom configuring the rig over a home network. The same code path serves all three. "Where is the display" becomes "what URL is the browser pointing at," which is a non-technical question with a simple answer.

The development workflow is excellent. UI changes get tested by reloading a browser page, with no firmware flashing required. The UI can be developed entirely in a browser with mocked services standing in for the rig's APIs. Debugging uses standard browser dev tools (DOM inspector, network inspector, JavaScript debugger, performance profiler). The whole make-a-change-see-if-it-works cycle is the web cycle, which is fast and pleasant compared to embedded firmware development.

The extensibility story is strong. Third-party developers can build their own Metapedal client apps just by making HTTP and WebSocket calls to the rig. A custom recording tool, a custom visualization, a custom MIDI controller integration, a custom analysis tool, all become straightforward web apps. The rig is essentially a server with a documented API; clients can be anything that speaks HTTP and WebSocket.

The text rendering, font handling, layout, and graphics rendering questions that a custom protocol would have to solve are all answered by the browser. Fonts come from the consumer device's system fonts or from web fonts the page loads. Layout is CSS flexbox or grid. Graphics rendering uses Canvas or SVG or WebGL depending on the visualization's needs. The platform inherits decades of work on these problems for free.

### The Hardware Architecture: ESP32 as Network Coprocessor

The H7 is not the right chip to run a web server. The H7's strengths are deterministic real-time DSP, hardware peripheral control, and low-latency audio processing. Asking it to also serve HTTP responses, maintain a TCP/IP stack, handle WebSocket connections, and host a web page adds complexity that competes for resources with the audio work that is the H7's actual job. The H7 has the capability to do this (lwIP for the TCP/IP stack, plus a small HTTP server, plus a WebSocket implementation), but it is not its native habitat.

The right tool for the web-serving job is an ESP32 or ESP32-S3 as a dedicated network coprocessor. The ESP32 is a dual-core Xtensa processor with WiFi and Bluetooth built in, mature TCP/IP and HTTP libraries (the ESP-IDF includes everything needed), $3-5 in production quantities, and explicitly designed for this kind of small-networked-device role. The ESP32 bolts onto the H7 through a simple interface (UART, SPI, or both, with the SPI variant supporting higher throughput for the streaming data) and handles all the network and UI work. The H7 produces state and event streams over the interface to the ESP32; the ESP32 wraps that data in HTTP responses and WebSocket frames for browser clients to consume.

This is also how it works in practice for serious embedded products that need network capabilities alongside real-time work. Many commercial audio devices use a dedicated network coprocessor exactly this way: the main DSP runs on a Cortex-M, the network and management interface runs on an ESP32 or similar, and the two communicate through a simple protocol over SPI or UART. The architecture is proven and the parts are commodity.

The ESP32 also brings WiFi for free, which solves a real problem. The wireless touchscreen scenario (the sound person walking around the venue with a tablet) becomes natural: the ESP32 hosts a WiFi access point or joins an existing network, and tablets connect over WiFi to the rig's web server. No proprietary wireless protocol needed; just standard WiFi that any device already knows how to use. The Bluetooth peripheral that earlier sketches assumed for wireless touchscreen support gets subsumed into the ESP32's built-in capabilities, simplifying the architecture.

### Physical Placement and Variant Considerations

The ESP32 with WiFi capability needs to be outside the metal enclosure to avoid the Faraday cage problem. The standard module's metal box blocks 2.4 GHz signals, so the ESP32 lives on a small daughterboard either integrated with the back-panel mezzanine or as a separate small board attached through a connector on the rear panel. The daughterboard has the ESP32, a small chip antenna, and the few supporting passives the ESP32 needs.

The cost picture is reasonable. The ESP32 is $3-5, the chip antenna is around $0.50, the supporting passives are under $1, and the daughterboard PCB plus its connector to the main motherboard is maybe $2-3. Total BOM addition is around $6-10 for the complete WiFi/web-server capability.

The question of which modules include the ESP32 network coprocessor is a variant-level design decision. Only one module in a chain needs to host the web server, since clients access the chain through the chain leader's network interface. Putting WiFi on every module is wasteful since most modules in a chain do not need it. Following the variant-based product differentiation pattern, the natural placements are:

The A variant (the natural chain leader and configuration host) gains the ESP32 and WiFi as a standard feature, making every chain that includes an A variant network-capable. This is clean because the A variant is already positioned as the chain coordination role, and adding network capability fits that role naturally.

Alternatively, a network-capable peripheral plugs into a module's M8 Smart Digital port and provides the web server externally. Any chain that wants browser-based UI access adds the peripheral; chains that do not, do not pay for it. The web server runs on the peripheral's ESP32, with the peripheral fetching state from the chain over the inter-module bus through its M8 connection. This makes network capability an opt-in add-on rather than a built-in feature.

Either approach works. The A-variant-includes-it approach is simpler for users (always available, no peripheral to remember) but adds cost to every A variant. The peripheral approach is more flexible (users choose) but adds setup steps. For v1, the A-variant-includes-it approach is probably right because it makes the network capability a baseline feature that every chain has access to. The peripheral approach can come as an alternative for users who prefer modularity.

### The Service API

The HTTP and WebSocket API exposed by the ESP32's web server is the platform's presentation-layer API. The specifics need to be designed carefully because the API becomes a long-term commitment that third-party developers will build against. A few principles for the API design:

REST conventions for resource-oriented operations (channels, scenes, presets, configuration) where the verbs map cleanly to HTTP methods. GET to retrieve current state, POST to create new resources, PUT or PATCH to update existing ones, DELETE to remove. The resources are mostly things users think about (channel 3, scene "verse-1", preset "clean tone"), making the API feel natural to web developers.

WebSocket streams for real-time data that needs continuous updates: per-channel level meters, spectrum data, parameter values that change frequently, IMU motion data. The WebSocket connection stays open for the session and pushes updates as they happen. Clients subscribe to whatever streams they need; the server pushes only the subscribed data.

JSON for message formatting. The slight overhead compared to binary formats is worth it for the developer experience: JSON is human-readable, debuggable with standard tools, parseable in every language. Real-time streams might use a more compact format (binary WebSocket frames with CBOR or MessagePack payloads) if profiling shows JSON overhead matters, but the default should be JSON.

CORS configuration that allows browsers to load the Metapedal web app from the rig's IP address, with appropriate security headers. The web app served from the rig itself does not need CORS, but third-party web apps that talk to the rig do, so the API needs to be CORS-friendly.

Authentication and authorization that fit the use case. For most home and small-venue use, no authentication is needed (the rig is on a trusted local network). For larger installations or environments where access control matters, basic authentication or token-based auth can be added. The default should be no-auth-required to keep setup simple, with auth available as a configuration option for users who need it.

WebSocket reconnection handling because real-world WiFi connections drop. The client and server need to handle disconnect-reconnect cycles gracefully, with the client resyncing state on reconnect rather than starting from scratch.

### The Web App Itself

The Metapedal UI as a web app is meaningful software development effort but uses standard tools. A modern web framework (React, Vue, Svelte) plus a charting/visualization library (D3, Chart.js, or Canvas-based custom rendering for performance-critical visualizations) plus standard CSS for layout. The app is served as static files from the ESP32's web server, with the dynamic data flowing through the API endpoints.

The app's architecture mirrors the chain's architecture: the home view shows the chain's overall state with each module represented, navigating into a module shows that module's detail view with its specific parameters, navigating into a channel shows that channel's processing chain with effects parameters and routing. The user navigates through the same conceptual hierarchy whether they are using the web UI or interacting with the modules directly through their footswitches and OLEDs.

The development effort is real but bounded. A v1 web app that covers the mixer use case and the basic configuration use case is maybe a few person-months of web development. Subsequent capabilities (custom visualizations, advanced scene management, recording controls, MIDI mapping configuration) build on the established foundation. The web app is the kind of thing that benefits from community contribution: anyone who can build a web app can build features for it, and the open-source release of the app lets the community pile in.

### Comparison to the Earlier Custom Protocol Sketch

Earlier drafts of this document sketched a custom binary protocol with declarative widget commands, imperative drawing commands, and a custom client app for each consumer type. That approach has been retired in favor of the web platform approach for several reasons: it required designing protocols from scratch (text rendering, font handling, color models, transformation matrices, image handling, state management), it required building custom client apps for each device type, it could not leverage the web ecosystem's tools and talent, and it would have been significantly more work to deliver less capability than the web platform provides for free.

The web platform approach is the right answer because the problems it solves (rendering UIs across devices with different capabilities, transmitting data between services and clients, handling real-time updates) are exactly the problems the web platform was designed to solve, and the platform has decades of accumulated solutions for them. Adopting the web platform inherits all of those solutions; designing a custom protocol re-creates them poorly.

## Presets and the Configuration Data Model

A preset is a complete snapshot of the routing graph and all parameter values in the chain at a moment in time. Activating a preset replaces the current routing-plus-parameters with the saved one, atomically across all modules. The audio processing functions themselves do not change, because the modules are still the same modules. What changes is which signals flow to which parameters and what values those parameters hold.

This data model is small and clean. A preset is essentially a serialization of the routing graph plus parameter snapshots. The preset can be stored locally on a module, transmitted between modules over the inter-module bus, exported through the phone app to cloud storage, shared between users, and so on. The format should be designed for stability across firmware versions, with explicit versioning and forward compatibility considerations, because users will accumulate preset libraries over time and those libraries need to survive firmware updates.

Presets are addressed by module role and position in the chain, not by physical identity. A preset says "the first Processor module in the chain should have its drive parameter at this value," and the chain matches that to whatever physical module fills the role. This makes presets portable between similar-but-not-identical chains. The enumeration protocol that runs when modules are connected assigns roles and positions in a deterministic way, so presets can refer to them reliably.

Switching between presets at performance speed introduces timing concerns. When a preset switch happens, the entire chain needs to switch atomically, or at least appear atomic to the listener. If module A switches twenty milliseconds before module B, the audio has a brief glitch where the signal is being processed by a half-old, half-new configuration. The inter-module bus needs to support synchronized switching, where a preset change is announced ahead of time, all modules acknowledge readiness, and a synchronized commit message tells them all to switch at exactly the same audio sample boundary. The latency budget on the bus needs to be tight enough that this round-trip negotiation completes within a few milliseconds.

A simpler approach that works for most cases is parameter ramping rather than instantaneous switching. Instead of hard discontinuities, parameters slew from their old values to their new values over fifty milliseconds or so, with the audio glitches smoothed out by the crossfade. For most musical contexts this is fine because the listener cannot tell the difference between an instantaneous switch and a fifty-millisecond crossfade, especially in a live mix. Hard synchronization is needed only for edge cases like switching on a precise beat at fast tempo. The specification supports both modes, with crossfaded switching as the default and synchronized switching available for users who need it.

Partial presets are a useful extension. The user might want a preset that affects only some modules, leaving others untouched. A preset that switches the distortion's settings but leaves the delay and reverb alone is a clear use case. The data model supports this through presets that are diffs against the current state rather than complete replacements. This is a small extension but it matters for the user experience because the alternative is forcing the user to specify every parameter on every module every time they create a preset.

Preset stacks or layers, where multiple partial presets are active simultaneously on different parts of the chain, are a further extension that adds power at the cost of user interface complexity. For version one this is probably too much; flat replacement-style presets are simpler to understand and implement. The architecture should not preclude layering but should not implement it initially.

## The Streaming Router

Not every module in a chain actively processes every stream that flows through it. A control-only module that extracts triggers from a drum mic has no business touching a guitarist's audio stream that happens to be passing through the same inter-module bus. Asking the CPU to receive every byte of audio just to forward most of them unchanged is wasteful. The architectural answer is a streaming router that handles passthrough traffic in hardware, without CPU involvement.

The router is dedicated logic that moves data between input and output paths based on a routing table maintained by the CPU but executed in hardware. When a module is configured, the CPU computes which streams the module needs to process locally and which streams just need to be forwarded through to the next module unchanged. The CPU writes this routing information into the router hardware. From that point on, incoming streams marked for local processing get routed to the codec or DMA buffers for DSP. Streams marked for passthrough get routed directly to the appropriate output port. The CPU is free to do its actual DSP work without being interrupted by forwarded traffic.

The latency benefit is significant. Without hardware passthrough, a stream traveling through four modules accumulates buffer latency at each module even when only one of them actually processes it. With hardware passthrough, only modules that actually process a stream contribute to its latency. The drummer's audio can pass through the guitarist's pedalboard on its way to the mixing console with essentially zero added latency, while still being available for any module in the chain that wants to receive it.

The pattern generalizes to control signals as well. Control events flowing through the inter-module bus can be routed by the same hardware, with the router examining destination addresses and forwarding accordingly. The inter-module bus becomes something more like a true network than a shared medium, with each module acting as a small routing node rather than as a bottleneck through which all traffic must flow serially.

The hardware that implements the router can be modest. The microcontroller's DMA controllers, particularly the master DMA on the H7 family, can handle multi-channel transfers between peripherals without CPU involvement once configured. A streaming router implemented in firmware using DMA channels as its execution engine is probably sufficient for the bandwidth and channel counts that version one needs. A dedicated routing ASIC or FPGA becomes a meaningful product enhancement if later scaling demands it, but is not needed initially.

The streaming router pattern implies that the inter-module bus is fundamentally a packet-switched network, not a circuit-switched one. Each stream is identified by a stream ID, and the router examines the ID to decide where each packet goes. The bus protocol carries stream IDs in every packet, supports variable-length packets, and synchronizes between modules at the packet level rather than the sample level. This is more complex than a simple time-division-multiplexed bus but is the right architecture for the routing flexibility the platform needs.
