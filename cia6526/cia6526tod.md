# MOS 6526 CIA (time-of-day clock)

## Clock

The time-of-day clock (TOD) is a special purpose real-time clock for various applications. It consists of a 24-hour (AM/PM) clock with a resolution of one tenth of a second. It has four registers divided into hours, minutes, seconds, and tenths. Each register is maintained in a BCD format for easy reading and conversion for driving clock displays etc. The clock requires a 50 or 60 Hz stable reference signal into the TOD pin (pin 19) of the chip to keep accurate time, this signal is typically derived from the mains power frequency.

## Alarm

The clock also has an alarm function that can raise an alarm and an interrupt request at a pre-set time. The alarm is set using the same registers as setting the current time, using the ALARM bit of control register B to select if the alarm or current time is being written. The alarm is write-only, any read from the time registers will read back the current time regardless of the state of the ALARM bit.

## Registers

| REG | Name      | Description            |     7 |     6 |     5 |     4 |     3 |     2 |     1 |     0 |
|:----|:----------|:-----------------------|------:|------:|------:|------:|------:|------:|------:|------:|
| 8   | TOD 10THS | Tenths of seconds      |     0 |     0 |     0 |     0 |   TL8 |   TL4 |   TL2 |   TL1 |
| 9   | TOD SEC   | Seconds                |     0 |   SH4 |   SH2 |   SH1 |   SL8 |   SL4 |   SL2 |   SL1 |
| A   | TOD MIN   | Minutes                |     0 |   MH4 |   MH2 |   MH1 |   ML8 |   ML4 |   ML2 |   ML1 |
| B   | TOD HR    | Hours + AM/PM          |    PM |     0 |     0 |   HH1 |   HL8 |   HL4 |   HL2 |   HL1 |
|     |           |                        |       |       |       |       |       |       |       |       |
| D   | IMR/ICR   | Interrupt mask/control |       |       |       |       |       | ALARM |       |       |
|     |           |                        |       |       |       |       |       |       |       |       |
| E   | CRA       | Control register A     | TODIN |       |       |       |       |       |       |       |
| F   | CRB       | Control register B     | ALARM |       |       |       |       |       |       |       |

The TOD clock has four registers of its own and one bit in each control register and the interrupt control/mask register:
* The 10THS register holds one digit  for the tenths of seconds count (4 bits).
* The SEC   register holds two digits for the seconds count (3 + 4 bits).
* The MIN   register holds two digits for the minutes count (3 + 4 bits).
* The HR    register holds two digits for the hours count (1 + 3 bits) and an AM/PM flag (1 bit).
* The ALARM bit in the interrupt control register indicates that the current time matches the set alarm time.
* The TODIN bit in control register A sets the expected input frequency on the TOD pin (0=60Hz, 1=50Hz).
* The ALARM bit in control register B sets the target for writes to the TOD time registers (0=set alarm, 1=set clock).

In addition to the time registers, the TOD clock also has a 6-stage internal tick counter.

### Power-up state / initial values

On power up/reset, the current time will be 00\:00\:00.0, the set alarm time will be 01\:00\:00.0, and TODIN will be 0 (60Hz).

## How the clock counts

The basis of the count is the input 50/60Hz pulse on the TOD pin. This pulse advances the six-stage internal tick counter one step each rising edge. When the tick counter equals exactly five (TODIN=50Hz) or exactly six (TODIN=60Hz) after the pulse is counted, the counter is reset and the tenths digit is incremented. The other digits will stay the same or also increment according to the internal rules:

* Tenths
    * If the tenths digit matches exactly 9 before an increment,<br/>
      it will be reset to zero and the seconds low digit will be incremented
* Seconds
    * If the seconds low digit matches exactly 9 before an increment,<br/>
      it will be reset to zero and the seconds high digit an be incremented
    * If the seconds high digit matches exactly 5 before an increment,<br/>
      it will be reset to zero and the minutes low digit will be incremented.
* Minutes
    * if the minutes low digit matches exactly 9 before an increment,<br/>
      it will be reset to zero and the minutes high digit will be incremented
    * If the minutes high digit matches exactly 5 before an increment,<br/>
      it will be reset to zero and the hours low digit will be incremented
* Hours
    * If the hours high and low digits matches exactly 09 before an increment, they will be set to 10
    * If the hours high and low digits matches exactly 12 before an increment, they will be set to 01
    * After an increment, if the hours high and low digits now matches exactly 12, the AP/PM bit is toggled

The clock will always count following these rules, and as long as it is loaded with valid values it will always represent the time correctly. But the digits are actually binary registers (one per digit) and they will behave as such. The tenths, seconds low, minutes low, and hours low are 4-bit registers that each can hold values from $0-$F, and the seconds high, and minutes high are 3-bit registers that can hold values from $0-$7. Any of those values can be written to them respectively and be read back, and when incremented past their maximum they will just wrap around to zero.

The minimum possible value is thus 00\:00\:00.0 AM, and the maximum value is 1F\:7F\:7F\.F PM due to the number of bits used for each digit. Thanks to the rules and rollovers of the digits, the clock will however always reach a valid value sooner or later from any value and continue with only valid values from there.

Moreover, the 6-stage internal counter relies on the exact comparison to 5 or 6 according to the state of the TODIN bit. If the TODIN bit is changed at just the right time, the comparison will not match and the counter will continue and roll over. If that happens, the tenths increment will be delayed until it matches the TODIN bit the next time around.

## Reading and writing to the registers

A specific sequence must be followed for proper setting of the time. The clock is automatically stopped by a write to the hours register, and will only start again by a write to the tenths register. This ensures that the clock will always start at the set time.

Some important points with regard to writing to the TOD registers:
* If the clock is not running, the internal tick counter will be reset by a write to any of the TOD registers.
* A write of the value $12 to the hours register will flip the AM/PM bit
  (to the opposite value it was set to with the same write).
* A write to the current time or the set alarm time will raise the alarm if they now match.

A similar sequence is used when reading the time. The current value of all registers are latched by a read from the hours register and the latch is only released by after read to the tenths register. The latch is used to keep all time values constant during the read operation since the CPU can only read one register at a time and a rollover of tenths to seconds to minutes to hours could happen at any time. The clock continues to count internally during this time however. If only one register is to be read, the rollover is not a problem, but keep in mind that a read of the tenths register is required to disable the latching enabled by a read of the hours.

## Alarm and interrupts

When the current time and alarm time match exactly, the alarm will be raised by setting the ALARM bit in the interrupt control register (ICR). If the corresponding bit in the interrupt mask register (IMR) is also set, an interrupt will be raised one cycle later (old 6526 CIA) or the same cycle (new 6526A or later CIA).
