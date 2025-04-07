# Tool Learning Log

## Tool: **GROMACS**

## Project: **Chemistry app thingy**

---

### 10/6/24:
* I've actually been using this tool a bit before I even started my project. so that's cool I guess
so gromacs is command line based. I think I'll end up learning Bash at some point to make this all easier.
The primary workflow of GROMACS is to first create a pdb file. A pdb file is usually meant for describing the dimensions of big proteins (ew biology) but it's suitable for smaller molecules as well. You need to define the type of atom, the residue (getting into that later) and its coordinates (cartesian 3d). Unless you're downloading from the pdb archive though, you can't just pull these numbers out of your ass. you do in fact need to have knowledge on molecular geometries (ap chemistry referece) and calculate the positions based on bond length and angles. Forrrrr example....

Ethane has 8 atoms. 2 carbons that are bonded to each other and 3 hydrogens each. Since there's no lone pairs that's two tetrahedral geometries which has a bond angle of 109.5 degrees (It might be different in experimental data but idk how to calculate that lolll). Do the math and boom you got the coordinates:

```
HETATM    1  C1  ETH A   1       0.000   0.000   0.000  1.00  0.00           C 0
HETATM    2  C2  ETH A   1       1.080   1.080   1.080  1.00  0.00           C 0
HETATM    3  H1  ETH A   1       0.771  -0.771  -0.771  1.00  0.00           H 0
HETATM    4  H2  ETH A   1      -0.771   0.771  -0.771  1.00  0.00           H 0
HETATM    5  H3  ETH A   1      -0.771  -0.771   0.771  1.00  0.00           H 0
HETATM    6  H4  ETH A   1       1.860   1.860   0.318  1.00  0.00           H 0
HETATM    7  H5  ETH A   1       1.860   0.318   1.860  1.00  0.00           H 0
HETATM    8  H6  ETH A   0       0.318   1.860   1.860  1.00  0.00           H 0
TER       8      ETH A   0                                                      
END              
```
not going into the syntax of the file. whitespace is annoying and i just end up using a specialized editor for it. 

So thats step 1. next you have to generate a topology for that. That's what GROMACS is for though. With the following command:

`gmx pdb2gmx -f ethane.pdb -o ethane_processed.gro -p topol.top`

so what exactly is a gro and top file? Well gro is just another file format that represents molecules in a similar way to pdb. basically the same thing and it even works with visual representations such as VMD. The main point of the command though is the topology. The topology includes lots of important information about the molecule (in a not so readible format) such as the mass, residue number (later), angles, and dihedrals (I honestly have no idea what those are LMAO)

HOWEVER. GRomacks by default only has residues predefined for certain molecules (since its mostly meant for biochemical systems). A residue pretty much just tells you what kind of molecule is in your file. A residue number represents that atoms are apart of the same molecule. This way you can have multiple molecules in the same file and gromacs can recognize them. You need to manually add your own custom residue to the rtp file for the specific force field that you want to use (which is a long and annoying process). Only then you can update the topology and begin energy minimization.

### 10/14/24

Finding resources for learning this tool is really hard.

(https://www.compchems.com/gromacs-file-formats-understanding-topology-itp-and-gro-files/#gro-file-format)[https://www.compchems.com/gromacs-file-formats-understanding-topology-itp-and-gro-files/#gro-file-format]

In order to actually add the custom residue you have to modify some files in the build. Each forcefield has their own set of files. I'm using opls-aa because it's good for inorganic molcules i think. So in order to access that forcefield in particular you use the following path: `/usr/local/gromacs/share/gromacs/top/oplsaa.ff`. From here there's a bunch of different files in the `itp`, `dat`, and `rtp` formats. According to the gromacs manual, `itp` files are a way to store information for a molecule that's used often, such as water:

```[ moleculetype ]
; molname nrexcl
SOL       2

[ atoms ]
;   nr   type  resnr residue  atom   cgnr     charge       mass
     1  opls_116   1    SOL     OW      1      -0.8476
     2  opls_117   1    SOL    HW1      1       0.4238
     3  opls_117   1    SOL    HW2      1       0.4238

#ifndef FLEXIBLE
[ settles ]
; OW    funct   doh     dhh
1       1 0.1   0.16330

[ exclusions ]
1       2 3
2       1 3
3       1 2
#else
[ bonds ]
; i     j funct length  force.c.
1       2 1     0.1     345000  0.1     345000
1       3 1     0.1     345000  0.1     345000

[ angles ]
; i     j k     funct   angle   force.c.
2       1 3     1       109.47  383     109.47  383
#endif
```

Q: Uhhh I don't see water anywhere in this file
A: Water in gromacs have different types, such as spce, spc, tip3p, etc. I don't really know why. It's probably something to do with the accuracy of the calculations and the work required. That or computational chemists are just very pretentious people. But whatever you do, whatever molecule you use with the `pdb2gmx` command must also be the same one used to construct the topology and solvate the system (otherwise you get really annoying errors about how the coordinates don't match up).

Back to residues. First off add your residue name to `residuetypes.dat`: `ETH     Other`

Then add it to `aminoacids.rtp`. No, ethane is not an amino acid. It's just where the residues are:
```
[ ETH ]
 [ atoms ]
    C1     opls_135     0.000  0
    C2     opls_135     0.000  1
    H1     opls_140     0.000  0
    H2     opls_140     0.000  0
    H3     opls_140     0.000  0
    H4     opls_140     0.000  1
    H5     opls_140     0.000  1
    H6     opls_140     0.000  1
 [ bonds ]
   C1    C2
   C1    H1
   C1    H2
   C1    H3
   C2    H4
   C2    H5
   C2    H6
```
Based on the itp file for a simple molecule, it would obviously be pretty infeasible to do the same for an ethane molecule (because how am I supposed to know the exact partial charges?). So therefore you get GROMACS to do some of the work for you by defining the bonds and atoms. Notice `olps_135` and `opls_140` next to the atom name? Those are also important because the information for an atom is different depending on its bond and the group it belongs to (such as methyl or hydroxyl)

Normally you do the same for `aminoacids.hdb` (hydrogen data base). But as you can see I already have my hydrogens defined in the residue so that's unecessary.

So now I can successfully perform `pdb2gmx` because GROMACS is no longer trying to look for a residue that doesn't exist! Yipee!

It's not magic. Turns out there's a lot of extra work you have to do by yourself.

### 10/20/24

and btw these models can be visualized with vmd and stuff but im too lazy to attach images hahahahaha

ok cool i have the residue now. so there's a few neat things you can do with the gro file.

First you can pretty much clone the molecule that's inside of it with the `insert-molecules` command:

`gmx insert-molecules -ci ethane.gro -o ethane_box.gro -nmol 10-box 3 3 3`

You should probably do this before generating the topology though.

sooo what this basically does is insert 10 of the same molecules in the gro file into a 3 by 3 by 3 dimension. 3 what? bananas? idk either

water is a solvent. you probably knew that. but nobody wants to insert a bunch of water molecules manually. that's why you can do this:

`gmx solvate -cp ethane_box.gro -cs spc216.gro -p topol.top`

so yeah just shove a bunch of spc216 water molecules into the box we just made yay thats it

### 11/11/24

before i do anything with this cool system of ethane and water i have to run energy minimization. so pretty much get the molecules into their ideal positions so that no unexpected results come from further analysis.

First a `min.mdp` file. Idk what each parameter does I just include the basics

```
integrator = steep
nsteps = 50000
nstxout = 100
```

According to this tutorial I read steep just means move molecules according to their forces until an energy criteria is reached. Seems about right to me

Then I gotta run this command which does... something idk

`gmx grompp -f min.mdp -c ethane_box.gro -p topol.top -o min -pp min -po min`

Then run the mdrun

`gmx mdrun -v -deffnm min`

And now this is where my cool chemistry knowledge comes in because otherwise you wouldn't know if the results were fine or not. GROMACS tells u the max force and potential energy. If the potential energy is large and negative and the max force is also low, then it's fine because that means the forces are attractive and the distances between the molecules are as if the system is at equilibrium. woah.

And also gromacs gives you a trr file to watch the cool animation on vmd

### 11/17/24

Also gromacs gives you an xvg file with `gmx energy`. I downloaded xmgrace on the cli to see the file. It's pretty much just a graph of potential energy vs time (steps).

The first equilibration to be performed is NVT. Constant moles (n), volume, and temperature. The point of this is to optimize the solute (ethane) with the solvent (water), bring the system to a desired temperature (usually high like 500 K), and increase the pressure until the system reaches the correct density of solute and solvent. I'm using the following file for this:

```
coulombtype = pme
fourierspacing = 0.1
pme-order = 4
rcoulomb = 1.0
vdw-type = Cut-off
rvdw = 1.0
constraint-algorithm = lincs
constraints = hbonds
continuation = no
tcoupl = v-rescale
tc-grps = system
tau-t = 0.5 0.5
ref-t = 360 360
cutoff-scheme = Verlet
nstlist = 10
ns_type = grid
gen-vel = yes
gen-temp = 360
comm_mode = linear
comm_grps = system
```

Yeah idk what most of these parameters mean I'll figure it out.

From the xvg file I created it looks like the system was not given enough time to stabilize the temperature. So I'll have to try again later.

### 11/24/24 

I screwed up somewhere because I accidentally used thewrong file for an input. I gotta have a better way of doing this. 

Luckily, I don't need to start over completely. I already have my solvated system and residues, and generating the topology is as easy as a single command. oh yeah my property of interest is the interaction between nonpolar and polar molecules. I mean I'm already supposed to know what happens, but I'm doing this experiment to make sure I'm actually using gromacs properly.

And oh yeah there's another way to create your "system". I used "insert-molecules" which is random, but you can also use "genconf", which will control where your molecules go and how many of them in a specific area (density). It's slower for minimization and equilibriation, but the complexity of my system isn't super high right now so it works great. Additionally it's easy to see the difference between the pre min and min files.

When I use this command:

`gmx genconf -f ethane.pdb -nbox 5 5 5 -o ethane_grid.pdb`

I get a grid of 125 ethane molecules, but they're all in the same position, which isn't what I want. So I tried modifying the -dist argument:
`gmx genconf -f ethane.pdb -nbox 5 5 5 -dist 1 1 1 -o ethane_grid.pdb`

Now they're in a proper grid, but for some reason the carbon-carbon bond is missing. :/

### 12/8/24

Restarted my simulations. I have to figure out how to solvate my system without the random positioning. Perhaps theres some sort of way to combine different coordinate files

I mean, it's probably as simple as just copying and pasting the coordinates of the other file. Where the problem arises is making sure my "water" has the correct residue and model

I should probably learn more about the actual math that goes on when I run gromacs, since I'm planning to modify it (or adapt it) for my own personal use anyways.

For my "experiment", I'm using non-bonded (which really means non-covalent) interactions between uncharged molecules. Which are way easier to deal with, by the way. Bonded interactions would mean doing stuff with dihedrals and bond stretching potentials (I still don't know what those are)

Intermolecular interactions involve repulsive forces and dispersive (attractive) forces. Attractive forces includes the basic dipole-dipole, induced dipole, and LDFs. Repulsive forces include electron-electron interactions (Pauli exclusion principle. It's complicated.) Both of them can be combined into a single equation used to calculate the Lennard-Jones potential (in kj/mol): $$V(r)=4ϵ((\frac{σ}{r})^{12}−(\frac{σ}{r})^{6})$$

Which produces a graph that looks very similar to a potential energy curve you might see in your chemistry class. At distance σ, the potential energy is at 0 (equal to the radius of the molecule). At the minimum potential energy (well depth) is the energy of the non-bonded interaction at equilibrium. A more negative well depth equates to a stronger interaction.

The first term represents the repulsive forces at very short distances, while the second term represents the attractive forces at intermdiate distances.

So that's what GROMACS is doing to make my molecule positions look nice and pretty. Cool.

There's also Buckingham potential, which serves the same purpose but is more computationally expensive.

### 1/5/25

Having finally finished npt, I made the graph for both pressure and density (which are both related obviously)

At first I was pretty confused because the pressure and density wildly fluctuated. I thought these were both supposed to stabilize bc it was, well, equilibrium. Normally in npt you expect a deviation in around the tens, but at certain points I was seeing 1 bar + up to 500 bars, so something was definitely wrong. I looked at the graphs for temperature, which was supposed to be constant. But it also fluctuates (yet the average seemed to be around 300K, which was expected). Nothing weird seemed to be happening with potential energy either. So I came to the conclusion these deviations were due to my relatively small system. It made sense, because in reality systems are much larger, but I only had a few thousand molecules (very little on a macro scale). So I'm just going to move on with the analysis, and in the worst case I'll just restart NPT. 

### 1/12/25

At this point I've realized my lack of motivation is probably due to the fact I haven't been coding. Like at all. So now I am. I'm going to use Fortran for writing coordinate files and analyzing data anwyays, so I gotta learn the basics, especially IO. And what better way to do so than implement my favorite sorting algorithm?

So, I already know some fortran, but I'm not quite proficient. So doing this involved a lot of researching documentation (and admittedly a lot of chatgpt, but only for conceptual errors, I promise). Here's the runner code I got:
```fortran
PROGRAM MAIN
    USE MERGE_SORT
    IMPLICIT NONE
    INTEGER, ALLOCATABLE :: ARR(:)

    ALLOCATE(ARR(5))
    ARR = [2, 7, 9, 0, -3]
    PRINT *, SORT(ARR)

END PROGRAM MAIN
```

It looks very stupid and ugly, because it is. IDK how to make it better. Also, I'm using SCREAMING_CAMEL_CASE because it makes me feel like those pioneering data scientists working on those really big computers

The reason I made the array "allocatable" just to allocate space and give it elements is because my sort() function only takes in allocatable arrays. Arrays in fortran are fixed in size, so I can't just assume the array being passed is a certain size. There's probably a better way I don't know though.

And then here's the actual thing:

```fortran
MODULE MERGE_SORT
    IMPLICIT NONE
CONTAINS
    RECURSIVE FUNCTION SORT(ARR) RESULT(RES)
        INTEGER, DIMENSION(:), ALLOCATABLE, INTENT(IN) :: ARR
        INTEGER, DIMENSION(:), ALLOCATABLE :: LEFT, RIGHT, RES
        INTEGER :: MIDPOINT

        IF (SIZE(ARR).LE.1) THEN ! BASE CASE
            RES = ARR
            RETURN
        END IF
        
        MIDPOINT = SIZE(ARR) / 2
        ! ASSIGN SUBARRAYS
        LEFT = ARR(1:MIDPOINT)
        RIGHT = ARR(MIDPOINT + 1:SIZE(ARR))

        ! GO BACK UP CALL STACK ONCE SIZE OF SUBARRAYS REACHES 1
        LEFT = SORT(LEFT)
        RIGHT = SORT(RIGHT)

        ! MERGE ARRAYS IN ASCENDING ORDER AND GO UP CALL STACK AGAIN
        RES = MERGE_ARRAYS(LEFT, RIGHT)
        RETURN
    END FUNCTION

    FUNCTION MERGE_ARRAYS(LEFT, RIGHT) RESULT(RES)
        INTEGER, DIMENSION(:), ALLOCATABLE :: LEFT, RIGHT, RES
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

Wow. At this point I know how to write merge sort in three different programming languages, so I'll just let the code speak for itself and explain some of the quirks of the language. 

Recursive functions need to be explicitly declared as recursive. For simpler data types such as integers, the return type can be included before the function name, like in other programming languages. But for some reason you can't do that with arrays. So in the function header, you need an additional result() variable.

Also, the type of a variable is not determined in the parameters. it's declared in the function body and initialized separately. No such method or property that returns size exists for arrays: you must use an intrinsic function instead. Why? I don't know either. It's confusing too, because Fortran supports OOP.

Splicing arrays is one of the more straightforward features of fortran. For example, ARR(1, 4) returns an array from index 1 to index 4, inclusive. Oh, arrays also use 1-based indexing by default, but the lower bound for an array can be designated as 0 if you wish. So one small thing I had to remember is that iterators should start at 1 and the condition to stop should be <=, not <. Also, no compound assignment or increment/decrement operators. HUH????

The good thing about already knowing one programming language is that all I really need to do is figure out the syntax (however, fortran does some things very differently from other languages). Compare this to my java implementation:

```java
public static void sortTransactionsByAmount(ArrayList<Transaction> transactions)
{   if (transactions.size() <= 1) // To prevent sorting arrays further than necessary
   {   return;   } // Returns to parent call, not the first call.

   int midpoint = transactions.size() / 2;
   // subList() returns List, so I need to wrap it
   ArrayList<Transaction> leftArray = new ArrayList<>(transactions.subList(0, midpoint));
   ArrayList<Transaction> rightArray = new ArrayList<>(transactions.subList(midpoint, transactions.size()));

   // Recursively divide subarrays
   sortTransactionsByAmount(leftArray);
   sortTransactionsByAmount(rightArray);

   // Pass subarrays "up" the call stack to be merged
   ArrayList<Transaction> sortedTransactions = mergeTransactionsByAmount(leftArray, rightArray);
   transactions.clear();
   transactions.addAll(sortedTransactions); // Since merge sort doesn't work in place, the sorted array has to be copied to the original.
}

private static ArrayList<Transaction> mergeTransactionsByAmount(ArrayList<Transaction> leftArray, ArrayList<Transaction> rightArray)
{   ArrayList<Transaction> transactions = new ArrayList<Transaction>();
   int leftIndex = 0;
   int rightIndex = 0;

   while (leftIndex < leftArray.size() && rightIndex < rightArray.size())
   {   if(leftArray.get(leftIndex).getAmount() < rightArray.get(rightIndex).getAmount())
       {   transactions.add(leftArray.get(leftIndex));
           leftIndex++;
       } else 
       {   transactions.add(rightArray.get(rightIndex));
           rightIndex++;
       }
   }

   while (leftIndex < leftArray.size())
   {   transactions.add(leftArray.get(leftIndex));
       leftIndex++;
   }

   while (rightIndex < rightArray.size())
   {   transactions.add(rightArray.get(rightIndex));
       rightIndex++;
   }

   return transactions;
}
```
Cool stuff! File I/O next. maybe. if I can figure out how to actually use this language I think It will pay off.

3/2/25

im making the md sim all by myself. because i want to CODE

anyways i kind of got the hang of io. You have to read the file and then you can parse it into lines and stuff. Then you can use fortran formatting to get the information you want

```fortran
PROGRAM INOUT
    IMPLICIT NONE
    REAL :: X, Y, Z
    INTEGER :: LINE_NUM, I
    CHARACTER(LEN = 80) LINE ! PDB FILES ARE NO MORE THAN 80 COLUMNS PER LINE

    LINE_NUM = 4
    
    OPEN(UNIT=10, FILE='ammonia.pdb', STATUS='OLD', ACTION='READ')

    DO I = 1, LINE_NUM
        READ(10, '(A)') LINE ! READ ENTIRE LINE
    

        READ(LINE(31:38), '(F8.3)') X ! EXPECT 3 FP DIGITS
        READ(LINE(39:46), '(F8.3)') Y ! EXPECT 3 FP DIGITS
        READ(LINE(47:54), '(F8.3)') Z ! EXPECT 3 FP DIGITS

        PRINT *, X, Y, Z
    END DO

    CLOSE(10)

END PROGRAM INOUT
```

this just reads the first line though. If you want to read all of the lines you can do this

```
OPEN(UNIT=10, FILE=F, IOSTAT=IOS, STATUS='OLD', ACTION='READ')

DO
 READ(10, '(A)', IOSTAT=IOS) LINE
 IF (IOS.NE.0) THEN
     EXIT
 END IF
 READ(LINE(1:6), '(A6)') STR
 IF (STR.EQ.'ATOM  '.OR.STR.EQ.'HETATM') THEN
     N = N + 1
 END IF
END DO

ALLOCATE(C(N))
REWIND(10)

DO I = 1, N
 READ(10, '(A)', IOSTAT=IOS) LINE
 READ(LINE(31:38), '(F8.3)') C(I)%X ! EXPECT 3 FP DIGITS
 READ(LINE(39:46), '(F8.3)') C(I)%Y ! EXPECT 3 FP DIGITS
 READ(LINE(47:54), '(F8.3)') C(I)%Z ! EXPECT 3 FP DIGITS
 PRINT *, C(I)%X
 PRINT *, C(I)%Y
 PRINT *, C(I)%Z
END DO
```

great stuff! lennard-jones potential is next maybe

### 3/23/25

I tried to add this code for reflecting boundary conditions.doesn't quite work though

```
SUBROUTINE REFLECT(R, V)
   REAL, DIMENSION(N, D) :: R, V
   INTEGER :: COL, ROW
       DO COL = 0, SIZE(R, DIM = 2)
           DO ROW = 0, SIZE(R, DIM = 1)
               IF (R(ROW, COL).LT.0) THEN 
                   V(ROW,COL) = -V(ROW, COL)
                   R(ROW, COL) = -R(ROW, COL)
               END IF
               IF (R(ROW, COL).GT.100) THEN
                   V(ROW, COL) = -V(ROW, COL)
                   R(ROW, COL) = (2.0 * 100) - R(ROW, COL)
               END IF
           END DO
       END DO
   RETURN
END
```

When a particle reaches a boundary it sort of just floats around there. I gotta figure that out. it seems like the velocity is staying the same but the position goes past 0 or 100.

### 4/1/25

Added a dump for my simulation:

```fortran
SUBROUTINE DUMP(R, T, I)
   CHARACTER(LEN = 20) :: FILENAME
   REAL, DIMENSION(N, D) :: R
   REAL :: T
   INTEGER :: I, J, K

   WRITE(FILENAME, '(A, I0, A)') 'dump/data', I, '.txt'

   OPEN(1, FILE=FILENAME, STATUS = 'new')

   WRITE(1, *) "ITEM: STEP"
   WRITE(1, '(I10)') I
   WRITE(1, *) "ITEM: DT"
   WRITE(1, '(F5.2)') DT
   WRITE(1, *) "ITEM: TIME"
   WRITE(1, '(F5.2)') T
   WRITE(1, *) "ITEM: N ATOMS"
   WRITE(1, '(I10)') N
   WRITE(1, *) "ITEM: BOX CONDITION"

   IF (BC) THEN 
       WRITE(1, *) "PERIODIC"
   ELSE 
       WRITE(1, *) "REFLECTIVE"
   END IF

   WRITE(1, *) "ITEM: BOX BOUNDS (CUBE)"
   WRITE(1, '(F5.3, A, F7.3)') 0.00, " ", L

   WRITE(1, *) "ITEM: X Y Z"
   
   DO J = 1, N
       WRITE(1, '(F10.6, F10.6, F10.6)') R(J, 1), R(J, 2), R(J, 3)
   END DO

   CLOSE(1)
END
```

It outputs data into a txt file for each step of the simulation:
```
 ITEM: STEP
        17
 ITEM: DT
 0.01
 ITEM: TIME
 0.17
 ITEM: N ATOMS
         5
 ITEM: BOX CONDITION
 REFLECTIVE
 ITEM: BOX BOUNDS (CUBE)
0.000 100.000
 ITEM: X Y Z
 54.986469 26.970545 23.766037
 65.483910 59.547302 98.639061
 79.378143 76.269287 32.916183
 69.127960 20.604038 81.038292
  7.952033 20.709009 53.371666
```
I can then use this data for when I make my 3D engine later.

### 4/6/25

progress was made check this out

```fortran
SUBROUTINE REFLECT(R, V)
   REAL, DIMENSION(N, D) :: R, V
   INTEGER :: COL, ROW
       DO COL = 0, SIZE(R, DIM = 2)
           DO ROW = 0, SIZE(R, DIM = 1)
               IF (R(ROW, COL).LT.0) THEN 
                   V(ROW,COL) = -V(ROW, COL)
                   R(ROW, COL) = -R(ROW, COL) ! REFLECT POSITION ABOUT 0
               END IF
               IF (R(ROW, COL).GT.L) THEN
                   V(ROW, COL) = -V(ROW, COL)
                   R(ROW, COL) = (2.0 * L) - R(ROW, COL) ! REFLECT POSITION ABOUT L
               END IF
           END DO
       END DO
   RETURN
END
```

This is the WORKING code for my RBC. As it turns out fortran does weird stuff when you call functions from subroutines and whatnot. I figured that since subroutines are essentially "void functions" it would be easier to do this since I have defined a velocity and position array. But for potential energy I can still use a function like so:

```fortran
FUNCTION LJPOT(R, A) RESULT(LJP) ! SUM OF POTENTIAL ENERGY BETWEEN MOLECULES
   REAL, DIMENSION(N, D) :: R, DRV ! POSITION, POTENTIAL ENERGY, DISPLACEMENT VECTOR (FOR EACH DIMENSION AND ATOM)
   REAL, DIMENSION(N - 1, D) :: NDRV ! DISPLACEMENT? VECTOR (WITHOUT ITH ATOM)
   REAL, DIMENSION(N - 1) :: DR ! DISTANCE BETWEEN NTH ATOM AND ITH ATOM
   REAL :: LJP
   INTEGER :: A, NEW_ROW, I

   NEW_ROW = 1
   LJP = 0

   DO I = 1, N ! CALCULATE COMPONENT WISE DISTANCE VECTORS. EX: IF ATOM K HAS POSITION VECTOR (1, 2, 3) AND ATOM L HAS POSITION VECTOR (5, 4, 2) THEN THE RESULTING VECTOR IS (4, 2, -1). THIS VECTOR CAN THEN BE USED FOR EUCLIDEAN DISTANCE CALCULATION.
       DRV(I, :) = R(I, :) - R(A, :)
   END DO

   DO I = 1, N 
       IF (I.EQ.A) THEN ! EXCLUDE ITH ATOM
           CYCLE
       END IF
       NDRV(NEW_ROW, :) = DRV(I, :)
       NEW_ROW = NEW_ROW + 1
   END DO

   DO I = 1, N - 1 ! CALCULATE EUCLIDEAN DISTANCE FOR EACH ATOM (ASSUMING D = 3)
       DR(I) = SQRT(NDRV(I, 1)**2 + NDRV(I, 2)**2 + NDRV(I, 3)**2)
   END DO

   DO I = N, N - 1 ! CALCULATE LENNARD-JONES POTENTIAL
       LJP = LJP + (4.00 * EPS * (((SIG/DR(I))**12)-((SIG/DR(I))**6))) ! REPLACE WITH SUM FUNCTION LATER
   END DO

RETURN
END
```

Right now it returns 0 no matter what but trust the vision (im pretty sure its the last part I need to fix)

$$U(r) = 4\\epsilon [(\\frac{\\sigma}{r})^{12} - (\\frac{\\sigma}{r})^{6}]$$

This is the lennard jones potential. it's pairwise (meaning just between two atoms) but in a real simulation I need ALL of the interactions between every atom. Thus in the function I tried to make it a sum of those potential energy calculations.

Basically, I needed to find the displacement vectors in order to calculate the distance (a scalar quantity). Also it's important to exclude the atom being passed (as the distance would just be 0) from there I just plug the numbers into the formula! Next is the derivative of lennard jones potential, the derivative of energy with respect to distance aka FORCE!!!
