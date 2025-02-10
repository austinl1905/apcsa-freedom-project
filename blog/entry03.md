# Entry 3
##### X/X/XX

My two goals were to get acquainted with the Fortran language and learn how to do file I/O (which is actually the first time I've done this).

I figured the easiest way to do it was by implementing some algorithms. So I started with my favorite: the merge sort. Here's what it looks like: 

```fortran
MODULE MERGE_SORT
    IMPLICIT NONE
CONTAINS
    RECURSIVE FUNCTION M_SORT(ARR) RESULT(RES)
        INTEGER, INTENT(IN) :: ARR(:)
        INTEGER, ALLOCATABLE :: LEFT(:), RIGHT(:), RES(:)
        INTEGER :: MIDPOINT

        IF (SIZE(ARR).LE.1) THEN ! BASE CASE
            RES = ARR
            RETURN
        END IF
        
        MIDPOINT = SIZE(ARR) / 2
        ! ASSIGN SUBARRAYS
        ALLOCATE(LEFT(MIDPOINT))
        ALLOCATE(RIGHT(SIZE(ARR) - MIDPOINT))
        LEFT = ARR(1:MIDPOINT)
        RIGHT = ARR(MIDPOINT + 1:SIZE(ARR))

        ! GO BACK UP CALL STACK ONCE SIZE OF SUBARRAYS REACHES 1
        LEFT = M_SORT(LEFT)
        RIGHT = M_SORT(RIGHT)

        ! MERGE ARRAYS IN ASCENDING ORDER AND GO UP CALL STACK AGAIN
        RES = MERGE_ARRAYS(LEFT, RIGHT)
        RETURN
    END FUNCTION

    FUNCTION MERGE_ARRAYS(LEFT, RIGHT) RESULT(RES)
        INTEGER, INTENT(IN) :: LEFT(:), RIGHT(:)
        INTEGER, ALLOCATABLE :: RES(:)
        INTEGER :: LI, RI, I ! ITERATION VARIABLES FOR SUBARRAYS AND OUTPUT

        LI = 1
        RI = 1
        I = 1

        ALLOCATE(RES(SIZE(LEFT) + SIZE(RIGHT)))

        DO WHILE (LI.LE.SIZE(LEFT).AND.RI.LE.SIZE(RIGHT)) ! ADD SMALLER ELEMENTS TO OUTPUT
            IF (LEFT(LI).LT.RIGHT(RI)) THEN
                RES(I) = LEFT(LI)
                I = I + 1
                LI = LI +  1
            ELSE
                RES(I) = RIGHT(RI)
                I = I + 1
                RI = RI + 1
            END IF
        END DO

        ! ADD REMAINING ELEMENTS TO OUTPUT
        DO WHILE (LI.LE.SIZE(LEFT))
            RES(I) = LEFT(LI)
            I = I + 1
            LI = LI + 1
        END DO

        DO WHILE (RI.LE.SIZE(RIGHT))
            RES(I) = RIGHT(RI)
            I = I + 1
            RI = RI + 1
        END DO
        RETURN
    END FUNCTION
END MODULE MERGE_SORT
```

I learned a bit about the best practice for arrays in Fortran (which are easily one of the most powerful features). Arrays meant to be input in a function can be declared as an assumed size array instead of [allocatable](https://fortran-lang.org/en/learn/best_practices/allocatable_arrays/). This way, any array can be used as an argument, not just allocatable ones. As for the other arrays, the size is dynamic and based on the size of the input array. Therefore it's easier to declare as an allocatable and allocate space to them later. 

Recursive functions need to be explicitly declared as recursive. For simpler data types such as integers, the return type can be included before the function name, like in other programming languages. But for some reason you can't do that with arrays. So in the function header, you need an additional result() variable.

Also, the type of a variable is not determined in the parameters. it's declared in the function body and initialized separately. No such method or property that returns size exists for arrays: you must use an intrinsic function instead. Why? I don't know either. It's confusing too, because Fortran supports OOP.

Splicing arrays is one of the more straightforward features of fortran. For example, ARR(1, 4) returns an array from index 1 to index 4, inclusive on both bounds. Arrays also use 1-based indexing by default, but the bounds can be set to anything. It also uses column-major order. 

Now for [I/O](https://fortran-lang.org/en/learn/best_practices/file_io/). Reading files with fixed format is easier because you can expect where you will find the data in the file. For example, in [PDB files](https://www.cgl.ucsf.edu/chimera/1.5.1/docs/UsersGuide/tutorials/pdbintro.html), the length of each line doesn't exceed more than 80 characters, the x coordinate is found from lines 31 to 38, lines 39 to 46 for y coordinates, and 47 to 54 for x coordinates.

So the file looks like this:

```
HETATM    1  C1  ETH A   1       0.000   0.000   0.000  1.00  0.00           C 0
HETATM    2  C2  ETH A   1       1.080   1.080   1.080  1.00  0.00           C 0
HETATM    3  H1  ETH A   1       0.771  -0.771  -0.771  1.00  0.00           H 0
HETATM    4  H2  ETH A   1      -0.771   0.771  -0.771  1.00  0.00           H 0
HETATM    5  H3  ETH A   1      -0.771  -0.771   0.771  1.00  0.00           H 0
HETATM    6  H4  ETH A   1       1.860   1.860   0.318  1.00  0.00           H 0
HETATM    7  H5  ETH A   1       1.860   0.318   1.860  1.00  0.00           H 0
HETATM    8  H6  ETH A   1       0.318   1.860   1.860  1.00  0.00           H 0
TER       8      ETH A   1                                                      
END                                                                 
```

It's also important to expect floating point values, so that must be specified when reading each line. For just one line, it would look like this:

```fortran
PROGRAM INOUT
    IMPLICIT NONE
    REAL :: X, Y, Z
    CHARACTER(LEN = 80) LINE ! PDB FILES ARE NO MORE THAN 80 COLUMNS PER LINE
    
    OPEN(UNIT=10, FILE='ethane.pdb', STATUS='OLD', ACTION='READ')

    READ(10, '(A)') LINE ! READ ENTIRE LINE
    CLOSE(10)

    READ(LINE(31:38), '(F8.3)') X ! EXPECT 3 FP DIGITS
    READ(LINE(39:46), '(F8.3)') Y ! EXPECT 3 FP DIGITS
    READ(LINE(47:54), '(F8.3)') Z ! EXPECT 3 FP DIGITS

    PRINT *, X, Y, Z

END PROGRAM INOUT
```

And the output is as expected: `   0.000   0.000   0.000  ` (the whitespace isn't showing, just trust me)

For multiple lines or even specific lines, it becomes more complicated. Let's say the target line is 8 (since that's how many atoms are in an ethane molecule). A loop and counter will be necessary to read the data from each line before and including the target:

```fortran
PROGRAM INOUT
    IMPLICIT NONE
    REAL :: X, Y, Z
    INTEGER :: LINE_NUM, I
    CHARACTER(LEN = 80) LINE ! PDB FILES ARE NO MORE THAN 80 COLUMNS PER LINE

    LINE_NUM = 8
    
    OPEN(UNIT=10, FILE='ethane.pdb', STATUS='OLD', ACTION='READ')

    DO I = 1, LINE_NUM
        READ(10, '(A)') LINE ! READ ENTIRE LINE
    

        READ(LINE(31:38), '(F8.3)') X ! EXPECT 3 FP DIGITS
        READ(LINE(39:46), '(F8.3)') Y ! EXPECT 3 FP DIGITS
        READ(LINE(47:54), '(F8.3)') Z ! EXPECT 3 FP DIGITS

        PRINT *, X, Y, Z
    END DO

    PRINT *, X, Y, Z
    CLOSE(10)

END PROGRAM INOUT
```
I have no idea how each line actually gets tracked. But it works how I expected it to, so no complaining.

Of course, in this example the length of the file is known. If it isn't, you would also need to keep track of the `IOSTAT`, which is 0 if there aren't any errors when reading each line.

Currently on step 5 of the engineering design process. I'll finish up with the learning while I create my MVP (which in this case will be using user input to parse these coordinate files through gromacs). And I will have to eventually learn OpenGL in some sort of programming language if I want to make 3d models again.

The skills I'm using are how to read and creativity. Finding modern, useful tutorials and documentation for this is hard. Also, the kind of project I'm working on is pretty niche so I have to find ways to take what resources I can find into something I can actually work with.

[Previous](entry02.md) | [Next](entry04.md)

[Home](../README.md)