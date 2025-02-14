# Plan

## Tool: Gromacs, (maybe CP2K), Fortran, ISO_C_BINDING? C++?
## Product: Chemical molecule simulation

---

## Timeline

#### MVP

- [ ] Use Fortran file parsing to take user input and create .pdb and .gro coordinate files (deadline: 2/19/25)
  - [ ] Write coordinate files for at least 5 types of molecules: tetrahedrals, trigonal planar, ionic compounds, linear, trigonal bipyramidal
- [ ] Create program to parse coordinate files into Gromacs/CP2K and save output files (deadline: 2/22/25)
  - [ ] GROMACS doesn't really support my needs for things such as phases, coordinate mapping for reactions, etc (not to mention I have no idea what I'm doing with it 60% of the time). So I may (definitely) write my own MD simulation in Fortran instead. Throughout the project I'll be ironing it out. (deadline: 4/1/25)
- [ ] Basically, I need Fortran to handle the calculations and another language to handle to pretty looking part. Use ISO_C_Binding to interoperate Fortran with C++, using C as an intermediate (so that I can connect my code with the rendering engine I'll make) (deadline: 4/10/25)
- [ ] Create 3D rendering engine for molecule models. Probably C++ as its highly interoperable with C (which is pretty much the only language I can use with Fortran) and it's actually a reasonable language to write one in (unlike Fortran and C. absolutely no way am I gonna try to do this in C) (deadline: 4/21/25)
  - [ ] Render at least the 5 aforementioned types of molecules

#### Beyond MVP

- [ ] Add support for more molecules
  - [ ] Rewrite last year's project and seamlessly integrate both of them


<!-- EXAMPLE

## Tool: APIs
## Product: Green Glass Door riddle app

## Timeline

### MVP

- [ ] Front-end
  - [x] Webpage to collect input from user (deadline: 4/15)
  - [ ] Webpage to display "yes, but a ___ can't" or "no, but a ___ can" (deadline: 5/1)
- [x] Back-end
  - [x] Use regex to test whether or not the word can go through the GGD (deadline: 3/1)
  - [x] Use the Twinword API to find related words (deadline: 3/15)
    - [ ] Iterate through the words until an opposite example can be found (deadline: 4/1)

#### Beyond MVP

- [ ] Use another API to make sure the opposite example is a noun
- [ ] Automate notification of API limit to make sure I don’t exceed free quota
- [ ] A multiple choice quizzer that will test the user’s knowledge of the solution

-->





<!-- DO NOT USE THIS YET

| Name | Glows | Grows |
| -------- | ------- | ------- |
|   |   |
|   |   |
|   |   |
|   |   |
|   |   |
|   |   |

-->
