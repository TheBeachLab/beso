# b.e.s.o.
Python code for a topology optimization method known as `b.e.s.o` bidirectional evolutionary structural optimization  using CalculiX FEM solver.

## Topology optimization, BESO method

BESO (Bi-directional Evolutionary Structural Optimization) method is one of methods, which can be used for a topology optimization with connection to FEM (Finite Element Method). BESO method works in iterations where in the each iteration least effective elements are removed (and some elements removed earlier can be added back). This method can be used in an initial phase of product design. It does not mean that product or part is completely designed by the program, but it can give an initial guidance how should a designer place material to be efficiently used. It is important to keep in mind that, firstly, it is a heuristic method so there is no certainty that given solution is the real optimum (i. e. the best solution). Secondly, BESO method needs some input parameters, which influences result quality and speed of convergence.


Presented program uses CalculiX solver and was at the beginning inspired by the algorithm described in the book: Evolutionary topology optimization of continuum structures, Methods and applications 2008, written by X. Huang and Z.M. Xie. The goal of the presented program is to achieve the design with low weight. User define list of allowable element states (effective densities, materials and for shell elements also thicknesses). Program switches elements from initial highest states to lower element states, based on sensitivity number. Element sensitivity number is computed as
failure index / effective density
where failure index equals actual stress divided by allowable stress.
Switching to softer state (material) is stopped after achieving goal mass (thanks to decreasing effective densities or thicknesses). Another stop criterion can be element failure (failure index >= 1).

Brief description of the algorithm was given in the conference paper which is available [here](doc_files/EM2017_proceedings_all.pdf). Extracted from the original ENGINEERING MECHANICS 201
7, 23rd INTERNATIONAL CONFERENCE [Book of full texts](http://www.engmech.cz/2017/im/doc/EM2017_proceedings_all.pdf) (pages 590-593):

LÖFFELMANN, F. Failure Index Based Topology Optimization for Multiple Properties. In _Engineering Mechanics 2017_. 1. konference Engineering mechanics 2014, Svratka, ČR: Brno University of Technology, 2017. s. 590-593. ISBN: 978-80-214-5497-2. ISSN: 1805-8248.

## Current implementation idea

User prepares .inp input file in the same way as for a common static analysis, than in the beso_conf.py file sets input parameters, which consist of material inputs of solid sections (shell sections) and parameters for the optimization. The optimization program run in iterations. In each iteration it copies input file with rewritten properties of elements in the optimization domain. Properties for elements are taken from user defined list of allowable states. The program in each iteration switches the least used elements to lower state (softer) and it can also switch maximally used or overstressed elements to higher state (stronger). It iterates until given mass is achieved or limit of failing elements is achieved.


For example lets have prescribed allowable states for 3 materials: "void", aluminium, and steel. Optimization starts with the highest state (all elements as steel). In each iteration sensitivity number is computed for all elements in prescribed domains. Sensitivity number (failure index divided by effective density) is heuristic measure how efficient given element is. Small portion of elements with the lowest sensitivity number are switched down by one state (e.g. from steel to aluminium), so mass decreases. Less elements with the highest sensitivity number (and failing elements) are switched up (e.g. from "void" to aluminium, or from aluminium to steel).


## Possibilities / limitations

Optimization can take into account all CalculiX volume, shell, axisymmetric, plane strain and plain stress element types. Thanks to the principle that input file is copied with rewritten optimization properties, it should be possible to use any other element types (e.g. beams) and keywords in the model which will remain untouched (i.e. model should still contain them). Simplification in coding was that centres of gravity and volumes of 2nd order elements are computed in the same way as for 1st order elements; errors in volumes and c.g. positions are assumed to be, for ordinary used meshes, low enough for using by filters.

Mass is sequentially removed (by switching down element states) until goal mass is achieved or limit of failing elements is violated.

Allowable element states may contain also thicknesses for shells. Instead of (together with) softening material, thickness can be reduced.

Basic method has problem with oscillations after achieving goal mass if there are more then 2 allowable states. Thus artificial decaying can be used to suppress oscillations.

Several filters were coded to prevent "checkerborad" effect, to fit different tasks (which is still not perfect). Model can contain different filter settings for different mesh domains. Filters can enforce minimum material (void) spot size, but other manufacturing constraints are not implemented yet.

Multiple load cases can be modelled by adding more steps into .inp file. By default CalculiX adds loads and boundary conditions of current step to those from previous step. Concentrated loads can be reset by line *CLOAD, OP=NEW, distributed loads by *DLOAD, OP=NEW and boundary conditions by *BOUNDARY, OP=NEW.

## Examples

### Example 1. Simply supported 2D Beam
Access the full worked example [here.](doc_files/example_1/example1.md)

### Example 2. Engine bracket
Access the full worked example [here.](doc_files/example_2/example2.md)

