# STM32 Half-Duplex Data Transfer

A two-board embedded Half-Duplex data trasnfer demonstration built with STM32CubeIDE for the
`NUCLEO-F091RC` and `NUCLEO-F411RE` boards.

The boards exchange a single hexadecimal counter value over a one-wire,
half-duplex UART connection. Pressing a board's user button sends its displayed
value to the other board. The receiver increments the value, wraps from `F` to
`0`, and shows the result on a seven-segment display.

## Hardware

- Boards: `NUCLEO-F091RC` and `NUCLEO-F411RE`
- IDE/toolchain: `STM32CubeIDE`
- Programmer/debugger: onboard ST-LINK
- Other: 2 common-anode seven-segment displays, x16 100 Ohm resistors,
2 breadboards, and jumper wires



## Operation

1. Power both boards and connect their grounds.
2. Connect `PA9` on the F091RC to `PA9` on the F411RE for the one-wire UART
  link.
3. Both displays start at `0`.
4. Press the onboard user button on one board to transmit its current value.
5. The receiving board adds one to the value and updates its display.
6. Press the receiving board's user button to send the new value back.
7. Continue alternating between boards. The counter cycles through `0` to `F`
  and then wraps to `0`.



## Wiring Sketch
<img width="1384" height="562" alt="DataTransferSketch_bb" src="https://github.com/user-attachments/assets/98cdf78c-9603-4ba5-8ab3-3e75caf85243" />


## 7 Segment LED Sketch
<img width="650" height="390" alt="7_Segment_LED" src="https://github.com/user-attachments/assets/1ad19b3d-9fa1-483f-9b6a-f20c2621ed2b" />



## Pin Mapping

The same display and control pins are used on both boards.


| Function          | STM32 Pin | Notes                                |
| ----------------- | --------- | ------------------------------------ |
| Display segment A | `PA0`     | GPIO output, active low              |
| Display segment B | `PA1`     | GPIO output, active low              |
| Display segment C | `PC10`    | GPIO output, active low              |
| Display segment D | `PC11`    | GPIO output, active low              |
| Display segment E | `PA4`     | GPIO output, active low              |
| Display segment F | `PA5`     | GPIO output, active low              |
| Display segment G | `PA6`     | GPIO output, active low              |
| Send button       | `PC13`    | Onboard user button, EXTI input      |
| Half-duplex UART  | `PA9`     | `USART1_TX`, open-drain with pull-up |
| Ground            | `GND`     | Connect `GND` together on both boards|


Do not connect separate UART TX and RX lines. `USART1` is configured in
single-wire half-duplex mode, so the two `PA9` pins form the data bus.

## Project Structure

- `Data Transfer F091RC Board/` - STM32CubeIDE project for the
`NUCLEO-F091RC`.
- `Data Transfer F091RC Board/Data Transfer F091RC Board.ioc` - STM32CubeMX
configuration for the F091RC board.
- `Data Transfer F091RC Board/Core/Src/main.c` - F091RC state machine, UART
handling, button interrupt, and display control.
- `Data Transfer F411RE Board/` - STM32CubeIDE project for the
`NUCLEO-F411RE`.
- `Data Transfer F411RE Board/Data Transfer F411RE Board.ioc` - STM32CubeMX
configuration for the F411RE board.
- `Data Transfer F411RE Board/Core/Src/main.c` - F411RE state machine, UART
handling, button interrupt, and display control.
- Each project's `Drivers/` directory contains the generated STM32 HAL and
CMSIS files.



## Building and Flashing

1. Install `STM32CubeIDE`.
2. Clone this repository.
3. Open STM32CubeIDE and choose `File > Import`.
4. Select `Existing Projects into Workspace`.
5. Browse to the repository and import both board projects.
6. Build `Data Transfer F091RC Board`.
7. Connect the `NUCLEO-F091RC` over USB and flash or debug its firmware.
8. Build `Data Transfer F411RE Board`.
9. Connect the `NUCLEO-F411RE` over USB and flash or debug its firmware.
10. Connect the displays and the inter-board wiring shown above.



## Configuration Notes

- The target MCUs are `STM32F091RCTx` and `STM32F411RETx`.
- Both projects run their core and peripheral buses at `48 MHz`.
- `USART1` uses single-wire half-duplex mode at `9600` baud with 8 data bits,
no parity, and 1 stop bit. The shared `PA9` line is open-drain with an
internal pull-up.
- Each transfer contains one raw byte with a value from `0` through `15`; it is
not an ASCII character.
- UART reception uses receive-to-idle interrupts. The main loop processes
received values with a three-state polling state machine and no RTOS.
- The user-button interrupt sets a transmit flag, and a `100 ms` delay provides
basic debounce before transmission.

