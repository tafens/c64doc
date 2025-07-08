# 1530/1531/C2N Datassette (counter)

## Counter

The counter has three digits counting from 000 up to 999, and then wraps around to 000 again;
it can also be reset back to 000 again at any time unsig the reset button.

The rightmost digit is driven from the right tape spool spindle via a belt and reduction gears. When it reaches 9 and
moves to zero again, a peg catches the digit to the left of it and moves it along one step, advancing it by one and so on. 

## Calculating the counter value

Calculating where the counter at is a little bit tricky, because as the tape spool fills up, the actual radius of it increases
making it rotate slower by each turn. For each revolution it will increase in radius by the thickness of the tape.
Additionally, the belt and gears that drives the counter from the spool will provide a ratio for how much
the counter increases for each revolution of the spool.

### Required information

What we will need to know in order to calculate the counter value is:
- how many revolutions the tape spool has made this far;<br/>
  (elapsed time, tape speed, tape thickness, and empty spool radius)
- how many revolutions of the tape spool that is required to advance the counter by one:<br/>
  (belt/gear ratio)

These are as follows:

- **t** = elapsed time (s);<br/>
          time in seconds that the tape has been running<br/>

- **v** = tape speed (m/s);<br/>
          the standard speed for cassette tape is 1 7/8 in/s, or 4.7625 cm/s<br/>
          **0.047625 m/s**

- **d** = tape thickness (m);<br/>
          tape thickness varies depending on the length of the tape in the cassette,
          ranging from 16-18 um in C60 cassettes and shorter, 11-12 um in C90, and
          even thinner in longer, less common tapes such as C120;
          for purpose of general calculation we assume 12 um as the most common tape stock used<br/>
          **0.000012 m**

- **r** = empty spool radius (m);<br/>
          for most cassettes, the empty spool diameter is 22 mm, making the radius 11 mm<br/>
          **0.0011 m**

- **f** = belt/gear ratio;<br/>
          this is how much the counter increases for each revolution of the tape spool,
          and this reduction ratio of the belt and gears together is approximately 0.525<br/>
          **0.525**

### Formulas for calculation

Total length of tape wound up on the spool at time ```t```:
```
L(t) = v*t
```

It is not as simple as dividing this by the cirumference of the tape spool however,
because as the tape winds up, each additional revolution wraps around a slightly larger radius.

The length of the tape that is wound up on the spool by the ```n```:th *single* revolution is:
```
L(n) = 2*pi*(r + d*(n-1))
```

It follows that the total length of tape wound up after ```N``` full revolutions
is the sum of the length added by each revolution from ```1``` to ```N```:
```
     N
L = sum 2*pi*(r + d*(n-1))  =>  L = 2*pi*(N*r + d*(N*(N-1))/2)  =>  L = pi*(d*N^2 + 2*r*N - d*N)
    n=1                             ------------(1)-----------          ----------(2)-----------
```

Breaking this down in smaller steps using the following standard summation formulas:
```
 N             N      |    N            |    N
sum A*n = A * sum n   |   sum k = N*k   |   sum n = (N*(N+1))/2 
n=1           n=1     |   n=1           |   n=1
```

```
     N                                 N                            N       N
L = sum 2*pi*(r + d*(n-1))  =  2*pi * sum (r + d*(n-1))  =  2*pi*( sum r + sum d*(n-1) )  =>
    n=1                               n=1                          n=1     N=1

            N           N                    N           N       N
L = 2*pi*( sum r + d * sum n-1 )  =  2*pi*( sum r + d*( sum n - sum 1 ) )
           n=1         n=1                  n=1         n=1     n=1

L = 2*pi*( N*r + d*( (N*(N+1))/2 - N ) )  =  2*pi*( N*r + d*( (N*(N+1))/2 - 2*N/2 ) )

L = 2*pi*( N*r + d*( (N*(N+1) - 2*N)/2 ) )  =  2*pi*( N*r + d*( (N*(N+1 - 2))/2 ) )

L = 2*pi*(N*r + d*(N*(N-1))/2)  =  pi*(2*r*N + d*N*(N-1))  =  pi*(2*r*N + d*N^2 - d*N)
    ------------(1)-----------

L = pi*(d*N^2 + 2*r*N - d*N)
    ----------(2)-----------
```

Because ```L = v*t```, substitutïng ```v*t``` for ```L``` makes it possible to rearrange
this to a quadratic equation in the form of ```a*x^2 + b*x + c = 0```:

```
v*t = pi*(d*N^2 + 2*r*N - d*N)  =  pi*(d*N^2 + (2r-d)*N)  =  pi*d*N^2 + pi*(2r-d)*N
      ----------(2)-----------

v*t = pi*d*N^2 + 2*pi*(r-d/2)*N  =>  pi*d*N^2 + 2*pi*(r-d/2)*N - v*t = 0
                                     ----------------(3)----------------
```

 Now this can be solved for ```N``` using the quadratic formula;
 ```x = (-b +/- sqrt(b^2 - 4*a*c)) / 2*a```, which gives:
            
```
N = (-2*pi*(r-d/2) +/- sqrt((2*pi*(r-d/2))^2 + 4*pi*d*v*t)) / 2*pi*d
```

Because the counter counts up as the tape moves forward, only the positive solution is considered:

```
N = (-2*pi*(r-d/2) + sqrt((2*pi*(r-d/2))^2 + 4*pi*d*v*t)) / 2*pi*d
```

Rearranging and simplifying:
```
N = (-2*pi*(r-d/2) + sqrt((2*pi*(r-d/2))^2 + 4*pi*d*v*t)) / 2*pi*d

N = (-2*pi*(r-d/2) + sqrt(4*pi^2*(r-d/2)^2 + 4*pi^2*d*v*t/pi)) / 2*pi*d

N = (-2*pi*(r-d/2) + 2*pi*sqrt((r-d/2)^2 + (d*v*t)/pi)) / 2*pi*d

N = (-(r-d/2) + sqrt((r-d/2)^2 + (d*v*t)/pi)) / d

N = -(r-d/2)/d + sqrt((r-d/2)^2 + (d*v*t)/pi)/d

N = -(1/d)*(r-d/2) + (1/d)*sqrt((r-d/2)^2 + (d*v*t)/pi)

N = -(1/d)*(r-d/2) + sqrt((1/d)^2*((r-d/2)^2 + (d*v*t)/pi))

N = -(1/d)*(r-d/2) + sqrt((1/d)^2*((r-d/2)^2 + (d*v*t)/pi))

N = -(1/d)*(r-d/2) + sqrt((1/d)^2*(r-d/2)^2 + (1/d)^2*(d*v*t)/pi)

N = -(r/d-1/2) + sqrt((r/d-1/2)^2 + (v*t)/(d*pi))

N = -(r/d-1/2) + sqrt((r/d-1/2)^2 + (v/(d*pi))*t)

N = sqrt((r/d-1/2)^2 + (v/(d*pi))*t) - (r/d-1/2)
----------------------(4)-----------------------
```

This is the final formula to calculate how many turns the pickup spool of the cassette has made at time ```t```;
```
N = sqrt((r/d-1/2)^2 + (v/(d*pi))*t) - (r/d-1/2)
----------------------(4)-----------------------
```

Calculating this involves a lot of operations, but as ```r```, ```d```, and ```v``` are predefined constants,
parts of the calculation can be prepared beforehand, namely all the sub expressions except ```t``` itself:

```
C1 = (r/d-1/2)^2
C2 = v/(d*pi)
C3 = (r/d-1/2)
```

Which leaves the final calculation for how many revolutions the tape spool has made at time ```t``` as:
```
N = sqrt(C1+C2*t)-C3
--------(5)---------
```

To get the actual counter value, the belt/gear ratio must be multiplied in:
```
C = f*(sqrt(C1+C2*t)-C3)
----------(6)-----------
```

The time t will be calculated as the number of cycles total passed under the tape head,
divided by the cycles per seconds of the tape image, which is part of the TAP format.


**DONE!**

### Alternate method

There is an alternate method of deriving the calculation, starting with what seems
to be a standard formula for the length of tape that a reel hub with diameter *d*
using tape of thickness *t* will accumulate in *n* turns starting from zero turns:
(https://www.tapeheads.net/threads/cassette-tape-thickness.52351/ in post #8 by Velktron)

```
L(n) = pi*(d*n +t*n^2)
```

Sustituting for the constants ```r``` (empty spool radius), ```d``` (tape thickness), and ```v``` (tape speed),
as used above, it becomes:
```
L = pi*(2*r*N +d*n^2)
```

That is very similar to the length formula ```(2)``` derived above, it is just missing the last part ```-d*N```,
probably because the tape thickness is very small so d*N will also be very small affecting the result very little.

Substituting ```L``` with ```v*t``` in this simpler formula, rearranging to a quadratic equation, and
solving for ```N``` using the quadratic formula, also yields a very similar end result:
```
N = sqrt((r/d)^2 + (v/(d*pi))*t) - (r/d)
```

Noting that the difference is only that it now uses ```(r/d)``` instead of ```(r/d - 1/2)```,
and this will only yield a very small difference; indeed it will reduce down to the same
final calculation ```(5)```; the constants ```C1``` and ```C3``` will only be slightly different:

```
C1 = (r/d)^2
C2 = v/(d*pi)
C3 = (r/d)
```