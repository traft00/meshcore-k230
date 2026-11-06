# Architecture options

The leading hypothesis is a Linux userspace application that reuses MeshCore packet handling and routing. Evaluate a small platform layer for radio I/O, time, entropy, persistent storage and logging, with the standalone UI layered above it.

| Option | What would justify it |
| --- | --- |
| Linux-native MeshCore port with direct radio access | A small platform layer and reliable access to radio interrupts and control signals |
| Linux UI with a separate MeshCore companion radio | The host UI is viable but direct onboard radio integration is blocked or disproportionate in effort |

For the Linux-native path, check whether the vendor radio example supports packet receive and transmit, IRQ handling, BUSY timeouts and reset. Test whether the upstream core can compile with a fake radio before connecting it to the vendor transport. Keep display and keyboard work out of the first over-the-air test.

Record the selected architecture, source revisions, rejected options and remaining risks after the hardware, MeshCore and vendor SDK reviews.
