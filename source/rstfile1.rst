.. START:section:0:0
What is cross-compilation?
==========================

.. END:section:0:0

.. START:main_content:0:0
Cross-compilation means compiling a program on a powerful host computer so that it runs on a different target computer. The host may be an x86-64 Linux workstation, while a PHYTEC board may use an ARM Cortex-A processor. The compiler runs on the host, but the generated program runs on the board.

.. END:main_content:0:0

|

.. START:table:0:0:2

.. csv-table:: Table 1 — Native versus cross compilation
   :header: "Method", "Best use"
   :align: center
   :class: align-center

   "Native", "Small experiments compiled directly on the running board."
   "Cross", "Normal application development using a faster host."
   "Yocto recipe", "Production application built reproducibly into an image."

.. END:table:0:0:2

.. START:tip:0:0:3

.. tip::

   PHYTEC application-development guidance describes native and cross-platform development. Cross-compilation usually gives better editor, CPU and storage resources than compiling on a small target.

.. END:tip:0:0:3

.. START:section:0:1
Understand the SDK
==================

.. END:section:0:1

.. START:main_content:0:1
A PHYTEC SDK is more than a compiler. It normally contains a cross-compiler, linker tools, target headers, target libraries, a sysroot and an environment setup script. The sysroot tells the compiler which target files to use.

.. END:main_content:0:1

.. START:table:0:1:0

.. csv-table:: Table 2 — Important SDK variables
   :header: "Variable", "Purpose"
   :align: center
   :class: align-center

   "``CC``", "C compiler command for the target."
   "``CXX``", "C++ compiler command for the target."
   "``AR``", "Target archive tool."
   "``LD``", "Target linker."
   "``SDKTARGETSYSROOT``", "Target headers and libraries used during compilation."
   "``PATH``", "Allows the SDK cross-tools to be found."
   "``PKG_CONFIG_SYSROOT_DIR``", "Rewrites pkg-config paths for the target sysroot."

.. END:table:0:1:0

.. START:warning:0:1:1

.. warning::

   Do not mix `CC`, `CXX` and `SDKTARGETSYSROOT` from different BSP releases. A binary may compile and still fail on the board because its libraries do not match.

.. END:warning:0:1:1

.. START:section:0:2
Choose the matching BSP and SDK
===============================

.. END:section:0:2

.. START:main_content:0:2
elect the SDK built for the same PHYTEC board family, processor architecture and BSP release as the target image. An AM62x SDK is not interchangeable with an i.MX 8M Plus SDK. A newer SDK may also contain incompatible library versions.

.. END:main_content:0:2

.. START:table:0:2:0

.. csv-table:: Table 3 — SDK matching checklist
   :header: "Check", "Expected result"
   :align: center
   :class: align-center

   "Board and SoM", "SDK and image target the same PHYTEC platform."
   "Processor architecture", "For example, AArch64 or 32-bit ARM, as appropriate."
   "BSP release", "SDK release matches the image on the board."
   "C library", "SDK and target use the same libc family."
   "Sysroot", "Contains headers and libraries required by the application."
   "Compiler ABI", "Floating-point and ABI choices match the target."

.. END:table:0:2:0

.. START:code:0:2:1

.. code-block:: console

   # On the target, record identity before selecting an SDK
   $ cat /proc/device-tree/model 2>/dev/null; echo
   $ uname -m
   $ cat /etc/os-release
   $ ldd --version 2>/dev/null | head -n 1

.. END:code:0:2:1

.. START:section:0:3
Install the SDK on a Linux host
===============================

.. END:section:0:3

.. START:main_content:0:3
PHYTEC documentation commonly provides an SDK installer as part of the BSP workflow. The exact filename, installation directory and command depend on the release. Use the release-specific PHYTEC guide and do not assume that one installer name applies to every BSP.

.. END:main_content:0:3

.. START:code:0:3:0

.. code-block:: console

   # Example pattern; replace the filename with the SDK from your BSP
   $ chmod +x <phytec-sdk-installer>.sh
   $ ./<phytec-sdk-installer>.sh
   
   # Choose a dedicated installation directory, for example
   # /opt/amplify/<bsp-release>/

.. END:code:0:3:0

.. START:list:0:3:1

.. LIST_META: {"type": "bullet", "head": "", "items": ["Use a Linux host or an approved container/virtual machine.", "Install the SDK in a path without spaces when possible.", "Record the SDK version and installation path.", "Do not install an NXP SDK over a TI SDK directory or vice versa."]}

- Use a Linux host or an approved container/virtual machine.
- Install the SDK in a path without spaces when possible.
- Record the SDK version and installation path.
- Do not install an NXP SDK over a TI SDK directory or vice versa.

.. END:list:0:3:1

.. START:important:0:3:2

.. important::

   If the target image was built by a different BSP release, build or obtain the matching SDK instead of trying to repair a mismatched sysroot manually.

.. END:important:0:3:2

.. START:section:0:4
Source the SDK environment
==========================

.. END:section:0:4

.. START:main_content:0:4
The SDK setup script configures the current shell. It normally updates `PATH`, `CC`, `CXX`, `SDKTARGETSYSROOT` and related variables. Source it again in every new terminal where you compile.

.. END:main_content:0:4

.. START:code:0:4:0

.. code-block:: console

   # Example; use the actual path installed on your host
   $ source /opt/amplify/<bsp-release>/environment-setup-<target-triplet>
   
   # Confirm the environment
   $ echo "$CC"
   $ echo "$CXX"
   $ echo "$SDKTARGETSYSROOT"
   $ echo "$PKG_CONFIG_SYSROOT_DIR"
   $ which "${CC%% *}" 2>/dev/null

.. END:code:0:4:0

.. START:table:0:4:1

.. csv-table:: Table 4 — What to check after sourcing
   :header: "Check", "Good result"
   :align: center
   :class: align-center

   "``CC``", "Contains a target-prefixed compiler."
   "``CXX``", "Contains the matching target C++ compiler."
   "``SDKTARGETSYSROOT``", "Points inside the selected SDK."
   "``PATH``", "Target tool directory appears before unrelated tools."
   "Compiler version", "Matches the selected BSP SDK."

.. END:table:0:4:1

.. START:warning:0:4:2

.. warning::

   Use `source` in the same shell where you run `make`. Running the setup script in one terminal does not configure a different terminal.

.. END:warning:0:4:2

.. START:section:0:5
Inspect the target triplet
==========================

.. END:section:0:5

.. START:main_content:0:5
The target triplet is part of the compiler name and identifies the target ABI. Do not guess it from the board marketing name. Read it from `CC` or the SDK directory.

.. END:main_content:0:5

.. START:code:0:5:0

.. code-block:: console

   # Example inspection commands
   $ echo "$CC"
   $ echo "$CXX"
   $ ${CC} --version
   $ ${CC} -dumpmachine
   $ ${CXX} -dumpmachine

.. END:code:0:5:0

.. START:table:0:5:1

.. csv-table:: Table 5 — Common architecture clues
   :header: "Command result", "Meaning"
   :align: center
   :class: align-center

   "``aarch64-...``", "64-bit ARM target, common for many modern PHYTEC SoMs."
   "``arm-...`` or ``arm-linux-...``", "32-bit ARM target; exact ABI must be checked."
   "Host compiler name", "Not proof that the target compiler is active."

.. END:table:0:5:1

.. START:tip:0:5:2

.. tip::

   The compiler name, `uname -m` on the board and the SDK documentation should tell one consistent architecture story.

.. END:tip:0:5:2

.. START:section:0:6
Sample application build
========================

.. END:section:0:6

Write a small C application
---------------------------

Start with a program that needs only the standard C library. This separates toolchain setup from application-specific dependencies.

.. START:code:0:6:1

.. code-block:: bash

   /* hello.c */
   #include <stdio.h>
   
   int main(void)
   {
       puts("Hello from a PHYTEC target");
       return 0;
   }

.. END:code:0:6:1

.. START:code:0:6:2

.. code-block:: console

   # Build with the SDK environment already sourced
   $ "$CC" hello.c -O2 -Wall -Wextra -o hello
   
   # Inspect the result on the host
   $ file hello
   $ readelf -h hello | head -n 15

.. END:code:0:6:2

.. START:table:0:6:3

.. csv-table:: Table 6 — Expected C build evidence
   :header: "Evidence", "What it tells you"
   :align: center
   :class: align-center

   "No compiler error", "The source and basic SDK environment work."
   "``file`` reports ARM/AArch64", "The output is not a host executable."
   "``readelf`` shows target machine", "The ELF header targets the board architecture."
   "No host-only library path", "The application is using the intended target sysroot."

.. END:table:0:6:3

Write a small C++ application
-----------------------------

.. START:code:0:6:5

.. code-block:: bash

   // hello.cpp
   #include <iostream>
   
   int main()
   {
       std::cout << "Hello C++ from a PHYTEC target" << std::endl;
       return 0;
   }

.. END:code:0:6:5

.. START:code:0:6:6

.. code-block:: console

   # Build with the matching C++ compiler
   $ "$CXX" hello.cpp -O2 -Wall -Wextra -std=c++17 -o hello-cpp
   
   # Check dynamic library requirements
   $ readelf -d hello-cpp | grep NEEDED

.. END:code:0:6:6

.. START:important:0:6:7

.. important::

   The C++ runtime library must exist on the target image. If it is not included, add the required package through the BSP/Yocto image rather than copying random libraries from another board.

.. END:important:0:6:7

Use a Makefile
--------------

Makefile makes the compiler and flags visible and repeatable. It should use the environment's `CC` and `CXX` rather than hard-coding a host compiler.

.. START:code:0:6:9

.. code-block:: bash

   CC ?= cc
   CXX ?= c++
   CFLAGS ?= -O2 -Wall -Wextra
   CXXFLAGS ?= -O2 -Wall -Wextra -std=c++17
   
   all: hello hello-cpp
   
   hello: hello.c
   	$(CC) $(CFLAGS) $< -o $@
   
   hello-cpp: hello.cpp
   	$(CXX) $(CXXFLAGS) $< -o $@
   
   clean:
   	rm -f hello hello-cpp

.. END:code:0:6:9

.. START:code:0:6:10

.. code-block:: console

   # Build after sourcing the PHYTEC SDK
   $ make clean
   $ make
   $ file hello hello-cpp

.. END:code:0:6:10

.. START:tip:0:6:11

.. tip::

   Use `?=` in a starter Makefile so the PHYTEC SDK environment can provide `CC` and `CXX`. A plain `CC = gcc` can silently build for the host.

.. END:tip:0:6:11

Use target libraries with the sysroot
-------------------------------------

When an application uses a target library, the compiler needs the target header and library from `SDKTARGETSYSROOT`. The library also needs to be present in the target root filesystem at runtime.

.. START:code:0:6:13

.. code-block:: console

   # Inspect a sysroot
   $ echo "$SDKTARGETSYSROOT"
   $ find "$SDKTARGETSYSROOT/usr/include" -maxdepth 1 -type d | head
   $ find "$SDKTARGETSYSROOT/usr/lib" -maxdepth 1 -type f | head
   
   # Compile a program with an explicit sysroot when needed
   $ "$CC" --sysroot="$SDKTARGETSYSROOT" app.c -o app

.. END:code:0:6:13

.. START:table:0:6:14

.. csv-table:: Table 7 — Compile-time and run-time requirements
   :header: "Stage", "Requirement"
   :align: center
   :class: align-center

   "Compile", "Target header exists in the SDK sysroot."
   "Link", "Target library and linker metadata exist in the sysroot."
   "Deploy", "Binary is copied to the board."
   "Run", "The target has the required shared library and loader."
   "Package", "The dependency is included in the production image."

.. END:table:0:6:14

.. START:warning:0:6:15

.. warning::

   The SDK sysroot is not automatically copied to the board. It is a host-side development tree.

.. END:warning:0:6:15

Use pkg-config safely
---------------------

Libraries such as GTK, GStreamer or libcamera may provide `.pc` files. Plain host `pkg-config` can accidentally return host paths. The PHYTEC SDK environment normally sets variables to help it use the target sysroot.

.. START:code:0:6:17

.. code-block:: bash

   # Inspect the active pkg-config environment
   $ echo "$PKG_CONFIG_SYSROOT_DIR"
   $ echo "$PKG_CONFIG_PATH"
   $ pkg-config --cflags --libs <target-library>
   
   # Confirm a returned include/library path belongs to the target sysroot
   $ pkg-config --cflags --libs <target-library> | tr ' ' '\n'

.. END:code:0:6:17

.. START:important:0:6:18

.. important::

   If `pkg-config` returns `/usr/include` or `/usr/lib` from the host, stop and fix the SDK environment before compiling.

.. END:important:0:6:18

Link and inspect shared libraries
---------------------------------

.. START:code:0:6:20

.. code-block:: bash

   # Inspect dynamic dependencies on the host
   $ readelf -d ./app | grep NEEDED
   
   # On the target, inspect the loader and libraries
   $ file /tmp/app
   $ ldd /tmp/app 2>/dev/null
   $ cat /etc/ld.so.conf 2>/dev/nu

.. END:code:0:6:20

.. START:table:0:6:21

.. csv-table:: Table 8 — Typical linker errors
   :header: "Error", "Likely cause"
   :align: center
   :class: align-center

   "``cannot find -l...``", "Library or development files are missing from the sysroot."
   "``undefined reference``", "Missing library, wrong order or incompatible API."
   "``No such file or directory`` at run time", "Missing loader or shared library on the target."
   "``GLIBC_... not found``", "Binary was built against a newer or different libc."

.. END:table:0:6:21

Deploy the application
----------------------

Copy the target binary to a writable test directory such as `/tmp`, `/opt` or a product-specific data directory. Use serial storage, SSH, a network file service or the PHYTEC deployment method documented for the board.

.. START:code:0:6:23

.. code-block:: console

   # Copy to a running target over SSH
   $ scp ./hello <user>@<board-ip>:/tmp/
   $ scp ./hello-cpp <user>@<board-ip>:/tmp/
   
   # Log in and inspect
   $ ssh <user>@<board-ip>
   $ uname -m
   $ file /tmp/hello
   $ chmod +x /tmp/hello
   $ /tmp/hello

.. END:code:0:6:23

.. START:table:0:6:24

.. csv-table:: Table 9 — Deployment locations
   :header: "Location", "Use"
   :align: center
   :class: align-center

   "``/tmp``", "Short test; commonly cleared at reboot."
   "``/home/<user>``", "User application development."
   "``/opt``", "Longer-lived manual experiment when writable."
   "``/data``", "Product data area when provided by the image."
   "Yocto package", "Repeatable production installation."

.. END:table:0:6:24

.. START:warning:0:6:25

.. warning::

   A binary copied manually is not part of the BSP image. Record the test and later create a Yocto recipe for production.

.. END:warning:0:6:25

Run and verify on the board
---------------------------

.. START:code:0:6:27

.. code-block:: console

   # Target-side checks
   $ id
   $ uname -m
   $ file /tmp/hello
   $ /tmp/hello
   
   # Check application exit status
   $ echo $?
   
   # If the program uses a device, inspect permissions and nodes
   $ ls -l /dev/<device> 2>/dev/null
   $ dmesg | tail -n 50

.. END:code:0:6:27

.. START:table:0:6:28

.. csv-table:: Table 10 — Cross-compiled application verification
   :header: "Check", "Expected result"
   :align: center
   :class: align-center

   "Architecture", "Binary architecture matches ``uname -m``."
   "Execute permission", "The file has an executable mode."
   "Loader", "Target can find the ELF interpreter."
   "Libraries", "All dynamic dependencies are installed."
   "Permissions", "User can access required files and devices."
   "Hardware", "Application result matches the board test."

.. END:table:0:6:28

CMake and other build systems
-----------------------------

CMake, Meson and Autotools need a cross-compilation configuration. Do not let them discover host headers and libraries by accident. Use the PHYTEC SDK environment and the build-system toolchain file or documented cross file.

.. START:code:0:6:30

.. code-block:: console

   # A simple CMake invocation; use the SDK-provided toolchain file when supplied
   $ cmake -S . -B build   -DCMAKE_C_COMPILER="$CC"   -DCMAKE_CXX_COMPILER="$CXX"   -DCMAKE_SYSROOT="$SDKTARGETSYSROOT"
   $ cmake --build build
   $ file build/<application>

.. END:code:0:6:30

.. START:important:0:6:31

.. important::

   For a real project, prefer the toolchain file supplied by the SDK or project documentation. Passing only a compiler path may not configure all target tools and search paths.

.. END:important:0:6:31

.. START:section:0:7
NXP and TI SDK differences
==========================

.. END:section:0:7

.. START:main_content:0:7
.. csv-table:: Table 11 — Platform-specific points
   :header: "Platform", "Check"
   :widths: 25, 75
   :align: center

   "NXP PHYTEC", "Confirm the i.MX architecture, BSP release, libc and matching SDK installer."
   "TI PHYTEC", "Confirm AM62x, AM64x, AM68x or exact K3 target and any SDK-specific runtime packages."
   "Both", "Use the target's image and the same release's sysroot."
   "Both", "Do not use a vendor MCU SDK as a replacement for the PHYTEC Linux SDK."

.. END:main_content:0:7

The application source can often be portable between NXP and TI, but the toolchain, sysroot, device nodes, hardware APIs and available packages may differ.

.. START:section:0:8
Troubleshoot common failures
============================

.. END:section:0:8

.. START:table:0:8:0

.. csv-table:: Table 12 — Cross-compilation troubleshooting
   :header: "Symptom", "Likely cause", "First action"
   :align: center
   :class: align-center

   "``command not found`` for target compiler", "SDK environment was not sourced.", "Source the exact environment-setup script."
   "Host binary is produced", "Makefile uses ``gcc`` or ``g++`` directly.", "Use environment ``CC``/``CXX`` and inspect with ``file``."
   "Wrong architecture", "SDK does not match the board.", "Compare ``-dumpmachine`` with target ``uname -m``."
   "Header not found", "Package development files are absent from sysroot.", "Install the matching SDK package or add it to the image."
   "Library cannot be found", "Wrong sysroot or pkg-config host paths.", "Check ``SDKTARGETSYSROOT`` and pkg-config output."
   "Runs on host but not target", "Host libraries or ABI were used.", "Inspect ``file``, ``readelf -d`` and target ``ldd``."
   "Permission denied", "File is not executable or filesystem is mounted noexec.", "Run ``chmod +x`` and inspect mounts."
   "Cannot access ``/dev`` device", "User permission or Linux driver/device tree issue.", "Check device node, groups and kernel logs."
   "C++ program fails at startup", "Missing libstdc++ or incompatible runtime.", "Add the matching runtime package to the image."

.. END:table:0:8:0

.. START:section:0:9
Safe working checklist
======================

.. END:section:0:9

.. START:section:0:10
Summary and References
======================

.. END:section:0:10

.. START:main_content:0:10
| Cross-compilation lets you build PHYTEC Linux applications on a fast host while targeting the exact processor and libraries on the board.
| The PHYTEC SDK supplies the compiler and target sysroot. Source its environment, use `CC`, `CXX` and `SDKTARGETSYSROOT`, inspect the output architecture, deploy it safely and verify it on the target. For production, move the application into Yocto so the result is repeatable.

.. END:main_content:0:10

.. START:important:0:10:0

.. important::

   A successful host build is only half the job. The binary, loader, libraries, permissions, device nodes and hardware must all work on the PHYTEC target.

.. END:important:0:10:0

References
----------
