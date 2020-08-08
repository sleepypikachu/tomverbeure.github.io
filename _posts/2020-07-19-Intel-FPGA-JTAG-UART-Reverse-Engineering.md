---
layout: post
title: Reverse Engineering the Intel FPGA JTAG UART Protocol
date:   2020-07-19 00:00:00 -0700
categories:
---

* TOC
{:toc}



* Virtual JTAG Notes

* [Intel Virtual JTAG User Guide](https://www.intel.com/content/dam/www/programmable/us/en/pdfs/literature/ug/ug_virtualjtag.pdf)

Instruction register and data registers for the SLD nodes are a subset of the USER1 and USER0 data
registers.

VIR/VID

USER1 -> select the VIR data path
USER0 -> select the VDR data paths

Compilation report doesn't show virtual JTAG entities for Altera-specific cores (e.g. JTAG UART) ?

USER1: [ address | VIR value ]

* Targets virtual IR for either sld_hub or SLD node
* VIR value width (m) = max(VIR of all SLD nodes)
* address width (n) = ceil(log2(nr of SLD node +1 ))
    * Address 0 = SLD hub
* Discover and enumeration -> interrogation of sld_hub to discover m and n
    * SLD hub HUB_INFO instruction -> shift 64 zeros in VIR register
    * 32-bit HUB IP configuraiton register
    * 32-bit SLD node info register for each node

SLD_NODE Discovery and Enumeration:

* Select USER1 (VIR)
* Scan: USER1 -> TDI: 0x4000..000, TDO: 0x2000.000 (135 bits)
    * This means that the USER1 register is only 1 bit wide
    * There are no user virtual JTAGs in there, so that's probably why?
* Select USER0 (VDR)
* Scan: USER0 -> TDI:  0, TDO: 5 (7 bits) 

* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: e (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 6 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 8 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 1 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 8 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 

* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: e (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 6 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: c (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 1 (7 bits) 

* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: e (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 6 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: c (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 2 (7 bits) 

* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: e (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 6 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: c (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 0 (7 bits)       -> Detects rotating 0? Indicates end of number of JTAG UARTs?

I0:
* Scan: USER1 -> TDI: 21, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: a8 (15 bits) 
* Scan: USER1 -> TDI: 20, TDO: 21 (7 bits) 
* Scan: 263: USER0 -> TDI:  9, TDO: 20080200802008020080200802008024 (128 bits) 
* Scan: 268: USER0 -> TDI:  1, TDO: ... (10243 bits) 
* Scan: 273: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 278: USER0 -> TDI:  1, TDO: ... (10243 bits) 
* Scan: 283: USER0 -> TDI:  9, TDO: 24 (6 bits) 
...
* Scan: 328: USER0 -> TDI:  1, TDO: ... (10243 bits) 
* Scan: 333: USER0 -> TDI:  9, TDO: 24 (6 bits) 
...
* Scan: 423: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 428: USER0 -> TDI:  1, TDO: ... (20483 bits) 
* Scan: 433: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 438: USER0 -> TDI:  1, TDO: ... (20483 bits) 
...
* Scan: 583: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 588: USER0 -> TDI:  1, TDO: ... (40963 bits) 
* Scan: 593: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 598: USER0 -> TDI:  1, TDO: ... (40963 bits) 
...
* Scan: 743: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 748: USER0 -> TDI:  1, TDO: ... (81923 bits) 
* Scan: 753: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 758: USER0 -> TDI:  1, TDO: ... (81923 bits) 
...
* Scan: 903: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 908: USER0 -> TDI:  1, TDO: 4 (4 bits) 

* Scan: 913: USER0 -> TDI:  1, TDO: 5 (4 bits) 
* Scan: 930: USER0 -> TDI:  1, TDO: 5 (4 bits) 
* Scan: 935: USER0 -> TDI:  1, TDO: 5 (4 bits) 
...
* Scan: 1076: USER0 -> TDI:  1, TDO: 5 (4 bits) 
* Scan: 1081: USER0 -> TDI:  ..., TDO: ... (10243 bits)         : Scan letter 'D'
* Scan: 1086: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 1091: USER0 -> TDI:  ..., TDO: ... (10243 bits)
* Scan: 1096: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 1101: USER0 -> TDI:  1, TDO: 4 (4 bits) 
* Scan: 1106: USER0 -> TDI:  1, TDO: 5 (4 bits) 


I1:
* Scan: USER1 -> TDI: 41, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: c6 (15 bits) 
* Scan: USER1 -> TDI: 40, TDO: 1 (7 bits) 
* Scan: 263: USER0 -> TDI:  9, TDO: 20080200802008020080200802008024 (128 bits) 
* Scan: 268: USER0 -> TDI:  1, TDO: ... (40963 bits) 
* Scan: 273: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 278: USER0 -> TDI:  1, TDO: ... (40963 bits) 
* Scan: 283: USER0 -> TDI:  9, TDO: 24 (6 bits) 
...
* Scan: 423: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 428: USER0 -> TDI:  1, TDO: ... (81923 bits) 
* Scan: 433: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 438: USER0 -> TDI:  1, TDO: ... (81923 bits) 
...
* Scan: 503: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 508: USER0 -> TDI:  1, TDO: 4 (4 bits) 
* Scan: xxx: USER0 -> TDI:  1, TDO: 5 (4 bits) 
* Scan: 681: USER0 -> TDI:  1, TDO: 5 (4 bits) 
* Scan: 686: USER0 -> TDI:  ..., TDO: ... (40963 bits)         : Scan letter 'D'
* Scan: 691: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 696: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 701: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 706: USER0 -> TDI:  1, TDO: 4 (4 bits) 
* Scan: 711: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 715: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 721: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 725: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 731: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 736: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 741: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 746: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 763: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 767: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 773: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 778: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 783: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 788: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 793: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 798: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 803: USER0 -> TDI:  ..., TDO: ... (40963 bits)
...
* Scan: 1204: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 1209: USER0 -> TDI:  ..., TDO: ... (40963 bits)
* Scan: 1214: USER0 -> TDI:  9, TDO: 24 (6 bits) 
* Scan: 1219: USER0 -> TDI:  1, TDO: 4 (4 bits) 

I2:
* Scan: USER1 -> TDI: 61, TDO: 0 (7 bits) 
* Scan: USER0 -> TDI:  0, TDO: 6a (15 bits) 
* Scan: USER1 -> TDI: 60, TDO: 21 (7 bits) 
* Scan: 263: USER0 -> TDI:  9, TDO: 20080200802008020080200802008024 (128 bits) 
* Scan: 268: USER0 -> TDI:  9, TDO: ... (10243 bits) 






* [OR1K virtual jtag](http://openocd.org/doc/doxygen/html/or1k__tap__vjtag_8c_source.html)


...






* [OR1K virtual jtag](http://openocd.org/doc/doxygen/html/or1k__tap__vjtag_8c_source.html)

