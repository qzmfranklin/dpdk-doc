Introduction to PCI Driver
==========================

Resources
---------

`O'Reily <http://www.oreilly.com/openbook/linuxdrive3/book/ch12.pdf>`_

Prerequisite
------------
:doc:`./pci-intro`


Driver
------

PCI drivers, when loaded via `modprobe`, register themselves to a :code:`static`
list of `struct pci_driver` using `int pci_register_driver(struct pci_driver
*drv)`


The PCI standard desribes [citation needed] a mechanism to allow OS or BIOS to
auto-detect all connected PCI functions. Within the rest of this section, this
mechanism is referred to as *PCI scanning*, or simply *scanning*.

A scanning traverses the PCI system, which is a tree, in a depth first manner.
When the Linux kernel scans PCI devices, it creates a :code:`struct pci_dev` for
each device and adds it to a :code:`static` list of :code:`struct pci_dev`.

When a new device, as in a :code:`struct pci_dev`, is added,
