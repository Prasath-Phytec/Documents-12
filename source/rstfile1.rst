.. START:section:0:0
Objectives
==========

.. END:section:0:0

.. START:list:0:0:0

.. LIST_META: {"type": "bullet", "head": "", "items": ["Explain what the bootloader does and which boot stages and files belong to a PHYTEC board.", "Check which bootloader your release uses instead of assuming.", "Find the bootloader repository, tag and dependent firmware for your BSP release.", "Clone, configure and build U-Boot for TI AM62x and NXP i.MX 8M Plus.", "Install the boot files on an SD card and understand the risks of eMMC and flash writes.", "Make a supported change, save it as a patch, and recover from a failed test."]}

- Explain what the bootloader does and which boot stages and files belong to a PHYTEC board.
- Check which bootloader your release uses instead of assuming.
- Find the bootloader repository, tag and dependent firmware for your BSP release.
- Clone, configure and build U-Boot for TI AM62x and NXP i.MX 8M Plus.
- Install the boot files on an SD card and understand the risks of eMMC and flash writes.
- Make a supported change, save it as a patch, and recover from a failed test.

.. END:list:0:0:0

.. START:section:0:1
Bootloader Role and Boot Stages
===============================

.. END:section:0:1

.. START:main_content:0:1
A bootloader runs before Linux. The processor Boot ROM starts first and loads the next stage from the configured boot source. Early stages prepare memory and security firmware, and the final stage loads the kernel, device tree and overlays and starts Linux. The stages and file names differ by processor family.

.. END:main_content:0:1

.. START:table:0:1:0

.. csv-table:: Table 1. Common boot components
   :header: "Component", "Responsibility"
   :align: center
   :class: align-center

   "Boot ROM", "Code inside the processor that selects the boot source and loads the first stage. It cannot be changed."
   "SPL", "Small early U-Boot stage that initialises memory and loads the next stage."
   "TF-A (BL31)", "Trusted Firmware-A, the secure-world firmware used on 64-bit ARM boot flows."
   "OP-TEE", "Optional trusted execution environment, used on the TI build shown in this guide."
   "TI firmware", "TI-specific firmware (for example the device manager) delivered in the ti-linux-firmware repository."
   "U-Boot", "Bootloader that loads Linux and offers a command line."
   "binman", "U-Boot tool that packs the stages and firmware into the boot images."

.. END:table:0:1:0

.. START:tip:0:1:1

.. tip::

   | **Find the last stage that spoke**
   | If a board stops before Linux, the kernel may be correct. Check which boot stage printed the last message on the serial console before you change anything else.

.. END:tip:0:1:1

.. START:section:0:2
U-Boot or Barebox: Check Your Release
=====================================

.. END:section:0:2

.. START:main_content:0:2
PHYTEC boards have used different bootloaders over time. Do not assume. The release notes of your BSP name the bootloader.

.. END:main_content:0:2

.. START:table:0:2:0

.. csv-table:: Table 2. Bootloader of example PHYTEC releases
   :header: "BSP release", "U-Boot tag in the release notes", "Repository family"
   :align: center
   :class: align-center

   "i.MX 8M Mini PD23.1.0", "v2022.04_2.2.2-phy5", "u-boot-imx on git.phytec.de"
   "i.MX 8M Plus PD24.1.0", "v2024.04-2.0.0-phy7", "NXP-based U-Boot"
   "AM62Ax PD25.1.0", "v2025.07-phy4", "TI-based BSP"
   "AM62Px PD26.1.0", "v2025.01-11.02.11-phy6", "u-boot-phytec-ti on GitHub"
   "AM62x standalone guide", "v2025.07-phy2 (shown as example)", "u-boot-phytec on GitHub"

.. END:table:0:2:0

.. START:warning:0:2:1

.. warning::

   | **Barebox commands do not work on U-Boot**
   | The releases in this guide use U-Boot. Barebox appears only in some older PHYTEC BSPs. A procedure for one bootloader does not apply to the other, so read the manual of your own release before you follow any command list.

.. END:warning:0:2:1

.. START:section:0:3
Identify the Hardware
=====================

.. END:section:0:3

.. START:main_content:0:3
A bootloader image contains board-specific memory, power, storage and boot settings. Write these down before you clone anything.

.. END:main_content:0:3

.. START:table:0:3:0

.. csv-table:: Table 3. Information to record before cloning
   :header: "Item", "Why it matters"
   :align: center
   :class: align-center

   "SoM and carrier board", "Selects memory, power, storage and peripheral support."
   "Processor variant", "Selects architecture and the early firmware (for example TI HS-FS or GP silicon)."
   "Hardware revision", "Can change memory, PMIC, boot straps or routing."
   "BSP release name", "Ties U-Boot, firmware, kernel and image format together."
   "Boot medium", "SD card, eMMC, OSPI NOR, NAND, USB or network."
   "Recovery method", "Serial console, USB DFU, UART boot or a known-good SD card."

.. END:table:0:3:0

.. START:warning:0:3:1

.. warning::

   | Never flash an image from another SoM or carrier board
   | Memory, power and storage settings are part of the image. An image for a similar board can leave your board silent.

.. END:warning:0:3:1

.. START:section:0:4
PHYTEC U-Boot Repositories and Tags
===================================

.. END:section:0:4

.. START:main_content:0:4
PHYTEC publishes several U-Boot repositories. Many of PHYTEC boards are also supported in upstream U-Boot, which has its own board documents.

.. END:main_content:0:4

.. START:table:0:4:0

.. csv-table:: Table 4. U-Boot repositories seen in PHYTEC documents
   :header: "Repository", "Seen in", "Notes"
   :align: center
   :class: align-center

   "u-boot-phytec (github.com/phytec/u-boot-phytec)", "PHYTEC AM62x standalone U-Boot guide", "Upstream-based tags such as v2025.07-phy2."
   "u-boot-phytec-ti (github.com/phytec/u-boot-phytec-ti)", "AM62Px PD26.1.0 release announcement", "TI-SDK-based tag v2025.01-11.02.11-phy6. [3]"
   "u-boot-imx (git.phytec.de/u-boot-imx)", "i.MX 8M Mini PD22 and PD23 manuals", "NXP-based tags such as v2022.04_2.2.2-phy5. [4]"
   "u-boot-phytec-imx", "Later i.MX BSP manuals (as cited in earlier drafts)", "Confirm the name in your own manual."

.. END:table:0:4:0

.. START:table:0:4:1

.. csv-table:: Table 5. How to read U-Boot tags
   :header: "Tag", "Base", "Meaning"
   :align: center
   :class: align-center

   "v2025.07-phy2", "Upstream U-Boot", "U-Boot 2025.07 plus PHYTEC revision 2."
   "v2025.01-11.02.11-phy6", "TI SDK", "U-Boot 2025.01 from TI SDK release 11.02.11, PHYTEC revision 6."
   "v2022.04_2.2.2-phy5", "NXP release", "U-Boot 2022.04 from NXP release 2.2.2, PHYTEC revision 5."
   "v2024.04-2.0.0-phy7", "NXP release", "U-Boot 2024.04 from NXP release 2.0.0, PHYTEC revision 7."

.. END:table:0:4:1

.. START:tip:0:4:2

.. tip::

   | **The release notes are the source of truth**
   | Take repository, tag and dependent firmware versions from the release notes and the standalone build guide of your own BSP. Never use the newest tag of a repository by default.

.. END:tip:0:4:2

.. START:section:0:5
Bootloader Revision for Your Release
====================================

.. END:section:0:5

.. START:list:0:5:0

.. LIST_META: {"type": "bullet", "head": "", "items": ["Open the release notes. They list the U-Boot version and tag, the kernel tag and the Yocto release.", "Open the standalone U-Boot guide of the same release, usually in the section on building the BSP.", "Cross-check the recipe in the PHYTEC layer if you have the BSP source.", "Write down the full set: repository, tag, defconfig, TF-A, OP-TEE and firmware revisions, and the output file names."]}

- Open the release notes. They list the U-Boot version and tag, the kernel tag and the Yocto release.
- Open the standalone U-Boot guide of the same release, usually in the section on building the BSP.
- Cross-check the recipe in the PHYTEC layer if you have the BSP source.
- Write down the full set: repository, tag, defconfig, TF-A, OP-TEE and firmware revisions, and the output file names.

.. END:list:0:5:0

.. START:code:0:5:1

.. code-block:: console

   # from an initialised PHYTEC BSP source tree
   $ find sources -path "*recipes-bsp/u-boot*" -type f
   $ grep -RInE "SRC_URI|SRCREV|BRANCH|PV" sources | grep -i "u-boot"
   $ grep -RInE "UBOOT_CONFIG|PREFERRED_PROVIDER.*u-boot" conf sources

.. END:code:0:5:1

Recipe names and layer paths differ between releases. Adjust the search to your layer layout.

.. START:section:0:6
Clone the Sources and Prepare the Host
======================================

.. END:section:0:6

.. START:main_content:0:6
Clone each required repository at the revision of your release, and create a work branch before editing. A tag is a fixed revision, so Git reports a detached HEAD after checking it out.

.. END:main_content:0:6

.. START:code:0:6:0

.. code-block:: console

   $ git clone https://github.com/phytec/u-boot-phytec.git
   $ cd u-boot-phytec
   $ git fetch --all --tags
   $ git checkout <tag-from-your-release-notes>
   $ git describe --tags --always
   $ git switch --create work/<short-bootloader-change>

.. END:code:0:6:0

.. START:list:0:6:1

.. LIST_META: {"type": "bullet", "head": "", "items": ["Install the host packages listed in the Host Setup section of the PHYTEC standalone guide for your release.", "Use the cross compilers named in the guide: arm-linux-gnueabihf- for the TI R5 stage and aarch64-linux-gnu- for 64-bit stages.", "If the release uses several repositories, clone each one at the documented revision. Do not combine a U-Boot tag with unrelated TF-A, OP-TEE or firmware versions."]}

- Install the host packages listed in the Host Setup section of the PHYTEC standalone guide for your release.
- Use the cross compilers named in the guide: arm-linux-gnueabihf- for the TI R5 stage and aarch64-linux-gnu- for 64-bit stages.
- If the release uses several repositories, clone each one at the documented revision. Do not combine a U-Boot tag with unrelated TF-A, OP-TEE or firmware versions.

.. END:list:0:6:1

.. START:warning:0:6:2

.. warning::

   | **Do not copy tags to another board**
   | The tags shown in this guide belong to the shown releases. Another SoM or release has its own tags and defconfigs.

.. END:warning:0:6:2

.. START:section:0:7
U-Boot Configuration
====================

.. END:section:0:7

.. START:main_content:0:7
U-Boot uses Kconfig in the same way as the Linux kernel. A defconfig selects board support, commands, drivers and memory layout, and the final settings are written to .config. Use a separate output directory with O= to keep builds apart.

.. END:main_content:0:7

.. START:table:0:7:0

.. csv-table:: Table 6. U-Boot configuration terms
   :header: "Term", "Meaning"
   :align: center
   :class: align-center

   "defconfig", "Starting configuration for a board or a boot stage, for example phycore_am62x_r5_defconfig."
   ".config", "Final settings of one build, stored in the output directory."
   "Config fragment", "A .config file that adds options on top of a defconfig, for example am62x_r5_usbdfu.config for a USB DFU build of the R5 stage. [2]"
   "O=<directory>", "Output directory. TI builds use one for the R5 stage and one for the A53 stage."
   "BINMAN_INDIRS", "Directory with firmware files that binman packs into the boot images, for example ti-linux-firmware."

.. END:table:0:7:0

.. START:code:0:7:1

.. code-block:: console

   $ make list_defconfigs | grep -i -E "phy|am62|imx8"
   $ make O=r5 menuconfig        # change options of a configured build
   $ make O=r5 savedefconfig     # writes a minimal defconfig into the output folder

.. END:code:0:7:1

Defconfig names differ between repositories. Upstream U-Boot names the i.MX 8M Plus board phycore-imx8mp_defconfig, with a hyphen, while the TI board uses underscores. Always take the name from your guide or from list_defconfigs.

.. START:section:0:8
Build for TI AM62x
==================

.. END:section:0:8

.. START:main_content:0:8
A TI AM62x boot image has three parts built from four sources: U-Boot, TF-A, OP-TEE and TI firmware. The sequence below follows PHYTEC and U-Boot documentation for the phyCORE-AM62x. Replace paths and tags with those of your release.

.. END:main_content:0:8

.. START:table:0:8:0

.. csv-table:: Table 7. Variables of the AM62x build
   :header: "Variable", "Value or purpose"
   :align: center
   :class: align-center

   "UBOOT_CFG_CORTEXR", "phycore_am62x_r5_defconfig (R5 boot processor stage)"
   "UBOOT_CFG_CORTEXA", "phycore_am62x_a53_defconfig (A53 main stage)"
   "TFA_BOARD", "lite (TF-A platform name for this SoM)"
   "OPTEE_PLATFORM", "k3-am62x"
   "BINMAN_INDIRS", "Path to the ti-linux-firmware repository"
   "BL31, TEE", "Paths to bl31.bin from TF-A and tee-pager_v2.bin from OP-TEE"

.. END:table:0:8:0

.. START:code:0:8:1

.. code-block:: console

   # 1. Trusted Firmware-A
   cd ~/trusted-firmware-a
   export ARCH=aarch64
   export CROSS_COMPILE=aarch64-linux-gnu-
   make PLAT=k3 SPD=opteed TARGET_BOARD=lite
    
   # 2. OP-TEE: build it as described in the guide of your release
    
   # 3. R5 stage of U-Boot
   cd ~/u-boot-phytec
   export ARCH=arm
   export CROSS_COMPILE=arm-linux-gnueabihf-
   export BINMAN_INDIRS=~/ti-linux-firmware
   make O=r5 phycore_am62x_r5_defconfig
   make O=r5 -j"$(nproc)"          # result: r5/tiboot3.bin
    
   # 4. A53 stage of U-Boot
   export ARCH=arm64
   export CROSS_COMPILE=aarch64-linux-gnu-
   export TEE=~/optee_os/out/arm-plat-k3/core/tee-pager_v2.bin
   export BL31=~/trusted-firmware-a/build/k3/lite/release/bl31.bin
   make O=a phycore_am62x_a53_defconfig
   make O=a -j"$(nproc)"           # result: a/tispl.bin and a/u-boot.img

.. END:code:0:8:1

.. START:note:0:8:2

.. note::

   | The OP-TEE file name differs between guides
   | The PHYTEC standalone guide uses tee-pager_v2.bin and the U-Boot board document uses tee-raw.bin. The correct file depends on the OP-TEE version and the guide of your release.

.. END:note:0:8:2

.. START:section:0:9
Build for NXP i.MX 8M Plus
==========================

.. END:section:0:9

.. START:main_content:0:9
| On i.MX 8M, the final bootable bootloader image combines the SPL, U-Boot, TF-A and the DDR firmware. PHYTEC calls this image imx-boot, built by imx-mkimage in its Yocto releases. The file u-boot.bin alone is not the final bootable image. [6]
| The upstream U-Boot board document for the phyCORE-i.MX 8M Plus shows the standalone sequence. Compare it with the manual of your PHYTEC release, because a release may package the image differently.

.. END:main_content:0:9

.. START:code:0:9:0

.. code-block:: console

   # 1. TF-A for i.MX 8M Plus
   $ export CROSS_COMPILE=aarch64-linux-gnu-
   $ make PLAT=imx8mp bl31
    
   # 2. NXP DDR firmware: install the firmware-imx package of your release
   # 3. copy bl31.bin and the lpddr4*.bin files into the U-Boot directory
    
   # 4. build the bootable image
   $ make phycore-imx8mp_defconfig
   $ make flash.bin

.. END:code:0:9:0

The result is flash.bin, which contains the SPL and a FIT image with U-Boot and TF-A.

Build with Yocto
~~~~~~~~~~~~~~~~

For a production image the Yocto build is normally safer, because it applies the patches, firmware and packaging rules of the release together. The provider and variable names depend on the release.

.. START:code:0:9:3

.. code-block:: console

   # inside an initialised PHYTEC build environment
   $ bitbake -e virtual/bootloader | grep -E "^(SRC_URI|SRCREV|PV)="
   $ bitbake -c compile -f virtual/bootloader
   $ bitbake <image-name>

.. END:code:0:9:3

.. START:table:0:9:4

.. csv-table:: Table 8. Standalone build compared with Yocto
   :header: "Workflow", "Use it when", "Main caution"
   :align: center
   :class: align-center

   "Standalone", "You need a fast loop or want to inspect one component.", "You must match all firmware and packaging by hand."
   "Yocto", "You need a reproducible, deployable image.", "Use the exact machine, layers and release metadata."

.. END:table:0:9:4

.. START:section:0:10
Install the Boot Files Safely
=============================

.. END:section:0:10

.. START:main_content:0:10
A built bootloader does nothing until it is on the boot medium. Start with an SD card, because it can be replaced without touching the factory image.

.. END:main_content:0:10

.. START:table:0:10:0

.. csv-table:: Table 9. Install methods used in the documentation
   :header: "Platform", "SD card", "Other media"
   :align: center
   :class: align-center

   "TI AM62x", "Copy tiboot3.bin, tispl.bin and u-boot.img to the FAT partition of the SD card. [2]", "OSPI NOR with mtd write commands from U-Boot; UART boot with sb for recovery. [2]"
   "NXP i.MX 8M Plus", "Write flash.bin to the SD card at the offset of 32 KiB. [2]", "eMMC and other media use other offsets. Follow the manual of your release."

.. END:table:0:10:0

.. START:code:0:10:1

.. code-block:: console

   # NXP example from the upstream board document: replace <sdcard-device> after checking it
   $ sudo dd if=flash.bin of=<sdcard-device> bs=1024 seek=32 conv=sync
    
   # TI example: a FAT partition on the SD card, mounted as <mountpoint>
   $ cp r5/tiboot3.bin a/tispl.bin a/u-boot.img <mountpoint>/

.. END:code:0:10:1

.. START:danger:0:10:2

.. danger::

   | **Check the target device before every write**
   | Writing a bootloader to the wrong device can destroy the boot path or user data. List the devices with lsblk, check the size and name of the SD card twice, and keep a known-good card and a serial console ready.

.. END:danger:0:10:2

.. START:section:0:11
Boot Modes and Recovery
=======================

.. END:section:0:11

.. START:main_content:0:11
PHYTEC boards select the boot source with switches or jumpers. Change them with the power off. The next table lists the settings of the phyCORE-AM62x development kit from the U-Boot board document. Confirm them in the manual of your kit revision.

.. END:main_content:0:11

.. START:table:0:11:0

.. csv-table:: Table 10. Boot switch settings of the phyCORE-AM62x kit
   :header: "Boot mode", "SW5 (positions 1 to 8)", "SW6 (positions 1 to 8)"
   :align: center
   :class: align-center

   "SD card", "11000010", "01000000"
   "eMMC", "11010010", "00000000"
   "OSPI NOR", "11010000", "10000000"
   "UART", "11011100", "00000000"
   "USB DFU", "11001010", "00100000"

.. END:table:0:11:0

.. START:table:0:11:1

.. csv-table:: Table 11. Recovery options
   :header: "Option", "Use"
   :align: center
   :class: align-center

   "Known-good SD card", "Best first test and recovery medium, if the board supports SD boot."
   "UART boot", "Loads tiboot3.bin, tispl.bin and u-boot.img over the serial port with a tool such as sb. Useful when flash content is damaged. [2]"
   "USB DFU", "Loads boot files over USB without normal boot media. A special R5 build with a DFU fragment may be needed. [2]"
   "Serial console", "Shows the output of every stage. Keep it connected during all tests."

.. END:table:0:11:1

.. START:tip:0:11:2

.. tip::

   | **Change one thing at a time**
   | Test a new bootloader from the SD card first. Move it to eMMC or flash only after it boots and after you have tested the recovery path.

.. END:tip:0:11:2

.. START:section:0:12
Environment Storage
===================

.. END:section:0:12

.. START:main_content:0:12
U-Boot keeps variables such as bootcmd and bootargs in its environment. The place where it is stored depends on the board and on the boot device. For the phyCORE-AM62x the U-Boot document describes this behaviour:

.. END:main_content:0:12

.. START:table:0:12:0

.. csv-table:: Table 12. Environment storage by boot device (AM62x)
   :header: "Boot device", "Where the environment lives"
   :align: center
   :class: align-center

   "eMMC or SD card", "A file on the FAT partition or a fixed offset, if configured."
   "SPI flash", "In the SPI flash, if enabled."
   "Other boot devices", "Not stored persistently."

.. END:table:0:12:0

.. START:warning:0:12:1

.. warning::

   | **Do not save the environment until you understand the storage**
   | Avoid saveenv, env save, mmc write and sf write until you know where the data goes and have a recovery method. Details of the prompt commands are in BSP-08.

.. END:warning:0:12:1

.. START:section:0:13
Supported Customizations and Limits
===================================

.. END:section:0:13

.. START:table:0:13:0

.. csv-table:: Table 13. Typical bootloader changes
   :header: "Change", "Typical place"
   :align: center
   :class: align-center

   "Default boot target or boot order", "U-Boot environment, boot script or standard boot configuration"
   "Kernel command-line arguments", "Environment variable or boot script"
   "Add a diagnostic command", "Kconfig option of the command, then rebuild"
   "Enable a storage or network feature", "Board configuration, driver option and device-tree entry (BSP-07)"
   "USB DFU or special boot builds", "Config fragment such as am62x_r5_usbdfu.config"
   "Change boot image packaging", "Binman description and the firmware directory"

.. END:table:0:13:0

.. START:table:0:13:1

.. csv-table:: Table 14. Changes that need extra care
   :header: "Change", "Why"
   :align: center
   :class: align-center

   "DDR timing, PMIC or power settings", "Board specific, applied before any recovery code runs. Change only with PHYTEC guidance and a hardware recovery path."
   "Boot ROM behaviour", "Fixed in silicon. Only the boot source and the image layout can be influenced."
   "TI firmware and TF-A versions", "Must match the U-Boot release and the SoC variant. Do not mix versions."
   "Secure boot or signed images", "Needs the signing and key process of your BSP manual. A wrong step can lock the board."

.. END:table:0:13:1

Make a Change You Can Repeat
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. START:code:0:13:3

.. code-block:: console

   $ git status --short --branch
   $ git diff --check
   $ git add <changed-files>
   $ git commit -m "bootloader: describe the board change"
   $ git format-patch -1 HEAD

.. END:code:0:13:3

.. START:table:0:13:4

.. csv-table:: Table 15. Record of a bootloader change
   :header: "Record", "Evidence"
   :align: center
   :class: align-center

   "Source", "Repository, branch, tag and commit"
   "Configuration", "Defconfig, .config and fragments"
   "Dependencies", "TF-A, OP-TEE, TI firmware and compiler versions"
   "Build", "Commands and output directories"
   "Artifacts", "File names, sizes and checksums"
   "Deployment", "Boot medium, offset or partition and recovery method"
   "Test", "Serial log, U-Boot version, boot result and rollback result"

.. END:table:0:13:4

.. START:section:0:14
Common Bootloader Problems
==========================

.. END:section:0:14

.. START:table:0:14:0

.. csv-table:: Table 16. Symptoms and first checks
   :header: "Symptom", "Likely cause", "First check"
   :align: center
   :class: align-center

   "No serial output", "Power, boot switches, cable or damaged early image", "Switch settings, console settings, known-good SD card"
   "ROM or SPL message only", "Wrong early firmware, DDR or TF-A", "Matching revisions and the defconfig"
   "U-Boot prompt but no Linux", "Wrong boot command, kernel, DTB or storage", "printenv, storage listing, file names"
   "Works from SD but not eMMC", "Wrong offset, partition or boot mode", "Boot chapter and image layout of your manual"
   "Build fails in binman", "Missing firmware or wrong path", "BINMAN_INDIRS, BL31, TEE"
   "Board silent after a flash", "Wrong image or overwritten recovery path", "Boot from SD or UART and restore the known-good image"
   "Environment change disappears", "No persistent storage for this boot device", "Boot device and environment configuration"

.. END:table:0:14:0

.. START:section:0:15
Safe Bootloader Workflow
========================

.. END:section:0:15

.. START:list:0:15:0

.. LIST_META: {"type": "auto", "head": "", "items": ["Identify the board, carrier revision and BSP release.", "Confirm the bootloader, boot medium and recovery method.", "Find repository, tag, defconfig and firmware versions in the release notes.", "Clone every required repository at the matching revision.", "Build the unmodified bootloader and compare the artifacts with the documented ones.", "Create a work branch and make one supported change.", "Build again and record file names and checksums.", "Test from an SD card or recovery medium first, with the serial console connected.", "Commit the change and keep the old boot image.", "Carry the patch into the Yocto build when the test passes."]}

1. Identify the board, carrier revision and BSP release.
2. Confirm the bootloader, boot medium and recovery method.
3. Find repository, tag, defconfig and firmware versions in the release notes.
4. Clone every required repository at the matching revision.
5. Build the unmodified bootloader and compare the artifacts with the documented ones.
6. Create a work branch and make one supported change.
7. Build again and record file names and checksums.
8. Test from an SD card or recovery medium first, with the serial console connected.
9. Commit the change and keep the old boot image.
10. Carry the patch into the Yocto build when the test passes.

.. END:list:0:15:0

.. START:section:0:16
Bootloader Checklist
====================

.. END:section:0:16

.. START:list:0:16:0

.. LIST_META: {"type": "bullet", "head": "", "items": ["Board, carrier revision, BSP release and boot medium recorded.", "Bootloader type confirmed from the release notes.", "Repository, tag and firmware revisions taken from the release.", "Work branch created and tree clean.", "Correct defconfigs and cross compilers used.", "Unmodified build produces the documented files.", "Install target checked twice before every write.", "Serial log captured and Linux starts.", "Patch saved with checksums and a rollback path."]}

- Board, carrier revision, BSP release and boot medium recorded.
- Bootloader type confirmed from the release notes.
- Repository, tag and firmware revisions taken from the release.
- Work branch created and tree clean.
- Correct defconfigs and cross compilers used.
- Unmodified build produces the documented files.
- Install target checked twice before every write.
- Serial log captured and Linux starts.
- Patch saved with checksums and a rollback path.

.. END:list:0:16:0

.. START:section:0:17
Glossary
========

.. END:section:0:17

.. START:table:0:17:0

.. csv-table:: Table 17. Terms used in this guide
   :header: "Term", "Meaning"
   :align: center
   :class: align-center

   "SPL", "Secondary Program Loader, the small first U-Boot stage."
   "TF-A", "Trusted Firmware-A, secure-world firmware for 64-bit ARM."
   "OP-TEE", "Open-source trusted execution environment."
   "R5 / A53", "The TI real-time boot core and the main 64-bit application cores."
   "binman", "U-Boot tool that builds the boot images from several inputs."
   "imx-boot / flash.bin", "Final bootable bootloader image of i.MX 8M boards."
   "DFU", "Device Firmware Upgrade, a USB loading method."
   "Standard boot", "U-Boot mechanism for finding and starting an operating system."

.. END:table:0:17:0

.. START:section:0:18
References
==========

.. END:section:0:18

.. START:main_content:0:18
[1] PHYTEC phyCORE-AM62x: `Standalone U-Boot Development <https://docs.phytec.com/projects/yocto-phycore-am62x/en/latest/developingwithyocto/build-u-boot.html>`_ — Repository, TF-A, R5 and A53 build steps and output files.

[2] U-Boot documentation: `phyCORE-AM62x and phyCORE-i.MX 8M Plus <https://docs.u-boot.org/en/latest/board/phytec/phycore-am62x.html>`_ — Boot artifacts, boot modes, environment storage and installation steps.

[3] PHYTEC phyCORE-AM62Px: `PD26.1.0 Software Release <https://blog.phytec.com/phycore-am62px-software-release-bsp-yocto-ampliphy-am62px-pd26.1.0>`_ — U-Boot tag, repository and Ampliphy-Boot.

[4] PHYTEC BSP documentation: `i.MX 8M Mini BSP Source <https://github.com/phytec/doc-bsp-yocto/blob/main/source/bsp/imx8/imx8mm/head.rst>`_ — u-boot-imx location and imx-boot image description.

[5] PHYTEC phyCORE-AM62x BSP: `Building the BSP <https://docs.phytec.com/projects/yocto-phycore-am62x/en/latest/developingwithyocto/buildBSP.html>`_ — Host setup for BSP and standalone builds.

[6] PHYTEC BSP documentation: `i.MX 8M Mini BSP Source <https://github.com/phytec/doc-bsp-yocto/blob/main/source/bsp/imx8/imx8mm/head.rst>`_ — Explains why u-boot.bin is not the final bootable image and describes imx-boot.

[7] U-Boot patch: `i.MX8MP Single Bootable Binary <https://git01.mediatek.com/plugins/gitiles/filogic/uboot/+/1d05e8ed86c206ced9c8d003b71ecc5493ef85ec%5E%21>`_ — flash.bin for phyCORE-i.MX8MP and the SD offset.

[8] PHYTEC phyCORE-AM62Px: `PD26.1.0 Release Notes <https://docs.phytec.com/projects/yocto-phycore-am62px/en/latest/releasenotes/pd26.1.0.html>`_ — Ampliphy-Boot based on U-Boot standard boot.

[9] U-Boot: `Official Documentation <https://docs.u-boot.org/en/latest/>`_ — Kconfig, build system and commands.

.. END:main_content:0:18
