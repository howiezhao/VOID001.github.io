---
title: Wine Packet.dll wpcap.dll TODO List
---

* [x] write Makefile.in
* [x] add packet.spec
* [x] reconfigure, build the new wine
* [x] Write functions
 * [x] First API PacketGetAdapterNames
 * [x] Write Testcase for PacketGetAdapterNames
 * [ ] Next API PacketOpenAdapter
 * [ ] Write Testcase for PacketOpenAdapter


# Stub notes
* trafficwatch.exe cannot work on wine 
 - when using wine wpcap.dll wine_pcap_datalink() return a wrong value 0x71 ( This val cannot find in pcap source )
 - when using windows wpcap.dll and use wine packet.dll it crash and say 
`   1 0x7e7732ef __wine_spec_unimplemented_stub+0x2e(module="packet.dll", function="PacketOpenAdapter") [/home/void001/GitRepos/wine_opensource_dev/Wine_wpcap/winesrc/dlls/winecrt0/stub.c:34] in packet (0x0033a314) `
 - when using both wpcap.dll and packet.dll native(Windows dll) it cannot find interface, Cause when lookup the register key it will fail, Try to fake it with the correspond register key
 (Information about regkey entry start with "4d36e972" https://technet.microsoft.com/en-us/library/cc780532(v=ws.10).aspx)
