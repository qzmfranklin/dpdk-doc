Introduction to PCI
===================

Resources
---------

    - `PCI Express Specification 3.1
      <https://www.slac.stanford.edu/grp/lcls/controls/docs/hw/uTCA/Standards/CB-PCI_Express_Base_r3.1_October8-2014.pdf>`_.

    - The `LDP book <http://www.tldp.org/LDP/tlk/dd/pci.html>`_ provides a lot
      of valuable details about PCI.

    - `O'Reily <http://www.oreilly.com/openbook/linuxdrive3/book/ch12.pdf>`_ is
      the authoritative reference for writing PCI device drivers.

    - `Kernel archive
      <https://www.kernel.org/doc/Documentation/ABI/testing/sysfs-bus-pci>`_ on
      :code:`/sys/bus/pci/...`.


Basic Concepts
--------------

PCI stands for *Peripheral Component Interconnect*. It is a set of protocols by
which peripheral devices can use to connect to the CPU, DRAM, and other
peripheral devices either directly (e.g., `DMA
<http://www.oreilly.com/openbook/linuxdrive3/book/ch15.pdf>`_) or indirectly.

The CPU connects to a PCI bus, the root PCI bus, or the *root bus* for short.
Devices that conform to the PCI standard, i.e., *PCI devices*, connect to the
root bus. A PCI device can have up to eight *PCI functions*.

A PCI function, or simply *function* for short, is a set of hardware components
on a PCI device that performs a specific function, such as ethernet card, audio
card, cd/dvd drive. Some common examples of multi-functional PCI device include
    - a cd/dvd drive with an audio card on it, and
    - a multi-port ethernet card, where each port is a PCI function.

A PCI function is the smallest entity in the PCI hierarchy. PCI drivers are
written for PCI functions. A lot of materials use the word 'device' to refer to
what should be more precisely referred to as 'function'. This doc, though, will
use 'function' for consistency.

Each functions is uniquely identified by its *PCI address*, a 16-bit number of
the following format:

+---------+----------------+----------+------+
| Offset  |     15-8       |   7-3    |  2-0 |
+=========+================+==========+======+
| field   |      bus       | device   | func |
+---------+----------------+----------+------+

Usually, PCI addresses are written in the `BDF
<https://wiki.xen.org/wiki/Bus:Device.Function_(BDF)_Notation>`_ (bus, device,
function) format. For example, :code:`02:1a.3` means function :code:`3` on
device :code:`1a` on bus :code:`02`.

The PCI standard permits
    - each system to host up to 256 buses,
    - each bus to host up to 32 devices, and
    - each device to host up to 8 functions.

For simplicity, this doc will refer to PCI buses/devices/functions as
buses/devices/functions when there is no ambiguity.

An import observation is that a system of PCI buses, devices, and function make
up a tree where the CPU is the root node, PCI buses and PCI-PCI bridges are the
edges, and PCI devices/function are the end nodes. It is possible to iterate
through all devices on the PCI system with a depth first traversal.


Address Spaces
--------------

A function has up to three address spaces:
    - configuration space
    - I/O
    - memory

This `wiki page <https://en.wikipedia.org/wiki/Memory-mapped_I/O>`_ is a pretty
good explanation of Memory Mapped I/O and Port Mapped I/O. This section only
talks about the configuration space.

The *configuration space*, or the *standard registers* is a set of hardware
registers whose total length is 256 bytes that every PCI device *MUST* implement
according to the following layout:

+---------+-------------+---------------+---------------+---------------+
| Offset  | 31-24       |   23-16       |   15-8        | 7-0           |
+=========+=============+===============+===============+===============+
|   00    | Device ID                   |  Vendor ID                    |
+---------+-------------+---------------+---------------+---------------+
|   04    | Status                      |  Command                      |
+---------+-------------+---------------+---------------+---------------+
|   08    | Class       | Subclass      | Prog IF       | Revision ID   |
+---------+-------------+---------------+---------------+---------------+
|   0c    | BIST        | Header Type   | Latency Timer | Cacheline Size|
+---------+-------------+---------------+---------------+---------------+
|   10    | Base Address Register 0 (BAR0)                              |
+---------+-------------+---------------+---------------+---------------+
|   14    | Base Address Register 1 (BAR1)                              |
+---------+-------------+---------------+---------------+---------------+
|   18    | Base Address Register 2 (BAR2)                              |
+---------+-------------+---------------+---------------+---------------+
|   1c    | Base Address Register 3 (BAR3)                              |
+---------+-------------+---------------+---------------+---------------+
|   20    | Base Address Register 4 (BAR4)                              |
+---------+-------------+---------------+---------------+---------------+
|   24    | Base Address Register 5 (BAR5)                              |
+---------+-------------+---------------+---------------+---------------+
|   28    | Cardbus CIS Pointer                                         |
+---------+-------------+---------------+---------------+---------------+
|   2c    | Subsystem ID                | Sybsystem Vendor ID           |
+---------+-------------+---------------+---------------+---------------+
|   30    | Expansion ROM Base Address                                  |
+---------+-------------+---------------+---------------+---------------+
|   34    | Reserved                                    | Capability Ptr|
+---------+-------------+---------------+---------------+---------------+
|   38    | Reserved                                                    |
+---------+-------------+---------------+---------------+---------------+
|   3c    | Max Lat     | Min Grant     | Interrpt PIN  | Interrupt Line|
+---------+-------------+---------------+---------------+---------------+

All fields are explained `here
<http://wiki.osdev.org/PCI#Configuration_Space>`_.

In Linux, :code:`lspci -s <bus:dev.func> -x -v` (no :code:`sudo`) displays a
hexdump of the configuration space plus the decoded, human-readable explanation
of the state [instruction and example needed].

The six *Base Address Registers*, or BARs are used to tell the kernel
    - where, in the host physical memeory, the I/O and memory address spaces of
      the PCI function, are mapped to, and
    - the sizes of address spaces.


Kernel Representation
---------------------

At last, pleae remember that
    - :code:`struct pci_dev` represents a PCI function in the kernel,
    - :code:`struct pci_dev` is defined in `pci.h
      <http://lxr.free-electrons.com/source/include/linux/pci.h>`_.

Exercises
---------

Exercise 1
~~~~~~~~~~

On a Linux machine, do the following:

    #.  Use :code:`lspci -s <bus:dev.func> -x` to hexdump the configuration
        spaces of a PCI function. Note that
            - the :code:`lspci` program should be invoked without :code:`sudo`,
              and
            - the configuration space is little-endian.
    #.  From the hexdump, find the vendor ID and device ID. According to `this
        table <http://pciids.sourceforge.net/v2.2/pci.ids>`_, what is the name
        of the vendor?
    #.  From the hexdump, find the class and subclass. According `this table
        <https://www-s.acm.illinois.edu/sigops/2007/roll_your_own/7.c.1.html>`_,
        what does this PCI function do?
    #.  From the hexdump, find the six BARs, for each BAR, figure out the
        following (all the ingredients you need can be found `here
        <http://wiki.osdev.org/PCI#Base_Address_Registers>`_:
            - Is the BAR mapped to an I/O address space or a memory address
              space?
            - Is the BAR mapped to a 64-bit address space or 32-bit address
              space?
            - Is the BAR prefetchable?
            - The starting physical address of the address space that this BAR
              points to.
    #.  Compare your parsed result with the true parsed result, i.e., as
        displayed by :code:`lspci -s <bus:dev.func> -x -v`.
    #.  How does the kernel know the size of each address space? Hint: Write all
        :code:`1` to the BAR, then read its value, as described `here
        <http://wiki.osdev.org/PCI#Base_Address_Registers>`_.

The goal of this exercise is to gain visibility into the configuration space.

Exercise 2
~~~~~~~~~~

In exercise 1, the :code:`lspci` program was invoked without
:code:`sudo`. That is because :code:`lspci` gets information from a globally
readable psedudo file system, `sysfs
<https://www.kernel.org/doc/Documentation/filesystems/sysfs.txt>`_. Each PCI
funcion has its own subdirectory, :code:`/sys/bus/pci/<bus:dev.func>`. Please do
the following:
    #.  Go to the :code:`/sys/bus/pci/<bus:dev.func>` of the function you looked
        at in exercise 1.
    #.  Explore the directory. Use :code:`tree` to dump the tree.
    #.  Find the configuration space of this function.
    #.  Confirm that the configuration space found here is the same with the one
        dumped by :code:`lspci -x`.

Exercise 3
~~~~~~~~~~

Repeat exercise 1 and 2 for a few different PCI functions.


Exercise 4
~~~~~~~~~~

Look for :code:`struct pci_dev` in `pci.h
<http://lxr.free-electrons.com/source/include/linux/pci.h>`_, read through the
source code carefully. The goal is to get familiar with the kernel
representation of a PCI device. Later, we will be talking about certain fields
in the :code:`struct pci_dev`. You need to be comfortable looking at the source
code.
