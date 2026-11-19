# MOS 6526 CIA (ports)

## Ports

The CIA has two input/output ports, poart A and port B.

## Registers

| REG | Name      | Description               |     7 |     6 |     5 |     4 |     3 |     2 |     1 |     0 |
|:----|:----------|:--------------------------|------:|------:|------:|------:|------:|------:|------:|------:|
| 0   | PRA       | Port Register A data      |   PA7 |   PA6 |   PA5 |   PA4 |   PA3 |   PA2 |   PA1 |   PA0 |
| 1   | PRB       | Port Register B data      |   PB7 |   PB6 |   PB5 |   PB4 |   PB3 |   PB2 |   PB1 |   PB0 |
| 2   | DDRA      | Data Direction Register A |   DA7 |   DA6 |   DA5 |   DA4 |   DA3 |   DA2 |   DA1 |   DA0 |
| 3   | DDRB      | Data Direction Register B |   DB7 |   DB6 |   DB5 |   DB4 |   DB3 |   DB2 |   DB1 |   DB0 |

### Power-up state / initial values

On power up/reset, the value of borth ports will be all zero and configured as inputs.
