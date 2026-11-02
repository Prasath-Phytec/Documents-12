.. START:section:0:0
Electrical pin configuration
============================

.. END:section:0:0

.. START:main_content:0:0
A processor pin is more than a name in a device tree. It is connected to a pad cell with a mux selector and electrical controls. The mux selects which internal peripheral reaches the pad. Pad controls affect how the signal behaves electrically, such as input enable, pull resistors, drive strength and slew rate.

.. END:main_content:0:0

.. START:table:0:0:1

.. csv-table:: Table 1 — Three layers that must agree
   :header: "Layer", "Question"
   :widths: 30, 70
   :align: center

   "Mux function", "Which internal peripheral or GPIO owns the pad?"
   "Electrical configuration", "How does the pad drive or receive the signal?"
   "Board hardware", "What voltage, pull, load and route exist outside the SoC?"

.. END:table:0:0:1

.. START:tip:0:0:2

.. tip::

   A correct mux can still produce a bad signal if the voltage domain, pull resistor, drive strength or slew rate is wrong.

.. END:tip:0:0:2

.. START:section:0:1
Register vocabulary
===================

.. END:section:0:1

.. START:table:0:1:0

.. csv-table:: Table 2 — Common register terms
   :header: "Term", "Simple meaning"
   :widths: 30, 70
   :align: center

   "Pad or pin", "The physical SoC connection that can carry a signal"
   "Mux selector", "A field choosing UART, SPI, GPIO or another function"
   "Pad control", "Bits controlling the electrical behavior of the pad"
   "Input enable", "Allows the input receiver to observe the external signal"
   "Pull-up", "Weakly biases a signal toward the high voltage level"
   "Pull-down", "Weakly biases a signal toward ground"
   "Drive strength", "How strongly an output can source or sink current"
   "Slew rate", "How quickly an output changes voltage"
   "Open drain", "Output can pull low and needs an external pull-up for high"
   "Schmitt trigger", "Input behavior that can improve noise tolerance on supported pads"

.. END:table:0:1:0

The names and available bits differ between NXP and TI. Some settings are represented by macros; others are encoded in a hexadecimal value. Always use the definitions supplied for the exact SoC.

.. START:section:0:2
Mux control versus pad control
==============================

.. END:section:0:2

.. START:main_content:0:2
Mux control answers: “Which function is connected to this pad?” Pad control answers: “How should this electrical connection behave?” These are related but separate decisions.

.. END:main_content:0:2

.. START:table:0:2:0

.. csv-table:: Table 3 — Example of separate choices
   :header: "Choice", "Example"
   :widths: 30, 70
   :align: center

   "Mux", "Select an SPI clock function instead of GPIO"
   "Input/output direction", "Enable the input receiver for a clock or data input"
   "Bias", "Use no internal pull when the board already has an external resistor"
   "Drive", "Select enough output strength for the trace and load"
   "Slew", "Choose a transition speed that meets timing without excess noise"

.. END:table:0:2:0

.. START:warning:0:2:1

.. warning::

   Changing a pad-control value does not select a peripheral. Changing only the mux does not guarantee a reliable electrical signal.

.. END:warning:0:2:1

.. START:section:0:3
Find the authoritative register definition
==========================================

.. END:section:0:3

.. START:main_content:0:3
Use the following order when choosing a value: the PHYTEC hardware manual and schematic, the exact SoC reference manual, the BSP pinctrl header and existing board DTS examples. A random internet example is not an authority.

.. END:main_content:0:3

.. START:table:0:3:0

.. csv-table:: Table 4 — Where to look
   :header: "Source", "Use it for"
   :widths: 35, 65
   :align: center

   "PHYTEC hardware manual", "Voltage domains, routed interfaces and board constraints"
   "PHYTEC schematic", "External pulls, level shifters, loads and shared nets"
   "SoC reference manual", "Register address, bit fields, reset state and electrical limits"
   "BSP pinfunc/pinctrl header", "Valid symbolic pad names and supported macros"
   "Existing PHYTEC DTS", "Working syntax and board-specific conventions"
   "Running board", "Final deployed state and actual hardware behavior"

.. END:table:0:3:0

.. START:code:0:3:1

.. code-block:: console

   # Run from the root of the NXP PHYTEC kernel repository
   $ rg -n 'fsl,pins|MX8MP_IOMUXC|IMX93_PAD|IMX95_PAD' \
       arch/arm64/boot/dts/freescale \
       include/dt-bindings/pinctrl

.. END:code:0:3:1

.. START:section:0:4
NXP i.MX electrical configuration
=================================

.. END:section:0:4

.. START:main_content:0:4
NXP i.MX Linux device trees commonly use `fsl,pins`. An entry normally contains a symbolic pad/mux macro followed by a pad-control value. The macro selects the pad and function; the value configures electrical behavior according to the exact i.MX family.

.. END:main_content:0:4

.. START:code:0:4:0

.. code-block:: console

   /* Illustrative example based on PHYTEC's i.MX 8M Plus Pollux BSP */
   &iomuxc {
       pinctrl_uart1: uart1grp {
           fsl,pins = <
               MX8MP_IOMUXC_UART1_RXD__UART1_DCE_RX 0x140
               MX8MP_IOMUXC_UART1_TXD__UART1_DCE_TX 0x140
           >;
       };
   };

.. END:code:0:4:0

.. START:table:0:4:1

.. csv-table:: Table 5 — How to read an NXP entry
   :header: "Part", "Meaning"
   :widths: 35, 65
   :align: center

   "MX8MP_IOMUXC_...", "Pad and alternate function selected by the macro"
   "0x140 (example)", "Example pad-control encoding; not a universal value"
   "pinctrl-0", "The device node's default pinctrl state"
   "pinctrl-names", "Names such as default, sleep or idle"

.. END:table:0:4:1

.. START:important:0:4:2

.. important::

   The same hexadecimal number can mean different things on different i.MX families. Read the matching `imx*-pinfunc.h` definitions and reference manual.

.. END:important:0:4:2

.. START:section:0:5
NXP pad-control fields
======================

.. END:section:0:5

.. START:main_content:0:5
Depending on the i.MX family, pad-control fields can include pull configuration, pull/keeper behavior, drive strength, slew rate, open-drain selection and hysteresis. The bit positions and legal combinations are not universal.

.. END:main_content:0:5

.. START:table:0:5:0

.. csv-table:: Table 6 — NXP settings to review
   :header: "Setting", "Review question"
   :widths: 30, 70
   :align: center

   "Pull or keeper", "Does the board already provide an external bias?"
   "Drive strength", "Can the output drive the trace and load without violating limits?"
   "Slew rate", "Is the edge speed suitable for timing and signal integrity?"
   "Open drain", "Is the signal shared and externally pulled high?"
   "Hysteresis", "Would noise filtering help the input on this signal?"
   "Voltage domain", "Does the selected bank match the PHYTEC hardware voltage?"

.. END:table:0:5:0

.. START:warning:0:5:1

.. warning::

   Do not enable an internal pull against a strong external driver. This can waste current, distort a bus or create contention.

.. END:warning:0:5:1

NXP practical review
--------------------

.. START:code:0:5:3

.. code-block:: console

    From the root of the NXP PHYTEC kernel repository
   # rg -n -C 3 'fsl,pins' arch/arm64/boot/dts/freescale/imx8mp-phyboard-pollux-rdk.dts

.. END:code:0:5:3

When reviewing a value, write down the intended electrical meaning in plain English. For example: “No internal bias, input enabled, medium drive, slow slew.” Do not rely on the hexadecimal number alone.

.. START:section:0:6
TI electrical configuration and settings
========================================

.. END:section:0:6

.. START:main_content:0:6
TI K3 Linux device trees commonly use `pinctrl-single,pins`. A pin entry contains a register offset and a value written to the pin configuration register. PHYTEC trees may use TI macros to make the SoC-specific offset and mode easier to read.

.. END:main_content:0:6

.. START:code:0:6:0

.. code-block:: bash

   /* Illustrative AM62x UART0 pinctrl form; confirm board routing */
   &main_pmx0 {
       main_uart0_pins_default: main-uart0-default-pins {
           pinctrl-single,pins = <
               AM62X_IOPAD(0x1c8, PIN_INPUT, 0)  /* UART0_RXD */
               AM62X_IOPAD(0x1cc, PIN_OUTPUT, 0) /* UART0_TXD */
           >;
       };
   };

.. END:code:0:6:0

.. START:table:0:6:1

.. csv-table:: Table 7 — How to read a TI entry
   :header: "Part", "Meaning"
   :widths: 30, 70
   :align: center

   "Offset", "The pin configuration register offset for the pad"
   "Pad value/flags", "Electrical configuration and mux-related bits"
   "Macro", "SoC-specific helper that may combine offset and mode"
   "Pinctrl state", "A named group selected by the Linux device node"

.. END:table:0:6:1

.. START:important:0:6:2

.. important::

   TI offset and bit definitions differ between AM62x, AM64x, AM68x and other K3 devices. Use the exact binding, header and SoC documentation.

.. END:important:0:6:2

TI electrical settings
----------------------

TI pad configuration may include input enable, pull enable, pull direction, drive strength, slew rate and mux mode. The available fields depend on the pad and device family. Some signals have special requirements in the TI technical reference manual.

.. START:table:0:6:4

.. csv-table:: Table 8 — TI settings to review
   :header: "Setting", "Typical concern"
   :widths: 30, 70
   :align: center

   "Input enable", "Must be enabled for an input or bidirectional signal"
   "Pull enable", "May be unnecessary or harmful when an external resistor exists"
   "Pull direction", "Must not fight an external device or bus"
   "Drive strength", "Must meet load and timing without excess current"
   "Slew rate", "Fast edges can increase ringing and EMI"
   "Mux mode", "Must select the intended peripheral rather than GPIO"

.. END:table:0:6:4

.. START:code:0:6:5

.. code-block:: console

   # From the root of the TI PHYTEC kernel repository
   $ rg -n -C 3 'pinctrl-single,pins' \
       arch/arm64/boot/dts/ti/k3-am625-phyboard-lyra-rdk.dts

.. END:code:0:6:5

Choosing pull-up and pull-down settings
---------------------------------------

A pull resistor is a weak default bias used when no stronger device is driving the line. It can keep a GPIO, reset line or bus signal at a known level. It is not a replacement for a required external resistor on every interface.

.. START:table:0:6:7

.. csv-table:: Table 9 — Pull-setting decisions
   :header: "Signal type", "Questions"
   :widths: 30, 70
   :align: center

   "I2C", "Are external pull-ups fitted and sized for the bus speed and capacitance?"
   "SPI chip select", "What state is safe while the controller is not initialized?"
   "Reset", "What level keeps the device safely held or released?"
   "GPIO input", "What level should an unconnected or inactive input read?"
   "UART", "Could a pull affect an external transceiver or boot console?"
   "Open-drain interrupt", "Is there a valid pull-up and input receiver enabled?"

.. END:table:0:6:7

.. START:warning:0:6:8

.. warning::

   Never assume “pull-up” or “pull-down” is safe from the signal name alone. Check the peripheral datasheet, PHYTEC schematic and power-up behavior.

.. END:warning:0:6:8

.. START:section:0:7
Drive strength and slew rate
============================

.. END:section:0:7

.. START:main_content:0:7
Drive strength controls the output current capability. Slew rate controls how quickly the output changes voltage. More drive and faster slew are not automatically better: they can increase ringing, overshoot, crosstalk and electromagnetic interference.

.. END:main_content:0:7

.. START:table:0:7:1

.. csv-table:: Table 10 — Symptoms and possible electrical causes
   :header: "Symptom", "Possible cause"
   :widths: 35, 65
   :align: center

   "Slow or rounded edges", "Insufficient drive, excessive load or too-slow slew"
   "Ringing or overshoot", "Too much drive, too-fast slew or poor termination"
   "I2C stuck low", "External device, wrong pull-up or pin not released"
   "SPI data errors", "Wrong input enable, timing, drive or signal integrity"
   "Random GPIO interrupts", "Floating input or unsuitable pull configuration"
   "UART framing errors", "Voltage, level shifter, baud timing or signal integrity issue"

.. END:table:0:7:1

.. START:tip:0:7:2

.. tip::

   Use an oscilloscope or logic analyzer when the software configuration looks correct but the signal is unreliable. A register dump alone cannot show waveform quality.

.. END:tip:0:7:2

.. START:section:0:8
Open-drain and bidirectional signals
====================================

.. END:section:0:8

.. START:main_content:0:8
Open-drain buses such as I2C use devices that pull the line low while an external resistor returns it high. Both SDA and SCL need the correct input/output behavior. Reset, interrupt and GPIO lines can also be bidirectional or open-drain depending on the circuit.

.. END:main_content:0:8

.. START:table:0:8:0

.. csv-table:: Table 11 — Open-drain review
   :header: "Check", "Why"
   :widths: 35, 65
   :align: center

   "External pull-up voltage", "Must match the SoC and attached device voltage"
   "Pull-up resistance", "Controls rise time and current when the line is low"
   "Input enable", "Allows the controller to read the shared line"
   "Output mode", "Must not drive high against another device"
   "Idle state", "Line should return high when no device pulls low"

.. END:table:0:8:0

.. START:section:0:9
Electrical constraints and Device-tree states
=============================================

.. END:section:0:9

Electrical constraints from the board
-------------------------------------

The SoC setting is only one part of the circuit. PHYTEC boards may include level translators, external pull resistors, series resistors, ESD components, multiplexers or onboard peripherals.

.. START:table:0:9:1

.. csv-table:: Table 12 — Board-level evidence
   :header: "Evidence", "What it tells you"
   :widths: 30, 70
   :align: center

   "Voltage rail", "The logic-high range and safe bank voltage"
   "External resistor", "Whether the board already supplies a pull or termination"
   "Level shifter", "Whether direction and voltage translation are involved"
   "Series resistor", "Affects edge rate and signal integrity"
   "Onboard peripheral", "May own the line or load it even when unused"
   "Connector note", "Shows whether the signal is available to the user"

.. END:table:0:9:1

Device-tree states: default and sleep
-------------------------------------

Linux can select different pinctrl states for active use, idle use and sleep. A low-power state may change pulls or output levels. Review all states, not only the `default` group.

.. START:code:0:9:3

.. code-block:: console

   /* Illustrative structure */
   &peripheral {
       pinctrl-names = "default", "sleep";
       pinctrl-0 = <&peripheral_default>;
       pinctrl-1 = <&peripheral_sleep>;
       status = "okay";
   };

.. END:code:0:9:3

.. START:table:0:9:4

.. csv-table:: Table 13 — Pinctrl state review
   :header: "State", "Electrical question"
   :widths: 25, 75
   :align: center

   "Default", "Are mux, direction and drive correct during normal operation?"
   "Idle", "Does the line remain safe when the peripheral is inactive?"
   "Sleep", "Does the state reduce leakage without breaking wakeup?"
   "Reset", "Does the device see the required level during initialization?"

.. END:table:0:9:4

.. START:important:0:9:5

.. important::

   A peripheral can work in the default state and still fail during suspend, resume or runtime power management if the sleep state is wrong.

.. END:important:0:9:5

.. START:section:0:10
Verify register settings safely
===============================

.. END:section:0:10

.. START:main_content:0:10
Prefer the Linux pinctrl debug information and source definitions. Direct register reads can help during expert debugging, but reading or writing an address without the exact SoC manual can be unsafe.

.. END:main_content:0:10

.. START:code:0:10:0

.. code-block:: console

   # Inspect Linux pinctrl ownership
   $ mount -t debugfs none /sys/kernel/debug 2>/dev/null
   $ cat /sys/kernel/debug/pinctrl/*/pinmux-pins 2>/dev/null
   $ cat /sys/kernel/debug/pinctrl/*/pins 2>/dev/null
   
   # Search logs for pinctrl and peripheral errors
   $ dmesg | grep -Ei 'pinctrl|gpio|spi|i2c|uart|can'

.. END:code:0:10:0

.. START:table:0:10:1

.. csv-table:: Table 14 — Evidence levels
   :header: "Evidence", "Confidence"
   :widths: 40, 60
   :align: center

   "DTS source contains a value", "The source was edited, not necessarily built or used"
   "Built DTB contains the node", "The artifact contains the change"
   "Live DT contains the node", "The running kernel received the change"
   "Pinctrl debugfs shows ownership", "The driver requested the pin state"
   "Measured signal works", "The complete software and hardware path works"

.. END:table:0:10:1

.. START:warning:0:10:2

.. warning::

   Do not use a raw register write as a permanent fix. It disappears on reboot and can interfere with drivers, power management or another peripheral.

.. END:warning:0:10:2

.. START:section:0:11
Common errors
=============

.. END:section:0:11

.. START:table:0:11:0

.. csv-table:: Table 15 — Electrical configuration troubleshooting
   :header: "Symptom", "Likely cause", "First action"
   :widths: 30, 35, 35
   :align: center

   "Peripheral is missing", "Mux mode or device node is wrong", "Check the live DT and pinctrl ownership"
   "Input always reads one level", "Wrong pull, input disabled or external driver", "Check schematic, input enable and measured voltage"
   "Bus has errors", "Pull-up, drive, slew or voltage issue", "Check board resistors and waveform"
   "Output cannot reach a level", "Wrong bank voltage, load or drive setting", "Check voltage domain and external load"
   "Signal rings badly", "Drive or slew too aggressive", "Reduce strength/speed and inspect the waveform"
   "Sleep/resume breaks device", "Incorrect sleep pinctrl state", "Compare default and sleep groups"
   "Changing value has no effect", "Wrong DTB or driver state not selected", "Verify the deployed DTB and debugfs state"
   "Board no longer boots", "Critical console, storage or reset pin changed", "Restore the known-good DTB and pin configuration"

.. END:table:0:11:0

.. START:section:0:12
Safe working checklist
======================

.. END:section:0:12

.. START:list:0:12:0

.. LIST_META: {"type": "auto", "head": "", "items": ["Identify the exact PHYTEC board, SoM, processor, revision and BSP release.", "Find the signal and its electrical requirements in the PHYTEC documents.", "Read the exact SoC reference manual register fields.", "Use the BSP pinctrl header and existing DTS examples.", "Separate mux selection from pad electrical configuration.", "Check external pulls, voltage rails, loads, level shifters and termination.", "Choose the least aggressive valid drive and slew setting first.", "Review default, idle and sleep pinctrl states.", "Build and verify the deployed DTB and live pinctrl ownership.", "Measure the signal when needed; record the final value, reason and evidence."]}

1. Identify the exact PHYTEC board, SoM, processor, revision and BSP release.
2. Find the signal and its electrical requirements in the PHYTEC documents.
3. Read the exact SoC reference manual register fields.
4. Use the BSP pinctrl header and existing DTS examples.
5. Separate mux selection from pad electrical configuration.
6. Check external pulls, voltage rails, loads, level shifters and termination.
7. Choose the least aggressive valid drive and slew setting first.
8. Review default, idle and sleep pinctrl states.
9. Build and verify the deployed DTB and live pinctrl ownership.
10. Measure the signal when needed; record the final value, reason and evidence.

.. END:list:0:12:0

.. START:section:0:13
Summary and References
======================

.. END:section:0:13

.. START:main_content:0:13
Pinmux registers connect Linux configuration to real electrical behavior. The mux selects the internal function, while pad-control fields define input behavior, bias, drive and edge speed. NXP and TI encode these settings differently, so the exact SoC reference manual and PHYTEC board design must guide every value. Use symbolic macros and existing BSP patterns where possible, verify the live pinctrl state, and measure the signal when software evidence is not enough. This verification process confirms the deployed result.

.. END:main_content:0:13

.. START:note:0:13:0

.. note::

   Hexadecimal pad-control values have meaning only for the exact SoC family and register definition that owns them.

.. END:note:0:13:0

.. START:danger:0:13:1

.. danger::

   Do not write unknown register values or use an incompatible voltage setting. Incorrect electrical configuration can damage hardware or create unsafe contention.

.. END:danger:0:13:1

References
----------

Linux Kernel — Pin Control Documentation

Linux Kernel — pinctrl-single Binding

PHYTEC Documentation Home

PHYTEC AM62x Documentation

PHYTEC i.MX 8M Plus BSP Documentation

Supplied NXP PHYTEC kernel tree: linux-phytec-imx-6.12.34-2.1.0-phy9/

Supplied TI PHYTEC kernel tree: linux-phytec-ti-6.1.80-09.02.00.009-phy8/

PHYTEC i.MX 8M Plus UART1 pinmux example (Pollux BSP)

Linux device-tree example — TI AM62x PHYBOARD-Lyra
