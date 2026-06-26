# Metapedal: Technical Specification

This is the top-level index for the Metapedal platform's implementation-level technical documentation. The platform is built from standard modules: a shared motherboard provides core compute and inter-module communication, an optional mezzanine adds variant-specific audio I/O, and connector daughter boards carry the external ports. Each subsystem has its own dedicated document; this page is the map.

## Core hardware

- Motherboard Specification (`tech-08-motherboard.md`): the standard motherboard — the microcontroller choice, the power architecture, the functional blocks and their component selections, the ESP32 wireless coprocessor, the connector daughter boards and mezzanine interface, and the storage subsystem. This is the core hardware document; the motherboard is shared across all variants.

## Companion technical documents

- Mezzanine Bus Specification (`tech-04-mezzanine-bus.md`): the contract for third-party mezzanine designers, including the 80-pin connector pin allocation, the lease-based I2S protocol, and the identification protocol.
- Daughter Card Specification (`tech-07-daughter-card-spec.md`): the connector daughter cards (combo jack, USB-C inter-module, M8 peripheral, and the motherboard-direct TRRS path), including their mechanical, electrical, and field-repairability design.
- Audio Analog Design (`tech-02-audio-analog.md`): the B variant analog circuitry, including quarter-inch path, XLR microphone support, phantom power, and galvanic isolation.
- DSP and Audio Processing (`tech-03-dsp.md`): the DSP architecture, Faust language choice, format and latency configuration, and DSP budget analysis with worked examples.
- Inter-Module Transport (`tech-01-transport.md`): connectors, cable quality, ethernet variant, latency analysis, topology choices, bus protocol, and bandwidth.
- Peripherals (`product-01-peripherals.md`): the peripheral ecosystem including contact mic breakout, footswitch breakout, tile system, rotational foot control, pin header, external displays, USB MIDI controllers, and the M8 peripheral identification protocol.
- Web Platform and Configuration (`tech-05-web-platform.md`): the presentation protocol, presets and configuration data model, and streaming router.
- Battery Accessory (`product-02-battery-accessory.md`): the 18650 battery backpack specification.

For the user-facing vision, the core architectural principles, and the use cases the platform serves, see the companion vision document (`product-00.md`).
