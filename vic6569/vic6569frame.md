# MOS 6569 VIC (cycles)

## Video display
Vertical blanking occurs every frame after the last visible line until the start of the first visible line of the next frame.
Horizontal blanking occurs every line after the last visible x-coordinate until the first visible x-coordinate of the next line.

### Lines

| VIC version         | Cycles    | Lines total | VBlank | Visible | First | Last  |
|:--------------------|:----------|:------------|:-------|:--------|:------|:------|
| 6567r65A (NTSC)     | 64 (1-64) | 262 (0-261) |  13-40 | 234     | 41    | 12    |
| 6567r8, 8562 (NTSC) | 65 (1-65) | 263 (0-262) |  13-40 | 235     | 41    | 12    |
| 6569, 8565 (PAL)    | 63 (1-63) | 312 (0-311) | 300-15 | 284     | 16    | 299   |

The line is reset to zero in the middle of the lower border area on the 6567r65A and 6567r8,
and at the very top of the frame (in the vblank area) on the 6569.

### X-coordinates

| VIC version         | X-coords total   | First                        | Last                       |
|:--------------------|:-----------------|:-----------------------------|:---------------------------|
| 6567r65A (NTSC)     | 0 - 511 ($1FF)   | 412 ($19C), start cycle 1.1  | 411 ($19B), end cycle 64.2 |
| 6567r8, 8562 (NTSC) | 0 - 511 ($1FF)   | 412 ($19C), start cycle 1.1  | 411 ($19B), end cycle 65.2 |
| 6569, 8565 (PAL)    | 0 - 503 ($1F7)   | 404 ($194), start cycle 1.1  | 403 ($193), end cycle 63.2 |

| VIC version         | X-coords visible | First                        | Last                       |
|:--------------------|:-----------------|:-----------------------------|:---------------------------|
| 6567r65A (NTSC)     | 411              | 488 ($1E8), start cycle 10.2 | 388 ($184), end cycle 61.2 |
| 6567r8, 8562 (NTSC) | 418              | 489 ($1E9), start cycle 10.2 | 396 ($18C), end cycle 62.1 |
| 6569, 8565 (PAL)    | 403              | 480 ($1E0), start cycle 10.2 | 380 ($17C), end cycle 60.2 |

The x-coordinate is reset to zero at the start of the second phase of cycle 13 in all VIC versions.

The 6567r65A goes all the way from 0-511 over its 64 cycles,
the 6567r8 restarts at coordinate $18C in both halves of cycle 63 and the first half of cycle 64
(the range of $18C-$18F is reset three times and thus repeated four times in total),
the 6569 only goes up to coordinate $1F7 before the reset point in cycle 13 is reached.

