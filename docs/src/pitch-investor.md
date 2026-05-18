# Metapedal: A Five-Page Pitch for Investors

## What Metapedal Is

A pedalboard is the rack of effects pedals that guitarists and other musicians use to shape their sound. The standard model has been the same since the 1960s: each pedal is a metal box on the floor with both the audio processing and the user controls in the same enclosure. To use the gear, the musician steps on switches and turns knobs that are physically attached to the audio circuits.

This design assumption is wrong. The controls and the processing should not have to live in the same box. Once you separate them, two things become possible that the traditional model cannot support: any control can drive any parameter (a sensor on your guitar's headstock can modulate the delay's feedback, the foot pedal can adjust the reverb's room size, a contact mic on the drum kit can trigger the synth voice), and the routing between sources and destinations becomes a software-defined graph rather than a fixed wiring of cables.

Metapedal is a modular hardware platform that delivers this decoupling. Each Metapedal module is a small pedal-format aluminum enclosure containing a microcontroller, an audio interface, and high-quality DSP. Modules connect to each other through USB cables; the connections carry both audio and control information. The whole system is configured through a web interface that runs on any phone, tablet, or laptop. Software-defined routing means the same hardware can be a guitar pedalboard, an electronic music performance rig, a multi-channel recording setup, a band's monitor mixer, or a synth-in-a-can.

The platform's market positioning is open hardware plus open protocol. The mezzanine bus that connects the motherboard to the audio interface boards is a published specification that third parties build conforming hardware against. Modules are USB-class compliant so any computer recognizes them as audio interfaces. The whole stack is documented openly so that the community can build on it.

The result is gear that costs $50-95 per module retail (competitive with traditional pedals) while delivering capability that traditional gear cannot match at any price.

## The Problem with Traditional Pedalboards

To see why this matters, consider what guitarists actually experience. A pedalboard has 5-10 pedals chained together; each pedal has 2-4 knobs plus a footswitch; the musician's two hands and two feet are the only ways to interact with the gear during a performance. A song that needs three pedals to change state simultaneously (kicking on distortion plus delay plus reverb for a chorus, for example) requires the musician to tap-dance across three footswitches in fractions of a second. Most musicians miss most of these transitions.

The fundamental problem is that the controls and the processing are bolted to the wrong places. The musician has plenty of expressive bandwidth available through their other movements (tilting the guitar neck, leaning into the amp, stomping their foot, breathing harder when they scream), but none of these can drive audio parameters in the current architecture. The controls are stuck on the pedals; the pedals are stuck on the floor; the musician is stuck working around the limitations.

Multiply this problem across use cases. The electronic musician who wants their drum pattern to change as they jump on stage cannot make that connection without complex MIDI rigging. The vocalist who wants their reverb to respond to their breathing cannot, period. The band recording demos in a basement cannot get per-performer monitor mixes without expensive specialty gear. The installation designer who wants different rooms in a theater to receive different audio cannot do it with standard pedals at all.

Metapedal solves these problems through a single architectural change: the controls and the processing are different products that connect through a documented protocol. The musician's gestures can drive any parameter. The routing between sources and destinations is software, not wires. The platform's value comes from being the right architecture, not from being more expensive than competitors.

## How the Platform Works

Each Metapedal module is built on three layers: a motherboard with the microcontroller and inter-module communication, an optional mezzanine board with the audio interface (codec plus high-quality analog circuitry), and the enclosure that holds them together. The motherboard is the same across all variants; the mezzanine determines what the module can do in addition to the basic functionality provided by the motherboard.

Standard mezzanines in the initial product line cover the main use cases: an A variant with no mezzanine for use as a controller-only module (footswitches, knobs, sensors), B1 and B2 variants with one or two combo jacks for instrument and microphone audio I/O, BX variant that adds a dedicated audio DSP chip for the most demanding algorithms, B16 and B32 variants for multi-channel recording and installation use. Each variant has the same motherboard underneath; the mezzanine is the locus of product differentiation.

Modules connect to each other through USB-C cables for short runs (typical pedalboard configuration) or through ethernet cables for longer runs and multi-performer setups (band on a stage, installation across a venue). The connections carry both audio (multiple channels at professional quality) and the control information that lets the platform know which module is which and what audio should flow where.

Configuration happens through a web interface that runs on any device with a browser. The musician's phone becomes a routing-graph editor where they can drag connections between sources and destinations, configure each module's specific behavior, save presets, and share configurations with other musicians. The web platform is open and runs on every modern device; the platform does not require dedicated displays or custom mobile apps.

The whole architecture is open hardware with an open mezzanine bus specification. Third parties can build conforming mezzanines (specialty I/O, exotic DSP, MIDI integration, wireless capability, whatever) and they work with any Metapedal motherboard. This ecosystem play is meaningful for the platform's long-term value because the community fills in use cases the core team would never have prioritized.

## What This Looks Like for Real Users

Concrete scenarios illustrate the platform's value better than abstract description.

**The hardcore punk kid** plays basements and VFW halls with a small pedalboard. Songs have huge dynamic shifts that need three or four pedals to change state simultaneously. Traditional pedalboards make him miss these transitions in the chaos of performance. With Metapedal, one footswitch becomes the "chorus transition" trigger; all the simultaneous changes happen at once on the beat. The accelerometer on his guitar's headstock makes the feedback respond to how he tilts his neck. The physical gestures he was already making become tonal controls automatically.

**The electronic music busker** runs a complete one-person rig on a single $80-95 Metapedal module: 808 drum machine, 303 bass synth, melodic synth voice for leads, vocal processing with effects. The MIDI backing track plays from microSD inside the module. The complete electronic music performance setup that costs $1,500-3,000 in commercial gear runs on hardware that costs less than $100.

**The basement band recording demos** uses three or four Metapedal modules instead of a $500 audio interface plus $1,000 of effects gear plus separate monitor mixing. The modules function as USB audio interfaces for recording while simultaneously providing per-performer monitor mixes through their TRRS headphone jacks. The drummer gets a click track; the singer gets reverb on their monitor but not on the recording; the producer gets dry stems plus processed signals for post-production flexibility. Total gear cost: comparable to the audio interface alone in the traditional setup.

**The installation customer** in a small theater uses Metapedal modules with ethernet connections to put audio processing in different rooms, with the gigabit switch at the venue connecting everything. The 100 Mbps ethernet capacity is comfortable for the realistic channel counts; the long cable runs (up to 100 meters) handle the geometry that USB cannot. Per-performer monitor mixes, click track distribution, and talkback all work in the same infrastructure.

**The keyboardist** uses a Metapedal module as a polyphonic synth-in-a-can: plug in a MIDI keyboard, plug in headphones, and have a complete portable instrument that fits in a backpack. The same module can run a vintage TR-808 emulation on Monday for an electronic music gig, a guitar amp simulation on Tuesday for a session, and a Hammond organ emulation on Wednesday for a church service. The hardware is general-purpose; the function is software-defined.

Each of these is a real use case that traditional gear either cannot serve at all or serves with substantially more cost and complexity. Metapedal's value is that the same platform serves all of them.

## The Market Opportunity

The traditional guitar pedal market is mature: roughly $1.5-2 billion globally, growing 5-7% annually, dominated by established brands (Boss, MXR, Electro-Harmonix, Strymon, Eventide, Line 6, Neural DSP, Kemper). Metapedal does not aim to displace these brands; it aims to create a category adjacent to them with substantially different positioning.

The realistic addressable markets are larger than the pedal market alone:

The professional and prosumer audio interface market is roughly $800 million annually. Every Metapedal B variant is also a USB-class compliant audio interface comparable to a Focusrite Scarlett 2i2 or PreSonus AudioBox. Customers buying audio interfaces who discover that the same module also does standalone effects and synthesis are getting substantially more value than the interface alone.

The hardware synthesizer market is roughly $400 million annually, with notable growth in boutique and modular categories. Metapedal modules running DX7 emulation, 808 drum machine emulation, 303 bass synth emulation, and other classics serve this market at price points well below dedicated synth hardware.

The small-format recording gear market (interfaces, mixers, multi-channel preamps) is roughly $600 million annually. Metapedal multi-module configurations serve the basement band and home studio segments of this market.

The installation and pro AV audio market is roughly $5 billion annually, with niche segments where Metapedal-class capability fits cleanly (small theaters, multi-room venues, broadcast applications with modest channel counts, distributed audio for retail and hospitality).

The hobbyist and maker market for audio gear is roughly $200 million annually with strong growth. The open hardware platform plus published mezzanine bus specification serves this market directly; community-built mezzanines extend the platform into use cases the core team would never have prioritized.

Conservatively addressable across these markets in years 3-5 of operation: $50-100 million in annual revenue potential at the product's positioning. This assumes 1-2% share of the relevant subsegments, which is realistic for a platform with the architectural advantages Metapedal has.

## Competitive Differentiation

The competitive landscape has several players that should be acknowledged honestly. Neural DSP, Kemper, and Line 6 dominate the high-end neural amp modeling market at $1,500-3,000 retail. Boss, Strymon, and similar brands dominate the high-end traditional pedal market at $200-500 per pedal. Eventide and Lexicon dominate the multi-effect processor market at $500-1,500. Focusrite and PreSonus dominate the entry-level audio interface market at $100-300.

Metapedal differentiates from each of these on dimensions that matter:

Against neural amp modelers: Metapedal does not compete on neural amp model quality (the BX variant can run neural models but at SHARC-class quality rather than commercial-product quality). Instead, Metapedal offers the modular architecture, multi-channel recording integration, per-performer monitor mixes, and ecosystem extensibility that neural amp products cannot match. The customer choosing between Quad Cortex and Metapedal is making different tradeoffs, not direct comparison.

Against traditional pedals: Metapedal offers substantially more capability per dollar (software-defined routing, multi-channel recording, USB audio interface integration, per-pedal preset memory beyond what traditional pedals offer). The Boss customer who wants more flexibility than their Boss pedalboard provides finds Metapedal directly addresses their pain points.

Against multi-effect processors: Metapedal offers modularity (modules can be configured into different layouts for different gigs) and recording integration that multi-effect processors do not provide. The Eventide customer who wants their reverb at a specific point in the signal chain finds Metapedal's routing-graph approach more flexible.

Against audio interfaces: Metapedal modules also function as audio interfaces, providing equivalent quality at competitive price points plus the standalone processing capability that interfaces alone cannot provide. The Focusrite customer who wants effects processing in their interface finds Metapedal offers substantially more value.

The fundamental competitive moat is the architecture itself plus the ecosystem that grows around it. Anyone trying to copy Metapedal has to either copy the published architecture (which is open and documented) or build something different that misses Metapedal's specific value proposition.

## Revenue Path and Business Model

Metapedal generates revenue through hardware sales: the standard variants (A, B1, B2, BX, B16, B32) at $50-170 retail each, plus accessories (peripherals, tile-based controllers, cables, enclosures). The platform itself is open hardware, which means the platform's value to customers does not depend on proprietary lock-in.

Revenue grows through several mechanisms over time:

Initial customer acquisition through enthusiast channels (Reverb, Sweetwater, direct sales, music industry trade shows). The first 1,000-5,000 customers are platform-curious musicians who want the modular flexibility that traditional gear cannot provide.

Ecosystem development through community mezzanine designers, who extend the platform into use cases the core team does not serve directly. Third-party mezzanines drive additional motherboard sales as users buy more modules to host more capabilities.

Word-of-mouth growth as customers demonstrate Metapedal-based rigs at gigs and online. The platform's value is visible in performance, which drives organic awareness in musician communities.

Market expansion into adjacent segments (installation, hobbyist, electronic music, recording) as the platform demonstrates value in each segment.

Year 1 revenue target: $100-500K from initial small-batch product launch and direct community sales. The first year focuses on validation rather than scale; the revenue is modest because the team is small and customer acquisition relies on direct community engagement rather than paid marketing.
Year 2 revenue target: $1-3M as the ecosystem matures, the Series A funds commercial scaling, and the platform reaches enthusiast awareness in target segments.
Year 3 revenue target: $5-10M as the platform reaches mainstream awareness through ecosystem growth and word-of-mouth in musician communities.
Year 5 revenue target: $20-40M as the platform establishes presence across multiple adjacent markets and the third-party mezzanine ecosystem drives motherboard sales velocity.

These are conservative estimates appropriate for a first-time-founder team building an architecturally-novel product. Aggressive growth scenarios (platform becomes the de facto standard for modular audio gear, ecosystem development accelerates substantially, a major brand picks up the platform for OEM use) could substantially exceed these targets, but the pitch is calibrated to what the team can responsibly commit to rather than to what is theoretically achievable.

The unit economics are favorable. BOM costs of $33-48 per module at moderate volumes give gross margins of 35-50% at retail price points of $80-95. Manufacturing scale would improve margins further. Software development costs amortize across the platform; the open hardware and protocol mean R&D investment is concentrated on the platform itself rather than on each variant.

## The Ask

Metapedal seeks $1.5-2.5 million in seed funding to fund 18 months of operations through initial demo product, validation with early customers, and the team-building work that prepares for a Series A focused on commercial scaling.

This is a deliberately modest seed round calibrated to a first-time-founder profile and an architecturally-novel product that benefits from validation before scaling. The seed proves the platform works and finds early customers; the Series A funds the commercial scaling work that turns early traction into a real business.

Use of funds:
- Engineering team (2-3 firmware and hardware engineers including hardware cofounder if not already in place, audio DSP and web platform work shared with founder): roughly 55% of budget
- Manufacturing prototypes and small-batch initial inventory: roughly 20% of budget
- Operating expenses (rent, equipment, legal, accounting): roughly 15% of budget
- Marketing and early customer outreach (modest at this stage; relies on direct community engagement rather than paid acquisition): roughly 10% of budget

Milestones the funding would deliver:
- Month 6: Engineering prototypes of A and B1 variants complete and tested with internal users. Mezzanine bus specification drafted and circulated to hardware advisor community for review.
- Month 12: A, B1, B2 variants in small-batch manufacturing run. Mezzanine bus specification published publicly. First 50-100 customers acquired through direct community engagement (hackspaces, music technology forums, music industry early adopters).
- Month 18: Initial product launch through enthusiast channels (Reverb, direct sales). 500-1,000 customers in the first cohort. BX variant in late-stage development. First third-party mezzanine designs in active development by community members.

Series A would target $5-10M raised in months 15-21 to fund commercial scaling: expanded engineering team for BX and beyond, marketing investment for mainstream awareness, sales operations, manufacturing scale-up, and the operational infrastructure needed for a hardware platform reaching $5M+ annual revenue.

This staged approach is honest about the first-time-founder execution risk. The seed funds validation and team-building; the Series A funds commercial scaling once the platform has demonstrated it works and the team has demonstrated it can execute. Investors at each stage are matched to the appropriate stage's risk profile.

## Risks and How the Founder Plans to Address Them

**Risk: First-time-founder execution gap on commercial product operations.** This is the most important risk to name explicitly. The founder is a software engineer with substantial architectural experience but no prior commercial hardware product shipped. The mitigation has several components: an experienced cofounder or senior advisor with prior commercial product experience in the founding team (currently in active discussions with [hardware EE cofounder candidate from local hackspace community]); a hiring plan that brings commercial operations experience into the team in months 6-12; advisors with deep experience in audio industry commercialization who provide ongoing input; and a conservative scope and timeline that does not require commercial-execution sophistication beyond what the team can deliver. The seed round's modest size and 18-month timeline are themselves part of this mitigation; the founder is not asking for resources beyond what a first-time team can responsibly execute.

**Risk: The platform fails to find product-market fit beyond a small enthusiast niche.** Mitigated by the platform's modular architecture serving multiple use cases (pedalboard, recording, synthesis, installation). The platform does not need to dominate any single market to be viable; it needs to find sufficient adoption across multiple adjacent markets. The seed phase explicitly funds validation work to test which markets respond first; the team adjusts strategy based on what early customers tell them.

**Risk: Component supply chain issues affect manufacturing.** Mitigated by the platform's open hardware specification that lets manufacturing happen at multiple facilities and by deliberately conservative component choices (STM32H7 family has been in production for years with multiple suppliers; the codecs and other components have second-source alternatives identified).

**Risk: A major audio brand launches a competing platform.** Mitigated by the open hardware approach (a major brand cannot effectively close a platform that is already published openly) and by the ecosystem effects (community-built mezzanines extend Metapedal's capability faster than any single brand can match). The major brands are also unlikely to launch genuinely-open platforms because their business models depend on proprietary lock-in.

**Risk: The platform's complexity exceeds what musicians want to deal with.** Mitigated by the web-platform configuration approach (musicians configure through familiar UI rather than learning specialized interfaces), the modular product structure (musicians start with one module and add more as needed), and clear positioning for specific use cases (the platform does not market itself as "do everything"; it positions for specific scenarios that align with specific customer needs).

**Risk: The development timeline slips and runway becomes a problem.** Mitigated by the deliberately bounded v1 scope (the platform commits to a specific set of capabilities and explicitly defers others), conservative milestone planning with buffer for delays, the architectural maturity of the platform's design (substantial design work has already been done, reducing the risk of fundamental architectural surprises during development), and the founder's track record of architectural work at the appropriate complexity level for this project.

## Why Now

The technical preconditions for Metapedal exist and did not previously. The STM32H7 microcontroller family delivers enough DSP capability to run sophisticated audio algorithms at pedal-format cost (this was not true five years ago). USB-C high-speed connectivity delivers the inter-module bandwidth at consumer cable cost (this was not true ten years ago). The web platform delivers cross-device configuration UI that runs on every musician's existing devices (this matured for embedded use in the past three years). The Faust DSP language plus the community of audio developers using it provide a development ecosystem for the platform's effects library (this community has grown substantially in the past five years).

The market readiness for Metapedal also exists. The boutique pedal market has grown for fifteen years, demonstrating that musicians value specialty hardware with personality. The modular synth market has grown for ten years, demonstrating that musicians value modularity even with the complexity it brings. The home recording market has grown enormously since 2020, demonstrating that the line between professional and amateur production is blurring. The hardware-startup ecosystem has matured to make small-batch manufacturing accessible to startups in ways that were not possible ten years ago.

Metapedal sits at the intersection of all these trends with an architecturally novel approach that no existing player has shipped. The opportunity is to establish Metapedal as the platform that defines this category, building the ecosystem that makes the platform increasingly valuable over time.

## Why This Team

The founder is a software engineer with substantial architectural experience in distributed systems, embedded software, and complex platform design. Over a multi-decade software career, the founder has developed deep expertise in the technologies Metapedal depends on: embedded firmware architecture, distributed systems protocols, audio DSP at a hobbyist-serious level, web platform development, and the kind of long-horizon platform thinking that an open hardware ecosystem requires. Recent personal architectural work (a Rust-based time-series buffer system, a low-latency distributed actor platform, and the ongoing book-length DiDiDe project on civic technology) demonstrates the capability to develop substantial systems with the architectural rigor that platform work demands.

This is the founder's first commercial product company. The founder has architectural sophistication but not yet a track record of commercial product execution. The team structure addresses this gap explicitly.

**Hardware cofounder (in active discussions).** The platform requires hardware expertise as a core founding capability, not as an outsourced or advisory function. The founder is in active discussions with EE candidates from the local hackspace hardware community, with the goal of converting at least one of them from advisor to cofounder before the seed close. The cofounder brings the analog circuit design, mixed-signal PCB layout, and audio engineering expertise that the software-engineer founder does not have. The hackspace context provides genuine hardware community validation; the platform's design has already been informally reviewed by the EE community whose feedback informed the architecture.

**Hardware advisor circle.** Multiple EE advisors from the local hackspace community have committed to advising on specific aspects of the design: analog circuit design, mixed-signal PCB layout, EMI/EMC considerations, manufacturing process selection. The advisors' commitment signals that the platform's design resonates with people qualified to evaluate it. Even if not all advisors convert to cofounders, their advisory commitment provides expertise across the specific hardware domains the platform requires.

**Audio industry connections.** The founder's musical background (multi-decade involvement with music technology, both as a musician and as a software engineer working on audio-adjacent projects) provides direct understanding of the platform's user needs. The platform's design is informed by actual musician needs rather than by engineering preferences imposed on assumed users. The founder's network in music technology communities provides early-customer connections that the seed phase explicitly funds turning into actual customers.

**Commercial execution capability.** This is the gap the founder is honest about. The first-time-founder profile means the team needs to bring commercial product experience either through cofounder selection or through senior hires funded by the seed round. The pitch's milestone plan reflects this: month 6-12 includes specific hiring for commercial operations expertise, possibly through a head of operations or business operations advisor with prior hardware-startup experience. Investors evaluating this pitch should weight the founder's architectural sophistication against the commercial execution gap and decide whether the staged seed-then-Series-A approach provides sufficient mitigation.

**Long-term commitment.** Open hardware platforms require founders who care about the platform's long-term success rather than near-term exit. The founder's interest in Metapedal is the natural extension of years of architectural thinking about modular systems, music technology, and open ecosystems. The platform is the project the founder has decided to take from architecture to commercial reality after decades of doing related architectural work in adjacent contexts. The commitment is genuine and durable.

The pitch's central proposition is that the founder's architectural capability is real and substantial, the gap in commercial execution is real but addressable through deliberate team structure, and the staged funding approach (modest seed, larger Series A after validation) appropriately matches the founder's profile to the investment risk. Investors who match this profile (early-stage hardware-platform investors, ideally with prior music technology investments, who understand that architectural sophistication is rarer and more valuable than commercial execution which can be hired) are the right audience for this pitch. Investors who require proven commercial-execution founders should evaluate the platform at the Series A stage instead.

---

Metapedal is a platform that solves real problems that musicians actually have, in an architecture that no existing competitor offers, at price points that match traditional pedal economics, with an open ecosystem that compounds value over time. The opportunity is to establish the platform now while the market has not yet recognized that the traditional pedalboard architecture is wrong, and to ride the ecosystem's growth as community-built capabilities extend the platform into markets the core team would not have anticipated.

The ask is $1.5-2.5 million for 18 months of operations to deliver initial demo products, validate the platform with early customers, build the team, and prepare for a Series A focused on commercial scaling. The founder is a first-time commercial founder with substantial architectural capability and a deliberately staged plan that matches investment to demonstrated execution.

The pitch is this. Pedalboards are built wrong. The architecture to build them right has been worked out in detail. The founder brings deep architectural sophistication and is honest about the commercial execution gap that the team structure and staged funding plan are designed to address. This is the right moment to invest at the seed stage in establishing Metapedal as the platform that defines this category, with the understanding that the Series A milestones will validate the founder's commercial execution before larger capital commits.
