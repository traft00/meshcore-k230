# Phase 1 feasibility plan

Can a Linux-native application on the T-Display K230 reuse MeshCore's networking stack and operate an onboard SX1262? Start with source review and host-side experiments before making a hardware purchase decision.

| Workstream | Evidence to collect |
| --- | --- |
| Hardware | Exact board revision, schematic, radio wiring, power and input/display interfaces |
| MeshCore | Pinned upstream revision, platform dependency inventory, minimal host build or concrete blockers |
| Vendor Linux stack | Pinned SDK and image, toolchain, boot and recovery process, candidate SX1262 transport and UI interfaces |
| Architecture | Recommended design, alternatives, portability gaps, and first bench test |
| Purchase decision | Candidate hardware and required accessories, estimated effort, risks, and a go, defer, or no-go decision |

Record each source URL and revision alongside the finding. Mark what the source confirms, what is inferred, and what requires a bench test. Favor upstreamable changes when practical.

Recommend a **go** when the evidence identifies a plausible radio and build path, preserves the MeshCore protocol implementation, and gives us a reproducible bench plan. **Defer** when a key unknown has a specific follow-up. Recommend **no-go** when a required capability is absent or the port would require a substantial networking-stack rewrite.

After a go decision, validate with a known-good MeshCore peer using matching radio settings. Send a message in each direction and record board revision, software revisions, settings, and logs. Track display, keyboard and battery operation separately from the first interoperability test.
