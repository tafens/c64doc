# MOS 6526 CIA (timers)

## Timers

The 6526 has two timers, timer A and timer B. The timers count down from a set 16-bit value down to zero and can either stop there or restart from the same value. Each time a timer reaches zero, an underflow condition occurs that sets that timer's flag in the interrupt register, and if the interrupt mask register also has that timer’s flag set, an interrupt is generated.

## Registers

| REG | Name      | Description             |       7 |       6 |       5 |       4 |       3 |       2 |       1 |       0 |
|:----|:----------|:------------------------|--------:|--------:|--------:|--------:|--------:|--------:|--------:|--------:|
| 4   | TALO      | Timer A count low byte  |    TAH7 |    TAH6 |    TAH5 |    TAH4 |    TAH3 |    TAH2 |    TAH1 |    TAH0 |
| 5   | TAHI      | Timer A count high byte |    TAL7 |    TAL6 |    TAL5 |    TAL4 |    TAL3 |    TAL2 |    TAL1 |    TAL0 |
| 6   | TBLO      | Timer B count low byte  |    TBH7 |    TBH6 |    TBH5 |    TBH4 |    TBH3 |    TBH2 |    TBH1 |    TBH0 |
| 7   | TBHI      | Timer B count high byte |    TBL7 |    TBL6 |    TBL5 |    TBL4 |    TBL3 |    TBL2 |    TBL1 |    TBH1 |
|     |           |                         |         |         |         |         |         |         |         |         |
| D   | IMR/ICR   | Interrupt mask/control  |         |         |         |         |         |         |      TB |      TA |
|     |           |                         |         |         |         |         |         |         |         |         |
| E   | CRA       | Control register A      |         |         |  INMODE |    LOAD | RUNMODE | OUTMODE |    PBON |   START |
| F   | CRB       | Control register B      |         |  INMODE |  INMODE |    LOAD | RUNMODE | OUTMODE |    PBON |   START |

### Power-up state / initial values
On power on/reset the current count of the timers will be 0xFFFF and the latch will be 0xFFFF.
For both timers the source will be phi1 clock, runmode continuous, no port out and state is stopped.

## Ticks

Each timer has a tick source. For timer A the source can be one of phi2 clock, or positive CNT pin transitions. For timer B the source can be one of phi2 clock, positive CNT pin transitions, timer A underflows, or timer A underflows while CNT pin is high.
The source is set with the INMODE bits in CRA/CRB.

The timer is always ticked in unison with the phi2 clock. In phi2 mode, the timer is always ticked with each phi2 clock as the tick source generates one tick each clock cycle. In CNT mode however, the tick source only generates a tick if the CNT pin has transitioned from low to high some time during the previous clock cycle.

## Start/Stop, Underflow, and Interrupt
In the second clock cycle after bit 0 (”running”) in the timer's control register has been set, the timer starts counting down from its current value back to zero. When it has reached zero and there is another tick incoming, an underflow event occurs and the timer is immediately reloaded from the latch. The reload itself however stalls the timer for one cycle, and any tick incoming during that time is ignored.

The interrupt control register (ICR) bit for the timer will be set immediately on underflow, and if interrupts are enabled for the timer, the IRQ pin will go low on the next clock cycle. Note that the newer 6526A CIA and later versions has no delay between ICR and IRQ.

```text
      latch = 000F, timer stopped with count = 000F

      * = control register START bit is set
      > = value stays the same, v = value counts down, + = reload from latch

       clock  |      |    1 |    2 |    3 |    4 |    5 |     |   15 |   16 |   17 |   18 |
      action  |    > | *  > |    > |    v |    v |    v |     |    v |   v+ |    > |    v |
       value  | 000F | 000F | 000F | 000E | 000D | 000C | ... | 0001 | 000F | 000F | 000E | ...
   interrupt  |      |      |      |      |      |      |     |      |  ICR |  IRQ |      |

```

## Force reload
In the next clock cycle after bit 4 ("force load") in the timer's control register has been set, the timer is reloaded from the latch. As with a normal underflow, the reload stalls the timer for one cycle, and any tick incoming during that time is ignored.

```text
      latch = 000F, timer running

      * = control register FORCE LOAD bit is set
      > = value stays the same, v = value counts down, + = reload from latch

       clock  |      |    1 |    2 |    3 |    4 |    5 |
      action  |    v | *  v |   >+ |    > |    v |    v |
       value  | 0005 | 0004 | 000F | 000F | 000E | 000D | ...

```

### One-shot
In one-shot mode the timer will stop and reload immediately when an underflow event occurs. Setting/clearing the one-shot control bit in the control register will affect the timer also when running, but there is a small latency when clearing it.
When set it will take effect immediately (in the same clock cycle as set), but when cleared it will remain in effect for the rest of the clock cycle. This means that clearing it in the same clock cycle as an underfow event will occur (i.e. when the counter is still at 0001 but will go to 0000) the timer will still stop and reload as if it was still set for the entire clock cycle. 

## Writing to the latch
Writing to the latch always updates the value in the latch that will be used on the next reaload of the timer. If the timer is stopped at the time of writing the high byte of the latch, the value will also set the current value of the timer (however the new value will take on additional cycle to load into the timer counter - just as with underflow and forced reload). If the timer is running, the new value in the latch will not be used until a timer reload is triggered. 

---

### Notes
In phi2 mode there is always another tick inbound, so a zero value will never be seen, and the latch value will be seen for two consecutive clocks before the count down is commenced.

