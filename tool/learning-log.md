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

For my "experiment", I'm using non-bonded (intermolecular) interactions between uncharged molecules. Which are way easier to deal with, by the way. Bonded interactions would mean doing stuff with dihedrals and bond stretching potentials (I still don't know what those are)

Intermolecular interactions involve repulsive forces and dispersive (attractive) forces. Attractive forces includes the basic dipole-dipole, induced dipole, and LDFs. Repulsive forces include electron-electron interactions (Pauli exclusion principle. It's complicated.) Both of them can be combined into a single equation used to calculate the Lennard-Jones potential (in kj/mol). $$V(r)=4ϵ[(σr)12−(σr)6]$$

