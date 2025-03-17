# Entry 4
##### 3/16/25

The project has been stagnant for pretty much the last 3 months, but I'm actually starting to make some good progress now. I've moved on to coding the MD from scratch, and I found actually a [tutorial](https://www.youtube.com/watch?v=ChQbBqndwIA) on how to do it. It's in Python, but I only need it to understand the concepts of MD so I can code it in Fortran instead.

Here's the important relationship between variables that I had to understand:

A molecule has position $r = (x, y, z)$, which is a function of time $t$, usually in femtoseconds (fs). This position is randomly generated on the bounds $[0, 100)$ with the following code. Fortran allows you to perform operations and function calls on arrays like so:
```fortran
REAL, DIMENSION(N, D) :: R
CALL RANDOM_NUMBER(R)
R = R * 100
```
N being the number of molecules and D being the number of dimensions (this will be a 3D simulation, so it is 3)

These particles are in motion, and at a value of $t$ for $t>0$, the position of the particle $r(t) = (x_t, y_t, z_t)$. The rate at which the position changes is represented as $\frac{dr}{dt}$, the velocity. This variable also has to be randomly generated (as it wouldn't make sense to generate positions for desired values of $t$ before calculating it):

```fortran
REAL, DIMENSION(N, D) :: V
V = (V * 200) - 100
```

The bounds of V are $[-100, 100)$. I'll deal with the acceleration later, so this can stay constant.

In order to calculate the position at $t$, the following definition can be used:

$r(t) = r_0 + v_0t$

This does NOT account for acceleration. It's coming soon.

In a simulation, the molecules would be constantly moving, and every unit is too high of a step to be updating the position, believe it or not. So a smaller one must be used. In this case, I made the change in time ($dt$) 0.01.

```fortran
FUNCTION UPDATE(R, V, DT) RESULT(NEWR)
        REAL, DIMENSION(N, D), INTENT(IN) :: R, V
        REAL, INTENT(IN) :: DT
        REAL, DIMENSION(N, D):: NEWR
        NEWR = R + V * DT
        RETURN
END FUNCTION
```
```fortran
DO I = 0, 10
        PRINT '(A, F10.6)', "POSITION AT T = ", DT * I
        PRINT *, R
        R = UPDATE(R, V, DT)
END DO
```
At the moment, I don't have a great way to model this graphically. There are some CLI tools that exist but I'll figure that out later.

In the meantime, there's another problem with this. It can be assumed that since the bounds of the positions are $[0, 100)$, that would also be the bounds of the simulation. However, since the molecules are continuously moving in one direction under these conditions, eventually the molecules will leave these bounds.

The solution to this is implementing a [**periodic boundary condition**](https://www.compchems.com/molecular-dynamics-periodic-boundary-conditions-pbc/#simulation-of-bulk-systems). In a bulk system of molecules, it can be assumed that for any given 3D space, the spaces adjacent and beyond have translational symmetry (identical coordinate values). This way, the illusion of an infinite simulation can be created without using excessive computational resources. But for now, I'm just going to recreate this with a single "box".

First, I explicitly defined a bound for the box as a parameter. Then
a simple mod function can make sure the molecules stay within those defined bounds:

```fortran
FUNCTION UPDATE(R, V, DT) RESULT(NEWR)
        REAL, DIMENSION(N, D), INTENT(IN) :: R, V
        REAL, INTENT(IN) :: DT
        REAL, DIMENSION(N, D):: NEWR
        
        IF (BC) THEN
            NEWR = MODULO(R + V * DT, L) ! L = 100
        ELSE 
            NEWR = R + V * DT
        END IF
        RETURN
    END FUNCTION
```
`%` does not serve the same purpose in Fortran as it does in other languages. It's actually used in OOP for accessing instance variables.

So why does this work? If the intention is for the molecules to be within [0, 100] for x, y, and z, any coordinate value mod 100 will give the same coordinate. Once this value is outside that range, the "progress" of that molecule's movement will get shifted as the new value will be the remainder. As a example, if the initial position of a molecule was $r=(20.000, 89.000, 60.000)$ and its velocity was $v=(-57.000, 10.000, 63.000)$, its position at $t=1$ will be $r=(63.000, 99.000, 23.000)$.

Another way to solve this problem is by using **reflective boundary conditions**, which pretty much means the molecules will bounce off the "walls". I'll do that later.

I'm at step 5 of the Engineering Design Process. At this point I'm starting to create the foundation for my project so that I can go forward with the more complex features later on.

I'm using the skills problem decomposition and how to learn. I'm working out the basics of MD so that eventually I will be able to put together what I've created into something cohesive. Additionally, I can't follow tutorials directly because they are written in different programming languages. Instead, I use the concepts I learn from them and code it myself in Fortran.

[Previous](entry03.md) | [Next](entry05.md)

[Home](../README.md)
