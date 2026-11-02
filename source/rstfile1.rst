.. START:section:0:0
Objectives
==========

.. END:section:0:0

.. START:list:0:0:0

.. LIST_META: {"type": "bullet", "head": "", "items": ["Identify the exact board, MACHINE and BSP release before you open any source file.", "Tell kernel configuration, device tree and boot selection apart.", "Recognise .dts, .dtsi, .dtb, .dtso and .dtbo files and know what each is for.", "Find the board entry point and follow every include down to the SoC file.", "Check that a device tree is registered in the kernel Makefile and built.", "• Find which device tree a Yocto build produces and which one a running board received."]}

- Identify the exact board, MACHINE and BSP release before you open any source file.
- Tell kernel configuration, device tree and boot selection apart.
- Recognise .dts, .dtsi, .dtb, .dtso and .dtbo files and know what each is for.
- Find the board entry point and follow every include down to the SoC file.
- Check that a device tree is registered in the kernel Makefile and built.
- • Find which device tree a Yocto build produces and which one a running board received.

.. END:list:0:0:0

.. START:section:0:1
Board Configuration Overview
============================

.. END:section:0:1

.. START:main_content:0:1
Linux must know which hardware is fitted and how it is connected. Two things provide that knowledge, and a third decides what is actually used at boot.

.. END:main_content:0:1

.. START:table:0:1:0

.. csv-table:: Table 1. Kernel configuration, device tree and boot selection compared
   :header: "Area", "Question it answers", "Typical files"
   :align: center
   :class: align-center

   "Kernel configuration", "Is a feature or driver built into the kernel?", "``.config``, defconfig, config fragments, Kconfig"
   "Device tree", "Which hardware is present and which resources does it use?", "``.dts``, ``.dtsi``, ``.dtso`` and binding documents"
   "Boot selection", "Which compiled device tree does the bootloader load?", "Bootloader environment, boot script, FIT image and boot configuration"

.. END:table:0:1:0

The figure shows the whole path. Source files are combined by the device-tree compiler into a binary, the bootloader passes that binary to Linux, and the running system shows the result.

.. START:important:0:1:2

.. important::

   | **A driver and a node must both exist**
   | A device-tree node cannot create a driver that was not built, and a built driver does not probe if its node is disabled or has wrong resources. Check both the kernel configuration and the device tree.

.. END:important:0:1:2

.. START:section:0:2
Identify the Board
==================

.. END:section:0:2

.. START:main_content:0:2
A file for a similar SoC or a similar product name may be wrong for your board. Write down these facts before you search the source

.. END:main_content:0:2

.. START:table:0:2:0

.. csv-table:: Table 2. Information to record before searching
   :header: "Item", "What to write down"
   :align: center
   :class: align-center

   "SoM", "PHYTEC module name and hardware revision."
   "Carrier board or kit", "Carrier or development-kit name and revision."
   "SoC", "Exact processor, for example i.MX 8M Plus or AM625."
   "BSP release", "Release name and release-notes version, for example PD26.1.0."
   "Yocto MACHINE", "The machine name from the release notes, for example ``phyboard-lyra-am62xx-3`` or ``phyboard-pollux-imx8mp-3``."
   "Boot medium", "SD card, eMMC, QSPI NOR, NAND, USB or network."
   "Kernel source", "Repository, tag and the output of ``git describe``."

.. END:table:0:2:0

PHYTEC release notes list, for every MACHINE, the SoM and carrier part, the bootloader defconfigs, the kernel defconfig and fragments, the default device tree and the overlays. For example, the AM62x release notes name phytec_ti_defconfig with phytec_ti_platform.config and the default device tree k3-am625-phyboard-lyra-rdk.dtb for phyboard-lyra-am62xx-2. [1] That table is the quickest way to find the right files for your hardware.

.. START:warning:0:2:2

.. warning::

   | **Product name and MACHINE name are not the same**
   | The kit name printed on a box is not always the Yocto MACHINE. PHYTEC also keeps older variants, for example phyboard-lyra-am62xx-1 and -2 are marked deprecated while -3 is the default kit for high-security (HS-FS) silicon. [1] Use the value from the release notes of your release.

.. END:warning:0:2:2

.. START:section:0:3
Kernel Configuration Files
==========================

.. END:section:0:3

.. START:main_content:0:3
These files decide which features are compiled. They do not describe the wiring of a particular carrier board.

.. END:main_content:0:3

.. START:table:0:3:0

.. csv-table:: Table 3. Kernel configuration terms
   :header: "File or term", "Meaning", "Where to look"
   :align: center
   :class: align-center

   "``.config``", "The final configuration of one kernel build, for example ``CONFIG_SPI=y``.", "The kernel build directory. In a Yocto build, it is in the kernel recipe work directory."
   "defconfig", "A saved starting configuration for a platform.", "``arch/<arm|arm64>/configs/``"
   "Config fragment", "A configuration file that adds or changes options on top of a defconfig.", "Next to the defconfig, for example ``phytec_ti_platform.config``."
   "Kconfig", "Files that define options, menus and dependencies.", "Across the kernel source, mainly in ``drivers/`` and ``arch/``."
   "CONFIG_ option", "One setting. ``y`` means built in, ``m`` means module, and unset means disabled.", "Search the final ``.config``."

.. END:table:0:3:0

PHYTEC defconfigs and fragments seen in the supplied trees:

.. START:table:0:3:2

.. csv-table:: Table 4. PHYTEC kernel configuration files in the supplied trees
   :header: "Platform", "Files under ``arch/<arch>/configs/``"
   :align: center
   :class: align-center

   "i.MX 8M Plus", "``imx8_phytec_defconfig``, ``imx8_phytec_distro.config``, ``imx8_phytec_platform.config``"
   "i.MX 6UL", "``imx_v6_v7_defconfig`` with ``imx6_phytec_distro.config``, ``imx6_phytec_machine.config`` and ``imx6_phytec_platform.config``"
   "AM335x", "``am335x_phytec_defconfig``"
   "AM62x, AM64x", "``phytec_ti_defconfig``, ``phytec_ti_platform.config``; the AM62x tree also has ``phytec_ti_rt.config`` for a real-time kernel."

.. END:table:0:3:2

Check a final configuration in the build directory, or on a running board when the kernel exports it:

.. START:code:0:3:4

.. code-block:: console

   $ grep -E "^CONFIG_(I2C|SPI|GPIO|CAN)(=| )" .config
   $ grep -E "^# CONFIG_(I2C|SPI|GPIO|CAN) is not set" .config
   $ find arch -path "*/configs/*" -name "*phytec*" | sort
   $ zcat /proc/config.gz | grep CONFIG_SPI    # only if the kernel provides /proc/config.gz

.. END:code:0:3:4

Names differ between releases. Use the defconfig and fragments named in the release notes, not a name you remember from another board.

.. START:section:0:4
Device Tree File Types
======================

.. END:section:0:4

.. START:main_content:0:4
Source files are text. Compiled files are binary files used by the boot flow.

.. END:main_content:0:4

.. START:table:0:4:0

.. csv-table:: Table 5. Device-tree file types
   :header: "Type", "What it is", "Normal use"
   :align: center
   :class: align-center

   "``.dts``", "Device-tree source for a board.", "Entry point for one board or product."
   "``.dtsi``", "Reusable include source.", "Shared SoC, SoM or carrier definitions."
   "``.dtb``", "Compiled device-tree blob.", "Loaded by the bootloader and passed to Linux."
   "``.dtso``", "Overlay source.", "Optional hardware or a small change to the base tree."
   "``.dtbo``", "Compiled overlay.", "Applied only when the boot flow supports and selects it."
   "Binding", "Document that defines a node.", "Lists required properties for a device type."

.. END:table:0:4:0

A .dtsi file is included by another source file. The bootloader does not load it directly. The build combines the includes and compiles the result into a .dtb.

.. START:tip:0:4:2

.. tip::

   | **Where to start reading**
   | Look for the .dts first. It is the board entry point. Then follow the .dtsi files it includes.

.. END:tip:0:4:2

.. START:section:0:5
How PHYTEC Device Trees Are Organized
=====================================

.. END:section:0:5

.. START:main_content:0:5
PHYTEC splits the hardware description into levels so that shared parts are reused by several boards.

.. END:main_content:0:5

.. START:table:0:5:0

.. csv-table:: Table 6. Typical levels of a PHYTEC device tree
   :header: "Level", "Describes", "Examples"
   :align: center
   :class: align-center

   "SoC (vendor) file", "Hardware blocks inside the processor.", "UART, I2C, SPI, Ethernet MAC, interrupt controller."
   "SoM file", "Parts mounted on the PHYTEC module.", "RAM, PMIC, eMMC, SoM Ethernet PHY, SPI NOR, RTC and EEPROM."
   "Carrier board file", "Parts on the carrier and what is routed to it.", "Connectors, Ethernet PHY, CAN, display, LEDs and keys."
   "Overlay", "An optional feature or a change to the SoM or carrier.", "Camera, display, expansion board or disabled SoM component."

.. END:table:0:5:0

Two include relationships were checked in PHYTEC and mainline sources. The i.MX 8M Plus board file includes the SoM file, which includes the SoC file.On newer kernels the AM62x board file includes the SoC, SoM and carrier files directly.

.. START:warning:0:5:3

.. warning::

   | **A shared file can change several boards**
   | A .dtsi is used by more than one board. A SoM file change affects every carrier that includes it, and the AM62x carrier file is shared by the AM625 and AM62A7 boards on newer kernels. If a change is for one board only, use the board .dts or an overlay.

.. END:warning:0:5:3

The model and compatible values are what you read back on the board to confirm the match.

.. START:note:0:5:5

.. note::

   | **The layout depends on the kernel version**
   | In the supplied 6.1.80 AM62x tree the carrier devices are still inside the board .dts. The file k3-am62x-phyboard-lyra.dtsi was added when PHYTEC moved the carrier part out so that the phyCORE-AM62Ax SoM could reuse it, and it is present in the newer supplied AM64x tree. Always read your own tree.

.. END:note:0:5:5

.. START:section:0:6
Where the Files Live per Platform
=================================

.. END:section:0:6

.. START:main_content:0:6
The next table is built from the supplied kernel trees. Search with find in your own tree because paths change with the kernel version.

.. END:main_content:0:6

.. START:table:0:6:0

.. csv-table:: Table 8. PHYTEC device-tree files in the supplied trees
   :header: "Platform", "Board and carrier files", "SoM files", "Overlays (``.dtso``)"
   :align: center
   :class: align-center

   "i.MX 8M Plus — ``arch/arm64/boot/dts/freescale/``", "``imx8mp-phyboard-pollux-rdk.dts``, plus camera and FPD-Link ``.dtsi`` files.", "``imx8mp-phycore-som.dtsi``", "``imx8mp-phycore-no-eth``, ``-no-rtc``, ``-no-spiflash``, ``-rpmsg``; ``imx8mp-phyboard-pollux-peb-av-10``, ``-peb-wlbt-05`` and camera overlays."
   "i.MX 6UL — ``arch/arm/boot/dts/nxp/imx/``", "``imx6ul-phytec-segin-ff-rdk-nand.dts``, ``-ff-rdk-emmc.dts``, ``imx6ul-phytec-segin.dtsi``", "``imx6ul-phytec-phycore-som.dtsi``", "Expansion ``.dtsi`` files such as ``...-peb-av-02`` and ``...-peb-wlbt-05``."
   "AM335x — ``arch/arm/boot/dts/ti/omap/``", "``am335x-phycore-rdk.dts``, ``am335x-phycore-emmc-rdk.dts``", "``am335x-phycore-som.dtsi``", "None seen in the listing."
   "AM62x (6.1.80) — ``arch/arm64/boot/dts/ti/``", "``k3-am625-phyboard-lyra-rdk.dts``, ``k3-am62a7-phyboard-lyra-rdk.dts``", "``k3-am62-phycore-som.dtsi``, ``k3-am62a-phycore-som.dtsi``", "``k3-am62-phyboard-lyra-gpio-fan``, ``-m2-lwb5p``, ``-oldi-lcd185``, ``-vm016-mipi``, ``-x17-spi0``, ``-x39-spi0``."
   "AM64x — ``arch/arm64/boot/dts/ti/``", "``k3-am642-phyboard-electra-rdk.dts``", "``k3-am64-phycore-som.dtsi``", "``k3-am642-phyboard-electra-gpio-fan``, ``-pcie-usb2``, ``-peb-c-010``, ``k3-am6xx-phycore-disable-eth-phy`` and similar overlays."

.. END:table:0:6:0

.. START:tip:0:6:1

.. tip::

   | **Overlay names change between releases**
   | The AM62x release notes list overlays such as k3-am6-phycore-disable-spi-nor.dtbo. [1] The newer AM64x tree names the same kind of overlay k3-am6xx-phycore-disable-spi-nor.dtso. Search by function, not by an exact old name.

.. END:tip:0:6:1

.. START:warning:0:6:2

.. warning::

   | **Your tree may hold files from another checkout**
   | The AM335x listing shows device trees in the old flat arch/arm/boot/dts/ path and also in the new vendor folders. Run git status and make sure the tree matches one tag before you trace includes.

.. END:warning:0:6:2

.. START:section:0:7
Find and Trace the Board Files
==============================

.. END:section:0:7

.. START:main_content:0:7
Run these commands from the kernel source root. The names in angle brackets are placeholders.

.. END:main_content:0:7

Confirm the Source
------------------

| $ git status --short --branch
| $ git describe --tags --always
| $ make kernelrelease    # needs a configured tree

Compare the result with the release notes.

Search for the Board Files
--------------------------

.. START:code:0:7:2

.. code-block:: console

   $ find arch -type f \( -name "*.dts" -o -name "*.dtsi" -o -name "*.dtso" \) | grep -Ei "phy(board|core|flex)|phytec"
   $ find arch/arm64/boot/dts/ti -name "*phyboard*" -o -name "*phycore*"
   $ grep -Rn "model = " arch/arm64/boot/dts/freescale/imx8mp-phyboard-pollux-rdk.dts

.. END:code:0:7:2

Trace the Includes
------------------

.. START:code:0:7:4

.. code-block:: console

   $ grep -nE "^#include|/include/" <board>.dts
   $ grep -nE "^#include|/include/" <som>.dtsi
   $ grep -nE "model =|compatible =" <board>.dts

.. END:code:0:7:4

Follow each included file in turn. The device-tree compiler also accepts /include/ lines, so search for both forms.

Check the Build Rule
--------------------

| $ grep -n "phyboard" arch/arm64/boot/dts/ti/Makefile
| $ grep -n "pollux" arch/arm64/boot/dts/freescale/Makefile

A board is built only when its DTB is listed, for example dtb-$(CONFIG_ARCH_K3) += k3-am625-phyboard-lyra-rdk.dtb. [4] Overlays need the base DTB to be built with symbols. PHYTEC added DTC_FLAGS_k3-am625-phyboard-lyra-rdk += -@ for that purpose.

Read the Final Merged Tree
--------------------------

A compiled DTB contains the merged result of all includes. Decompile it to see what the kernel really receives:

.. START:code:0:7:8

.. code-block:: console

   $ make ti/k3-am625-phyboard-lyra-rdk.dtb
   $ dtc -I dtb -O dts -o /tmp/board-merged.dts arch/arm64/boot/dts/ti/k3-am625-phyboard-lyra-rdk.dtb
   $ grep -n "status\|compatible" /tmp/board-merged.dts | head -n 60

.. END:code:0:7:8

Build the DTB in the same way as BSP-02. Use your own board target. If dtc is not installed, the kernel builds its own copy at scripts/dtc/dtc.

.. START:section:0:8
Find the Selected Device Tree in Yocto
======================================

.. END:section:0:8

.. START:main_content:0:8
The source tree shows which device trees exist. The Yocto machine configuration shows which ones the build compiles and deploys. The release notes already list the default device tree and overlays per MACHINE.

.. END:main_content:0:8

.. START:code:0:8:0

.. code-block:: console

   # inside an initialised PHYTEC build environment
   $ bitbake -e virtual/kernel | grep -E "^(MACHINE|KERNEL_DEVICETREE|KBUILD_DEFCONFIG)="
   $ grep -Rn "KERNEL_DEVICETREE" sources/meta-phytec 2>/dev/null
   $ find tmp/deploy/images -type f ( -name "*.dtb" -o -name "*.dtbo" -o -name "fitImage*" ) | sort

.. END:code:0:8:0

The variable names and the deploy folder can differ between releases. Read the surrounding lines of any match to see whether a value is set, appended or overridden.

.. START:important:0:8:2

.. important::

   | **Edited source and deployed file must belong to the same build**
   | A correct source edit has no effect if an older DTB or FIT image is written to the boot medium. Newer NXP releases boot a FIT image that packs the kernel, device trees and overlays, so finding a DTB in the deploy folder is only part of the check.

.. END:important:0:8:2

.. START:section:0:9
Check the Device Tree on a Running Board
========================================

.. END:section:0:9

.. START:main_content:0:9
Linux exposes the device tree it received.

.. END:main_content:0:9

.. START:code:0:9:0

.. code-block:: console

   $ cat /proc/device-tree/model; echo
   $ tr "\0" "\n" < /proc/device-tree/compatible
   $ ls /sys/firmware/devicetree/base | head
   $ dtc -I fs -O dts -o /tmp/running.dts /sys/firmware/devicetree/base
   $ grep -n "status = " /tmp/running.dts | head -n 40

.. END:code:0:9:0

.. START:list:0:9:1

.. LIST_META: {"type": "bullet", "head": "", "items": ["The model and compatible values identify the board description, but not the original file name.", "Write the decompiled file to a writable folder such as /tmp. The live tree is read-only and must not be edited.", "If the board name is wrong, the bootloader loaded another DTB or FIT configuration. Check the boot log and the boot configuration (BSP-07 and BSP-08)."]}

- The model and compatible values identify the board description, but not the original file name.
- Write the decompiled file to a writable folder such as /tmp. The live tree is read-only and must not be edited.
- If the board name is wrong, the bootloader loaded another DTB or FIT configuration. Check the boot log and the boot configuration (BSP-07 and BSP-08).

.. END:list:0:9:1

.. START:note:0:9:2

.. note::

   | **Overlays are not active just because the file exists**
   | Copying a .dtbo to the filesystem does nothing by itself. The bootloader and BSP must apply it, and PHYTEC documents the overlay selection per platform.

.. END:note:0:9:2

.. START:section:0:10
Device Tree Properties You Will See
===================================

.. END:section:0:10

.. START:table:0:10:0

.. csv-table:: Table 9. Common properties seen while tracing a node
   :header: "Property", "Meaning"
   :align: center
   :class: align-center

   "``status``", "``okay`` enables a node and ``disabled`` prevents normal use. A shared SoC file often disables a node until the board file enables it."
   "``compatible``", "Names the hardware so the kernel can match a driver."
   "``reg``", "Register address or bus address of the device."
   "``interrupts``", "Interrupt connection."
   "``pinctrl-names``, ``pinctrl-0``", "Pin configuration states used by the device (BSP-04)."
   "``clocks``, ``resets``, ``reset-gpios``", "Clock inputs and reset lines the device needs."
   "``*-supply``", "Regulator that powers the device."
   "``aliases``", "Fixed names such as ``serial2`` or ``mmc1`` that give stable device numbers."

.. END:table:0:10:0

.. START:section:0:11
Safe Workflow for a Device-Tree Change
======================================

.. END:section:0:11

.. START:main_content:0:11
| Confirm SoM, carrier, revision, BSP release and MACHINE.
| Confirm which DTB or FIT configuration the board boots today.
| Find the board ``.dts`` and trace its includes.
| Find the exact node in the final merged tree.
| Check the hardware manual and schematic before editing.
| Make one small change in the board file or an overlay, not in a shared file.
| Build through the documented workflow.
| Deploy the matching DTB or FIT image and keep the old files for rollback.
| Read the boot log and the live device tree.
| Run a functional test of the changed hardware and of one unrelated peripheral.

.. END:main_content:0:11

.. START:section:0:12
Common Mistakes and First Checks
================================

.. END:section:0:12

.. START:table:0:12:0

.. csv-table:: Table 10. Symptoms and first checks
   :header: "Symptom", "Likely cause", "First check"
   :align: center
   :class: align-center

   "The edited file has no effect.", "Wrong DTB, wrong include or an old artifact on the boot medium.", "``KERNEL_DEVICETREE``, deploy folder and ``/proc/device-tree/model``."
   "The expected board ``.dts`` is not found.", "Wrong repository, tag or architecture folder.", "Run ``git describe``, then search all ``arch/*/boot/dts`` directories."
   "A peripheral is missing.", "The node is disabled or not in the final tree.", "Check ``status`` in the merged tree and the live tree."
   "The driver reports a missing resource.", "Clock, regulator, reset, GPIO or pinctrl entry is missing.", "Check ``dmesg`` and the device binding."
   "The build does not create the DTB.", "The file is not listed in the Makefile or in the Yocto variable.", "Check the kernel DTS Makefile and ``KERNEL_DEVICETREE``."
   "Several boards change after one edit.", "A shared ``.dtsi`` was edited.", "Move the change to the board file or an overlay."
   "The kernel option is missing.", "Wrong ``.config`` or a fragment was not applied.", "Check the ``.config`` of the actual build and the release-notes defconfig."

.. END:table:0:12:0

.. START:section:0:13
Board File Checklist
====================

.. END:section:0:13

.. START:list:0:13:0

.. LIST_META: {"type": "bullet", "head": "", "items": ["SoM, carrier, revision, BSP release and MACHINE recorded.", "Kernel repository and tag confirmed with ``git describe``.", "Release-notes defconfig, fragments, default DTB and overlays noted.", "Board ``.dts`` found and all includes traced to the SoC file.", "Node status, compatible and resources checked in the merged tree.", "DTB listed in the kernel Makefile and in the Yocto configuration.", "Deployed DTB or FIT image matches the build.", "Live model and compatible checked on the board.", "Overlay support and selection confirmed before relying on an overlay."]}

- SoM, carrier, revision, BSP release and MACHINE recorded.
- Kernel repository and tag confirmed with ``git describe``.
- Release-notes defconfig, fragments, default DTB and overlays noted.
- Board ``.dts`` found and all includes traced to the SoC file.
- Node status, compatible and resources checked in the merged tree.
- DTB listed in the kernel Makefile and in the Yocto configuration.
- Deployed DTB or FIT image matches the build.
- Live model and compatible checked on the board.
- Overlay support and selection confirmed before relying on an overlay.

.. END:list:0:13:0

.. START:section:0:14
Glossary
========

.. END:section:0:14

.. START:table:0:14:0

.. csv-table:: Table 11. Terms used in this guide
   :header: "Term", "Meaning"
   :align: center
   :class: align-center

   "Board entry point", "The ``.dts`` file that represents one complete board."
   "Include", "A ``#include`` or ``/include/`` line that pulls another source file into a device tree."
   "Merged tree", "The final tree after all includes and overlays are combined."
   "Overlay", "A small device tree applied on top of a base tree."
   "FIT image", "A single file that can hold the kernel, device trees and overlays."
   "MACHINE", "The Yocto name of one board configuration."
   "compatible", "The string used to match a driver or a board description."
   "DTC", "The device-tree compiler."

.. END:table:0:14:0

.. START:section:0:15
References
==========

.. END:section:0:15

.. START:main_content:0:15
PHYTEC phyCORE-AM62x BSP release notes PD23.2.0: `Machine names, kernel configuration, default device tree and overlays <https://docs.phytec.com/projects/yocto-phycore-am62x/en/latest/releasenotes/pd23.2.0.html>`_.

i.MX 8M Plus phyBOARD-Pollux device-tree source: `Board includes, model and compatible strings <https://src.rcs.uwaterloo.ca/xref/freebsd-src/sys/contrib/device-tree/src/arm64/freescale/imx8mp-phyboard-pollux-rdk.dts>`_.

Linux kernel patch for the AM62x phyBOARD-Lyra carrier board: `Device-tree include relationships <https://lists.infradead.org/pipermail/linux-arm-kernel/2024-June/938559.html>`_.

Linux kernel patch for phyBOARD-Lyra-AM625: `Device-tree Makefile registration and SoM identity strings <https://lists.infradead.org/pipermail/linux-arm-kernel/2023-May/831295.html>`_.

PHYTEC AM625 device-tree overlay patch: `DTC symbol flag for overlay support <https://yhbt.net/lore/lkml/20240419193552.3090343-1-nmorrisson@phytec.com/>`_.

PHYTEC BSP documentation source: `Yocto BSP documentation repository <https://github.com/phytec/doc-bsp-yocto>`_.

Linux kernel documentation: `Linux and the Devicetree <https://docs.kernel.org/devicetree/usage-model.html>`_.

Devicetree Specification: `Syntax and property definitions <https://www.devicetree.org/specifications/>`_.

.. END:main_content:0:15
