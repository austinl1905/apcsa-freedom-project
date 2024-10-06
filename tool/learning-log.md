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

HOWEVER. GRomacks by default only has residues predefined for certain molecules (since its mostly meant for biochemical systems). A residue pretty much just tells you what kind of molecule is in your file. A residue number represents that atoms are apart of the same molecule. This way you can have multiple molecules in the same file and gromacs can recognize them. You need to manually add your own custom residue to the rtp file for the specific force field that you want to use (which is a long and annoying process). Only then you can update the topology and begin energy minimization
<!-- 
* Links you used today (websites, videos, etc)
* Things you tried, progress you made, etc
* Challenges, a-ha moments, etc
* Questions you still have
* What you're going to try next
-->
