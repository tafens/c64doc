# MOS 6526 CIA (time-of-day clock)

## Time-of-day Clock

The time-of-day clock (TOD) is a special purpose real-time clock for various applications. It consists of a 24-hour (AM/PM) clock with a resolution of one tenth of a second. It has four registers divided into hours, minutes, seconds, and tenths. Each register is maintained in a BCD format for easy reading and conversion for driving clock displays etc. The clock requires a 50 or 60 Hz stable reference signal into the TOD pin (pin 19) of the chip to keep accurate time, this signal is typically derived from the mains power frequency.

## Alarm

The clock also has an alarm function that can raise an alarm and an interrupt request at a pre-set time. The alarm is set using the same registers as setting the current time, using the ALARM bit of control register B to select if the alarm or current time is being written. The alarm is write-only, any read from the time registers will read back the current time regardless of the state of the ALARM bit.

## Registers

| REG | Name      | Description            |     7 |     6 |     5 |     4 |     3 |     2 |     1 |     0 |
|:----|:----------|:-----------------------|------:|------:|------:|------:|------:|------:|------:|------:|
| 8   | TOD 10THS | Tenths of seconds      |   0   |     0 |     0 |     0 |   TL8 |   TL4 |   TL2 |   TL1 |
| 9   | TOD SEC   | Seconds                |   0   |   SH4 |   SH2 |   SH1 |   SL8 |   SL4 |   SL2 |   SL1 |
| A   | TOD MIN   | Minutes                |   0   |   MH4 |   MH2 |   MH1 |   ML8 |   ML4 |   ML2 |   ML1 |
| B   | TOD HR    | Hours + AM/PM          |  PM   |     0 |     0 |   HH1 |   HL8 |   HL4 |   HL2 |   HL1 |
|     |           |                        |       |       |       |       |       |       |       |       |
| D   | IMR/ICR   | Interrupt mask/control |       |       |       |       |       | ALARM |       |       |
| E   | CRA       | Control register A     | TODIN |       |       |       |       |       |       |       |
| F   | CRB       | Control register B     | ALARM |       |       |       |       |       |       |       |

The TOD clock has four registers of its own and one bit in each control register:
* The 10THS register holds one digit  for the tenths of seconds count (4 bits).
* The SEC   register holds two digits for the seconds count (3 + 4 bits).
* The MIN   register holds two digits for the minutes count (3 + 4 bits).
* The HR    register holds one digit  for the hours count (3 bits) and an AM/PM flag (1 bit).
* The ALARM bit in the interrupt control register indicates that the current time matches the set alarm time.
* The TODIN bit in control register A sets the expected input frequency on the TOD pin (0=60Hz, 1=50Hz).
* The ALARM bit in control register B sets the target for writes to the TOD time registers (0=set alarm, 1=set clock).

In addition to the time registers, the TOD clock also has a 6-stage internal tick counter.

## How the clock counts

The basis of the count is the input 50/60Hz pulse on the TOD pin. This pulse advances the internal tick counter one step each rising edge. When the tick counter equals five (TODIN=50Hz) or six (TODIN=60Hz), the counter is reset and the tenths register is incremented. Note that this is an exact comparison, and if the TODIN bit is changed at the right time, the comparison will not match, the tick counter will roll over, and the tenths increment will be delayed until it matches the TODIN bit the next time.

When the tenths digit matches exactly 9 and is incremented again, it will be reset to zero and the seconds low digit will be incremented. However, the tenths digit is actually a 4-bit value that can go up to $F, then rolls over back to zero if incremented again without affecting any other digit.

When the seconds low digit matches exactly 9 and is incremented again, it will be reset to zero and the seconds high digit will be incremented. However, the seconds low digit is actually a 4-bit value that can go up to $F, then rolls over back to zero if incremented again without affecting any other digit. When the seconds high digit matches exactly 5 and is incremented again, it will be reset to zero and the minutes low digit will be incremented. The seconds high digit is actually a 3-bit value that can go up to $7, then rolls over back to zero if incremented again without affecting any other digit.

When the minutes low digit matches exactly 9 and is incremented again, it will be reset to zero and the minutes high digit will be incremented. However, the minutes low digit is actually a 4-bit value that can go up to $F, then rolls over back to zero if incremented again without affecting any other digit. When the minutes high digit matches exactly 5 and is incremented again, it will be reset to zero and the hours low digit will be incremented. The minutes high digit is actually a 3-bit value that can go up to $7, then rolls over back to zero if incremented again without affecting any other digit.

When the hours low digit matches exactly 9 and is incremented again, it will be reset to zero and the hours high digit will be incremented. However, the hours low digit is actually a 4-bit value that can go up to $F, then rolls over back to zero if incremented again without affecting any other digit. The hours high digit is only a 1-bit value and will just toggle between 0 and 1 when incremented.

If the increment of the hours causes the hour digits to now match $12, the AM/PM bit will be toggled.
If the hour digits already matched $12 before the increment, they will only be reset to $01 without toggling the AM/PM bit.

The clock will always count following these rules, and as long as it is loaded with valid values it will always represent the time correctly. However, as noted, the registers can be loaded with invalid values as well. The maximum value they can hold is 1F\:7F\:7F\.F AM/PM due to the number of bits used for each digit. 

## Reading and writing to the registers

A specific sequence must be followed for proper setting of the time. The clock is automatically stopped by a write to the hours register, and will only start again by a write to the tenths register. This ensures that the clock will always start at the set time.

Some important points with regard to writing to the time registers:
* If the clock is not running, the internal tick counter will be reset by a write to any of the time registers.
* A write to the current time or the set alarm time will raise the alarm if they now match.
* A write of the value $12 to the hours register will flip the AM/PM bit.

A similar sequence is used when reading the time. The current value of all registers are latched by a read from the hours register and the latch is only released by after read to the tenths register. The latch is used to keep all time values constant during the read operation since the CPU can only read one register at a time and a rollover of tenths to seconds to minutes to hours could happen at any time. The clock continues to count internally during this time however. If only one register is to be read, the rollover is not a problem, but keep in mind that a read of the tenths register is required to disable the latching enabled by a read of the hours.

## Alarm and interrupts

When the current time and alarm time match exactly, the alarm will be raised by seting the ALARM bit in the interrupt control register (ICR). If the corresponding bit in the interrupt mask register (IMR) is also set, an interrupt will be raised one cycle later (old 6526 CIA) or the same cycle (new 6526A or later CIA).
