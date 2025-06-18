# MAAS hardware tutorial

This is a first shot at a very basic hardware tutorial that you can use as a proof of concept for MAAS.  It uses the cheapest possible Mini-PCs or NUCs that you have available. No promises, but it should be possible to get a reasonable result in many cases (PRs welcome).

##  1: Requirements
Going forward, we won't drill-down on the type of small computing device you're attempting to deploy.  Might be a Mini-PC, might be a NUC, might be an old laptop with a robust BIOS (doesn't matter).  So don't get hung up on the term "NUC."  This tutorial may work with whatever you have available.

That said, there are some requirements:

| Requirement | Values | Note |
|----------|----------|------------|
| CPU  | Intel  | arm64 most likely |
| Boot capability      | PXE first     | or PXE can be moved to top of list |
| BMC      | Ideal      | Can work with WakeOnLAN or small PDU |
| Network | Ethernet NIC |  Won't work without one |
| Monitor | Anything you can see | Needed for BIOS settings |
|         |                      | Helpful for debugging |
| Keyboard | Basic keyboard | Needed for BIOS settings |

## 2: Hook up your NUC to the region controller network
Even if you haven't downloaded MAAS yet, just hook your NUC to the same box where you'll be downloading and installing MAAS.  It should work there.
