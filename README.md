# MAAS hardware tutorial

This is a first shot at a very basic hardware tutorial that you can use as a proof of concept for MAAS.  It uses the cheapest possible Mini-PCs or NUCs that you have available. No promises, but it should be possible to get a reasonable result in many cases (PRs welcome).

## Step 1: Set up your NUC
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
