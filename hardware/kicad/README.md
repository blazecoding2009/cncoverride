# CNC Override Pendant PCB · R1

R1 replaces the harness carrier with a **190 × 170 mm direct-mount panel PCB**. The display, four knobs and eight buttons sit on the front. The ESP32-C6-DevKitC-1 and expander mount on the rear. Four Ø3.2 mm holes on a 182 × 162 mm pattern mount the main board. Dimensions and panel clearances are provisional.

| References | Direct assembly |
|---|---|
| ENC1, ENC2 | Bourns PEC11H-4215F-S0024 push encoders for Feed and Spindle. Includes plated slots for the two anchors per encoder. |
| SW1, SW2 | Lorlin CK1049, one-pole PCB rotary switch with break-before-make contacts. Set the stop washers for three and five positions respectively. |
| SW3–SW10 | 1U Cherry MX PCB-mount switches with five pins including the plastic locating pegs. Use labeled keycaps for X/Y, Z, Feed Hold and Cycle Start. |
| J1, H5–H8 | Adafruit 2050 resistive HX8357D TFT breakout, rotated to landscape. Solder a 2.54 mm header to the breakout's JP2 and to the pendant PCB. Use four M2.5 standoffs so the header carries no display mounting load. |
| J2, J3 | Solder the DevKitC's two 16-pin strips directly to the rear. The native USB-C port faces the top edge and goes to the Raspberry Pi. |
| U1 | Rear-mounted MCP23017-E/SP, 7.62 mm DIP-28. A0–A2 are grounded for address 0x20. |

## KiCad project

Open **cncoverride-carrier.kicad_pro** in KiCad 8 or later. The single-sheet schematic and PCB use the included project-local symbols and footprints. The board references are linked to their schematic instances.

## TFT connection

J1 mirrors the physical JP2 hole pattern from the Adafruit 2050 Eagle CAD. The glass faces the controls. The four TFT mounting holes use the 91.44 × 60.96 mm landscape spacing from Adafruit's CAD. Set the TFT's IM2 jumper for SPI before mounting it. Set the display rotation in firmware.

| JP2 pin | Signal |
|---|---|
| 1, 15 | GND |
| 2 | USB 5 V to VIN |
| 4 | TFT SCK |
| 6 | TFT MOSI |
| 7 | TFT CS |
| 8 | TFT D/C |
| 19 | SD CS held high by R4 |
| Other holes | Unassigned |

## Control inputs

| ESP32-C6 GPIO | Function |
|---|---|
| 0 / 1 | Feed encoder A / B |
| 2 / 3 | Spindle encoder A / B |
| 6 / 7 | Expander I²C SDA / SCL |
| 10 / 11 | Feed Hold / Cycle Start |
| 18 / 19 | Z+ / Z− |
| 20 / 21 / 22 / 23 | TFT SCK / MOSI / CS / D/C |

MCP23017 bank A bits 0–6 are X+, X−, Y+, Y−, Feed push, Spindle push, and Rapid 25%. Bank B bits 0–6 are Jog OFF, .001″, .010″, .100″, CONT, Rapid 50%, and Rapid 100%. Enable their pull-ups. GPA7 and GPB7 are reserved because the current datasheet specifies them as output only. Selector common A goes to ground. Rapid uses throws 1–3 for 25%, 50% and 100%. Jog uses throws 1–5 in the order above. In firmware, an open or ambiguous jog selector disarms jogging.

RN1 is an eight-resistor bussed 10 kΩ pull-up network with common on pin 1. R1/R2 are 4.7 kΩ for I²C, R3/R4 are 10 kΩ for expander reset and TFT SD CS. C1 is 100 nF across the expander supply. C2 is 10 µF near the TFT supply. The DevKitC's 3.3 V rail powers the expander. Its 5 V rail supplies TFT VIN. The machine's hardware E-stop uses the machine's documented circuit.

## Job buttons

- **SW9 Feed Hold (GPIO10)** requests a controlled hold. Use CNCjs gcode:pause during a running job so the sender pauses along with Grbl. Feed Hold leaves the spindle running.
- **SW10 Cycle Start (GPIO11)** starts the job already loaded in CNCjs when CNCjs and Grbl are idle. Use gcode:start for a new cycle and gcode:resume to continue a paused job when Grbl is ready. Require a fresh button press and a live controller connection. Suppress Cycle Start while Grbl is jogging, homing, alarmed, or in a door state.

The command behavior is the target for the firmware and Pi bridge. [CNCjs job commands](https://github.com/cncjs/cncjs/blob/v1.11.5/src/server/controllers/Grbl/GrblController.js#L1305-L1359) · [Grbl real-time commands](https://github.com/gnea/grbl/blob/v1.1h.20190825/doc/markdown/commands.md#grbl-v11-realtime-commands)

## Review status

All numbered schematic pins and PCB pads agree, including unused pins. The PCB placement and assigned nets are ready for routing. KiCad ERC/DRC and physical fit are pending. Check the actual TFT revision and print the hole patterns at 1:1 before fabrication. Set the enclosure depth from the TFT standoffs, rotary shoulder heights, keycap travel and USB plug clearance. Use enclosure support for knob forces and space around the Feed Hold and Cycle Start caps.

[Bourns encoder](https://www.bourns.com/docs/product-datasheets/pec11h.pdf?sfvrsn=a18340f6_5) · [Lorlin CK drilling and part numbers](https://www.electrokit.com/upload/product/41002/41002752/LorlinCK-7.pdf) · [KiCad MX drilling pattern](https://github.com/KiCad/kicad-footprints/blob/master/Button_Switch_Keyboard.pretty/SW_Cherry_MX_1.00u_PCB.kicad_mod) · [Adafruit 2050 breakout CAD](https://github.com/adafruit/3.5inch-TFT-Breakout-PCB) · [Espressif pin tables](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32c6/esp32-c6-devkitc-1/user_guide.html#header-block) · [MCP23017 datasheet](https://ww1.microchip.com/downloads/aemDocuments/documents/APID/ProductDocuments/DataSheets/MCP23017-Data-Sheet-DS20001952.pdf)
