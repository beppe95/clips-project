# CLIPS-project
This CLIPS project has been written for the exam of Artificial Intelligence Laboratory at the University of Turin.

# Project Description
The project consists in trying to meet the needs of various Italian cities wares (product type can be *A*, *B*, *C*).

For simplicity it is assumed that each of the cities produces and needs only one type of product.

The following image reports the methods of production and consumption of each city: 
<p align="center">
  <img src="https://github.com/CesareIurlaro/clips-project/blob/master/stuff/initCIties.PNG"/>
</p>

We have several vehicles available: 
- 5 *vans* with capacity 4,
- 2 *planes* with capacity 7,
- 2 *ships* with capacity 11 
(where capacity is the maximum number of wares transportable from each vehicle).

Each vehicle is located in a prior known location: 
- 3 *vans* are in Bologna and 2 are in Rome,
- 1 *ships* is in Genoa and the other is in Venice,
- 1 *planes* is in Palermo, the other in Milan.


Each of the vehicles can perform three basic actions: 
- **load** (take goods),
- **unload** (drop goods), 
- **shift** (move between linked cities).

Each of these actions has a cost dependent on one or more parameters: 
- **loads** and **unloads**, the parameter is the *quantity of goods* that is treated by the action chosen, 
- **shifts**, the parameters are both the *vehicle type* that performs it and the *distance* between the two cities. 

The following are the functions used to calculate each cost:


<p align="center">
<img src="http://latex.codecogs.com/gif.latex?wares%5C_%5C%20cost%28quantity%29%20%3D%20quantity%20*%2010"/>
</p>


<p align="center">
  <img src="http://latex.codecogs.com/gif.latex?travel%5C_%5C%20cost%28type%2C%20distance%29%20%3D%20%5Cleft%5C%7B%5Cbegin%7Bmatrix%7D%20distance%2C%26%20%5C%3A%20if%20%5C%3A%20type%20%3D%20%22van%22%20%5C%5C%5B2pt%5D%20%5Cfrac%7B2%7D%7B3%7Ddistance%2C%26%20%5C%3A%20if%20%5C%3A%20type%20%3D%20%22ship%22%20%5C%5C%5B4pt%5D%20%5Cfrac%7B5%7D%7B4%7Ddistance%2C%26%20%5C%3A%20if%20%5C%3A%20type%20%3D%20%22plane%22%20%5C%5C%5B2pt%5D%20%5Cend%7Bmatrix%7D%5Cright."/>
</p>

Some journeys are not viable by all means of transport and therefore not all cities are directly connected to each other.


The following image represents the routes that can be traveled with the various types of vehicles 
(can also be found in the legend):
<p align="center">
  <img src="https://github.com/CesareIurlaro/clips-project/blob/master/stuff/route.PNG"/>
</p>

The following image represents the crow flies distances between the different cities:
<p align="center">
  <img src="https://github.com/CesareIurlaro/clips-project/blob/master/stuff/crow_flies_distances.PNG"/>
</p>

# Project management
The project, contained within the 'A-star' folder, has been divided into three main parts:
- `algorithm`, which contains all the CLIPS modules related to the algorithm used,
- `domain`, that contains functions, facts, rules and templates used specifically for the domain,
- `knowledge_bases`, that contains the knowledge bases used for the project.


There are also three other files necessary for the correct execution of the program.
They are:
- `loads`, which contains the instructions necessary to configure the execution environment,
- `projectCosts`, on which they are saved progressively the partial costs of the solution,
- `run`, used to open the CLIPS shell where to run the program.

To run the program open `run.bat` file (or use the tool **CLIPSIDE**) and type `(batch loads)` command.

# Search Algorithm
The suggested algorithms for solving the problem were **Iterative Deepening** and **A***. 
Both have been implemented and applied to perform the research on a graph in the space of the states.

**Iterative Deepening**: the algorithm proved to be unsuitable for the resolution of the problem, as it is unable to find a solution within a reasonable time. However, it was possible to make some observations:
drastically reducing the size of the problem it was possible to conclude the computation in a short time.
However, the addition of only a few facts to the knowledge base worsened of much the performances. 

His implementation is contained within the `no_evaluation_function` folder.

Because of this, we opted for the **A*** algorithm implementation.

**A\*:** this algorithm has a better behavior than the previous one, either because
- of the **linear spatial complexity** in the depth of the deepest optimal solution,
- it is helped by a **heuristic**.
**If the heuristic is admissible, then the algorithm is optimal.**

Despite this, **even A\* can not find a solution quickly.**

## Heuristic
We have implemented an **admissible heuristic**, which means that it is *never wrong for excess* and that it is *consistent* (or *monotonic*) for graph search applications. [Artificial Intelligence: A Modern Approach, S.J. Russel & P. Norvig]

Our heuristic associates to each node a cost that depends on the sum of:
- cost of operations (load or unload) on wares,
- cost of the journey as the crow flies in the case of a move.

The algorithm will choose the optimal path based on this heuristic.

## Sub-problems
To solve the non-termination due to the great complexity of the problem, it was necessary to divide it into several sub-problems, each of which deals with satisfying one of the previously mentioned cities.

The number of sub-problems into which the original problem has been divided appears to be
equal to the number of cities to be met; some of them have been further subdivided in such a way as to contain the `branching factor`.

This has precluded the possibility of finding the optimal solution of the whole problem but it guaranteed us the optimality of the sub-solutions. Therefore, the solution proposed by us turns out to be sub-optimal.

# Modeling the problem
The problem was modeled using, in combination, both ordered facts and non-ordered facts, contained within the `templates.clp` file, with prevalence of the first kind. The reason for this is to be found in the varied number of constructs usable only with ordered facts made available by CLIPS itself.

Where we have not considered necessary such constructs, we used non-ordered facts.

## The A* implementation in CLIPS
The A* implementation in the CLIPS language was done through the subdivision in the following modules:

- **Main**, which is responsible for instantiating the initial node and printing the total cost and the costs of the individual actions of the solution,
- **Expand**, that focus the algorithm node expansion on the actual best path cost,
- **Check**, which deals with checking whether within the various facts `status` the desired goal is contained,
- **New**, that:
  - deals both with updating the statistics (*closed*, *worse*, *better*) related to the algorithm,
  - ensures the aciclicity of the graph,
  - ensures that there is not more than one path that leads to equal open nodes. 


Basic data structures are needed to implement the A* algorithm. 
To realize them in CLIPS, ordered facts has been used.

They are:
- `node`, which represent the node currently examined,
- `newnode`, which expand, after making a action, the new nodes,
- `state`, which configure the environment through the a priori knowledge bases,
- `status`, which maintain informations about cities and means of transport. 
A precise configuration of a state of the state space is represented by the set of ordered facts `status` having the same value as the `ident` slot.

For goal modeling purpose, an ordered fact called `goal` was used.
By necessity, two goals have been modeled: one concerning cities and the other concerning means of transport.

## Persistency Problem and its resolution in CLIPS
To solve the persistency problem we have decided to use the `do-for-all-facts` CLIPS construct which cycles over any facts contained into the CLIPS Agenda matching a specified template. Each iteration duplicates any `status` fact present in the parent node that was and which still is `TRUE` in the child node.

## Knowledge Bases
The knowledge bases contain the cities and the vehicles necessary for satisfaction of the various sub-elements. 
They are contained, as described above, in an ordered fact called `state`, which represents the **root** of the graph on which we perform the search in the states space.

## Modeling sub-problems
Our sub-problems have been divided into:

**1. TO**: subgraph with Turin (TO), Rome (RM) and Palermo (PA). 
We meet Turin object A needs by plane (`vehicle_6`) which is located in PA.

**2. MI**: subgraph with Milan (MI), Rome (RM), Naples (NA), Bari (BA) and Reggio Calabria (RC). We meet Milan object A needs using two vans (`vehicle_4` and `vehicle_5`) which are located in RM and by plane (`vehicle_7`) located in MI. 

The original subproblem has been divided into:

- Transfer of type A goods from RC to NA,
- Transfer of 10 A goods to MI,
- Transfer of remaining 20 A goods to MI.

**3. VE**: subgraph with Venice (VE) and Bologna (BO). We meet Venice object B needs by van (`vehicle_2`) located in BO. 

**4. GE**: subgraph with Genoa (GE), Turin (TO), Milan (MI) and Bologna (BO). We meet Genoa object B need by van (`vehicle_1`) located in BO. The original subproblem has been divided into:

- Shift van (`vehicle_1`) from BO to TO,
- Transfer of B goods with `vehicle_1` from TO to GE.


**5. BA**: subgraph with Bari (BA), Milano (MI) and Bologna (BO). We meet Bari object B needs by van (`vehicle_3`) located in BO and by plane (`vehicle_7`) located in MI. 

The original subproblem has been divided into:

- Transfer of type B goods from BO to MI with van (`vehicle_3`),
- Transfer by plane (`vehicle_7`) of previously displaced goods from MI to BA.

**6. RC**: subgraph with Reggio Calabria (RC) and Naples (NA). We meet Reggio Calabria object B needs by van (`vehicle_5`) located in NA.

**7. NA_PA**: subgraph with Naples (NA), Palermo (PA) and Genoa (GE). We meet both Naples and Genoa object needs by ship (`vehicle_8`) which is located in GE.


**8. RM**: subgraph with Rome (RM), Turin (TO) and Milan (MI). We meet Rome object C needs by plane (`vehicle_6`) which is located in TO. 

**9. BO**: subgraph with Bologna (BO) and Venice (VE). We meet Bologna object C needs by van (`vehicle_2`) located in VE. 


The original subproblem has been divided into:

- Transfer of type C goods from MI to TO with van (`vehicle_3`),
- Transfer by plane (`vehicle_6`) of previously displaced goods from TO to RM.


## Domain rules
The domain rules are contained in the `domain_rules.clp` file.

In practice, we divided each of them in the following way:
- `load-prod` and `load-store` are the rules involved in the **load** action. 
  - `load-prod` is used to load any of the object **produced** by the cities into any means of transport which is located in those cities,
  - `load-store` is used to load any of the object **stored** by the cities into any means of transport which is located in those cities.
  
- `unload-need` and `unload-store` are the rules involved in the **unload** action.
  - `unload-need` is used to unload any of the object **needed** by each cities referred by our goals, 
  - `unload-store` is used to unload any of the object **not produced** and **not needed** by the cities to store them.
  
- `shift` is the rule involved in the **shift** action which is used to change the location of any means of transport. 

# Utilities
They are contained within the `utils` folder.

The **functions** are contained within the `functions.clp` file.
They are:
- `travel_cost_evaluation` calculates, given the means of transport used and the distance from travel, the cost of the shift
- `find_heuristic_costs` takes care of calculating the value of the heuristic used
- `string_comparator` compares two strings passed as parameters
- `prepare_string` produces a string that concatenates all of their `status` having the same value as the `ident` slot. 
The resulting string is representative of the configuration of that state.
- `sum_up_costs` is responsible for calculating the total cost of the solution and the costs of the sub-solutions.

The **rules** are contained within the `rules.clp` file, responsible of producing facts and performing calculations, supporting the calculation of the heuristic.

The `cf_distances.clp` file contains:
- `h_distance`, ordered fact which represents the crow flies distance; used for the calculation of the heuristic
- `distance`, not ordered facts which represents the crow flies distance that can actually be traveled and the specific means of transport which can be used to do it.

# Conclusions
The sub-problems have been satitisfied with the following actions:

**1. TO**

 ```
Eseguo azione unload-need con costo 60 (transport plane 1 TO 6 city TO 10 B 6 A 0 C carries 6 6 A)
 Eseguo azione shift con costo 836 (transport plane 1 RM 6 city RM 0 A 5 C 0 B city TO 10 B 6 A 0 C)
 Eseguo azione load-prod con costo 60 (transport plane 7 RM 6 city RM 6 A 5 C 0 B)
 Eseguo azione shift con costo 836 (transport plane 7 TO 6 city TO 10 B 6 A 0 C city RM 6 A 5 C 0 B)
 Eseguo azione unload-need con costo 40 (transport plane 3 TO 6 city TO 10 B 10 A 0 C carries 6 4 A)
 Eseguo azione unload-need con costo 30 (transport plane 0 TO 6 city TO 10 B 13 A 0 C carries 6 3 A)
 Eseguo azione shift con costo 836 (transport plane 0 RM 6 city RM 6 A 5 C 0 B city TO 10 B 13 A 0 C)
 Eseguo azione load-prod con costo 40 (transport plane 4 RM 6 city RM 10 A 5 C 0 B)
 Eseguo azione shift con costo 836 (transport plane 4 TO 6 city TO 10 B 13 A 0 C city RM 10 A 5 C 0 B)
 Eseguo azione shift con costo 745 (transport plane 4 PA 6 city PA 0 A 5 C 0 B city TO 10 B 13 A 0 C)
 Eseguo azione load-prod con costo 30 (transport plane 7 PA 6 city PA 3 A 5 C 0 B)
 Eseguo azione shift con costo 1995 (transport plane 7 TO 6 city TO 10 B 13 A 0 C city PA 3 A 5 C 0 B)
 Eseguo azione unload-need con costo 70 (transport plane 0 TO 6 city TO 10 B 20 A 0 C carries 6 7 A)
 Eseguo azione shift con costo 745 (transport plane 0 PA 6 city PA 3 A 5 C 0 B city TO 10 B 20 A 0 C)
 Eseguo azione load-prod con costo 70 (transport plane 7 PA 6 city PA 10 A 5 C 0 B)
 ```
 
 `Esiste soluzione con costo 7229`

**2. MI**

- subproblem a.

```
Eseguo azione unload-store con costo 40 (transport van 0 NA 5 city NA 5 B 5 C 16 A carries 5 4 C)
 Eseguo azione shift con costo 462 (transport van 0 RC 5 city RC 0 A 5 B 0 C city NA 5 B 5 C 16 A)
 Eseguo azione load-prod con costo 40 (transport van 4 RC 5 city RC 4 A 5 B 0 C)
 Eseguo azione shift con costo 462 (transport van 4 NA 5 city NA 5 B 5 C 16 A city RC 4 A 5 B 0 C)
 Eseguo azione unload-store con costo 40 (transport van 0 NA 5 city NA 5 B 5 C 12 A carries 5 4 C)
 Eseguo azione shift con costo 462 (transport van 0 RC 5 city RC 4 A 5 B 0 C city NA 5 B 5 C 12 A)
 Eseguo azione load-prod con costo 40 (transport van 4 RC 5 city RC 8 A 5 B 0 C)
 Eseguo azione shift con costo 462 (transport van 4 NA 5 city NA 5 B 5 C 12 A city RC 8 A 5 B 0 C)
 Eseguo azione unload-store con costo 40 (transport van 0 NA 5 city NA 5 B 5 C 8 A carries 5 4 C)
 Eseguo azione shift con costo 462 (transport van 0 RC 5 city RC 8 A 5 B 0 C city NA 5 B 5 C 8 A)
 Eseguo azione load-prod con costo 40 (transport van 4 RC 5 city RC 12 A 5 B 0 C)
 Eseguo azione shift con costo 462 (transport van 4 NA 5 city NA 5 B 5 C 8 A city RC 12 A 5 B 0 C)
 Eseguo azione unload-store con costo 40 (transport van 0 NA 5 city NA 5 B 5 C 4 A carries 5 4 C)
 Eseguo azione shift con costo 462 (transport van 0 RC 5 city RC 12 A 5 B 0 C city NA 5 B 5 C 4 A)
 Eseguo azione load-prod con costo 40 (transport van 4 RC 5 city RC 16 A 5 B 0 C)
 Eseguo azione shift con costo 462 (transport van 4 NA 5 city NA 5 B 5 C 4 A city RC 16 A 5 B 0 C)
 Eseguo azione unload-store con costo 40 (transport van 0 NA 5 city NA 5 B 5 C 0 A carries 5 4 C)
 Eseguo azione shift con costo 462 (transport van 0 RC 5 city RC 16 A 5 B 0 C city NA 5 B 5 C 0 A)
 Eseguo azione load-prod con costo 40 (transport van 4 RC 5 city RC 20 A 5 B 0 C)
 Eseguo azione shift con costo 462 (transport van 4 NA 5 city NA 5 B 5 C 0 A city RC 20 A 5 B 0 C)
 Eseguo azione shift con costo 219 (transport van 4 RM 5 city RM 0 A 5 C 0 B city NA 5 B 5 C 0 A)
```
 
`Esiste soluzione con costo 5239`

- subproblem b.

```
Eseguo azione unload-need con costo 30 (transport plane 4 MI 7 city MI 5 C 23 A 0 B carries 7 3 A)
 Eseguo azione shift con costo 888 (transport plane 4 BA 7 city BA 0 A 5 B 0 C city MI 5 C 23 A 0 B)
 Eseguo azione load-prod con costo 30 (transport plane 7 BA 7 city BA 3 A 5 B 0 C)
 Eseguo azione shift con costo 888 (transport plane 7 MI 7 city MI 5 C 23 A 0 B city BA 3 A 5 B 0 C)
 Eseguo azione unload-need con costo 70 (transport plane 0 MI 7 city MI 5 C 30 A 0 B carries 7 7 A)
 Eseguo azione shift con costo 888 (transport plane 0 BA 7 city BA 3 A 5 B 0 C city MI 5 C 30 A 0 B)
 Eseguo azione load-prod con costo 70 (transport plane 7 BA 7 city BA 10 A 5 B 0 C)
 Eseguo azione shift con costo 888 (transport plane 7 MI 7 city MI 5 C 30 A 0 B city BA 10 A 5 B 0 C)
 ```
 
`Esiste soluzione con costo 3752`

- subproblem c.

```
Eseguo azione unload-need con costo 60 (transport plane 1 MI 7 city MI 5 C 6 A 0 B carries 7 6 A)
 Eseguo azione shift con costo 955 (transport plane 1 NA 7 city NA 5 B 5 C 0 A city MI 5 C 6 A 0 B)
 Eseguo azione load-store con costo 60 (transport plane 7 NA 7 city NA 5 B 5 C 6 A)
 Eseguo azione shift con costo 955 (transport plane 7 MI 7 city MI 5 C 6 A 0 B city NA 5 B 5 C 6 A)
 Eseguo azione unload-need con costo 70 (transport plane 0 MI 7 city MI 5 C 13 A 0 B carries 7 7 A)
 Eseguo azione shift con costo 955 (transport plane 0 NA 7 city NA 5 B 5 C 6 A city MI 5 C 13 A 0 B)
 Eseguo azione load-store con costo 70 (transport plane 7 NA 7 city NA 5 B 5 C 13 A)
 Eseguo azione shift con costo 955 (transport plane 7 MI 7 city MI 5 C 13 A 0 B city NA 5 B 5 C 13 A)
 Eseguo azione unload-need con costo 70 (transport plane 0 MI 7 city MI 5 C 20 A 0 B carries 7 7 A)
 Eseguo azione shift con costo 955 (transport plane 0 NA 7 city NA 5 B 5 C 13 A city MI 5 C 20 A 0 B)
 Eseguo azione load-store con costo 70 (transport plane 7 NA 7 city NA 5 B 5 C 20 A)
 Eseguo azione shift con costo 955 (transport plane 7 MI 7 city MI 5 C 20 A 0 B city NA 5 B 5 C 20 A)
```
 
`Esiste soluzione con costo 6130`

**3. VE**

```
Eseguo azione unload-need con costo 10 (transport van 3 VE 2 city VE 10 C 1 B 0 A carries 2 1 B)
 Eseguo azione shift con costo 158 (transport van 3 BO 2 city BO 0 B 10 C 0 A city VE 10 C 1 B 0 A)
 Eseguo azione load-prod con costo 10 (transport van 4 BO 2 city BO 1 B 10 C 0 A)
 Eseguo azione shift con costo 158 (transport van 4 VE 2 city VE 10 C 1 B 0 A city BO 1 B 10 C 0 A)
 Eseguo azione unload-need con costo 40 (transport van 0 VE 2 city VE 10 C 5 B 0 A carries 2 4 B)
 Eseguo azione shift con costo 158 (transport van 0 BO 2 city BO 1 B 10 C 0 A city VE 10 C 5 B 0 A)
 Eseguo azione load-prod con costo 40 (transport van 4 BO 2 city BO 5 B 10 C 0 A)
 ```
 
`Esiste soluzione con costo 574`

**4. GE**

- subproblem a.

```
Eseguo azione shift con costo 138 (transport van 4 MI 1 city MI 5 C 0 A 0 B city TO 10 B 0 A 0 C)
 Eseguo azione shift con costo 206 (transport van 4 BO 1 city BO 5 B 10 C 0 A city MI 5 C 0 A 0 B)
```
 
`Esiste soluzione con costo 344`

- subproblem b.

```
Eseguo azione unload-need con costo 10 (transport van 3 GE 1 city GE 10 C 1 B 0 A carries 1 1 B)
 Eseguo azione shift con costo 170 (transport van 3 TO 1 city TO 0 B 0 A 0 C city GE 10 C 1 B 0 A)
 Eseguo azione load-prod con costo 10 (transport van 4 TO 1 city TO 1 B 0 A 0 C)
 Eseguo azione shift con costo 170 (transport van 4 GE 1 city GE 10 C 1 B 0 A city TO 1 B 0 A 0 C)
 Eseguo azione unload-need con costo 40 (transport van 0 GE 1 city GE 10 C 5 B 0 A carries 1 4 B)
 Eseguo azione shift con costo 170 (transport van 0 TO 1 city TO 1 B 0 A 0 C city GE 10 C 5 B 0 A)
 Eseguo azione load-prod con costo 40 (transport van 4 TO 1 city TO 5 B 0 A 0 C)
```
 
`Esiste soluzione con costo 610`

**5. BA**

- subproblem a.

```
Eseguo azione unload-store con costo 10 (transport van 3 MI 3 city MI 5 C 0 A 4 B carries 3 1 A)
 Eseguo azione shift con costo 206 (transport van 3 BO 3 city BO 0 B 10 C 0 A city MI 5 C 0 A 4 B)
 Eseguo azione load-prod con costo 10 (transport van 4 BO 3 city BO 1 B 10 C 0 A)
 Eseguo azione shift con costo 206 (transport van 4 MI 3 city MI 5 C 0 A 4 B city BO 1 B 10 C 0 A)
 Eseguo azione unload-store con costo 40 (transport van 0 MI 3 city MI 5 C 0 A 0 B carries 3 4 A)
 Eseguo azione shift con costo 206 (transport van 0 BO 3 city BO 1 B 10 C 0 A city MI 5 C 0 A 0 B)
 Eseguo azione load-prod con costo 40 (transport van 4 BO 3 city BO 5 B 10 C 0 A)
```
 
`Esiste soluzione con costo 718`

- subproblem b.

```
Eseguo azione unload-need con costo 50 (transport plane 2 BA 7 city BA 0 A 5 B 0 C carries 7 5 B)
 Eseguo azione shift con costo 888 (transport plane 2 MI 7 city MI 5 C 0 A 0 B city BA 0 A 5 B 0 C)
 Eseguo azione load-store con costo 50 (transport plane 7 MI 7 city MI 5 C 0 A 5 B)
```
 
`Esiste soluzione con costo 988`

**6. RC**

```
Eseguo azione unload-need con costo 10 (transport van 3 RC 5 city RC 0 A 1 B 0 C carries 5 1 B)
 Eseguo azione shift con costo 462 (transport van 3 NA 5 city NA 0 B 5 C 0 A city RC 0 A 1 B 0 C)
 Eseguo azione load-prod con costo 10 (transport van 4 NA 5 city NA 1 B 5 C 0 A)
 Eseguo azione shift con costo 462 (transport van 4 RC 5 city RC 0 A 1 B 0 C city NA 1 B 5 C 0 A)
 Eseguo azione unload-need con costo 40 (transport van 0 RC 5 city RC 0 A 5 B 0 C carries 5 4 B)
 Eseguo azione shift con costo 462 (transport van 0 NA 5 city NA 1 B 5 C 0 A city RC 0 A 5 B 0 C)
 Eseguo azione load-prod con costo 40 (transport van 4 NA 5 city NA 5 B 5 C 0 A)
```
 
`Esiste soluzione con costo 1486`

**7. NA_PA**

```
Eseguo azione unload-need con costo 50 (transport ship 6 NA 8 city NA 0 B 5 C 0 A carries 8 5 C)
 Eseguo azione shift con costo 493 (transport ship 6 PA 8 city PA 0 A 0 C 0 B city NA 0 B 5 C 0 A)
 Eseguo azione unload-need con costo 50 (transport ship 1 PA 8 city PA 0 A 5 C 0 B carries 8 10 C)
 Eseguo azione shift con costo 941 (transport ship 1 GE 8 city GE 0 C 0 B 0 A city PA 0 A 5 C 0 B)
 Eseguo azione load-prod con costo 100 (transport ship 11 GE 8 city GE 10 C 0 B 0 A)
```
 
`Esiste soluzione con costo 1634`


**8. RM**

- subproblem a.

```
Eseguo azione unload-store con costo 10 (transport van 3 TO 3 city TO 0 B 0 A 4 C carries 3 1 A)
 Eseguo azione shift con costo 138 (transport van 3 MI 3 city MI 0 C 0 A 0 B city TO 0 B 0 A 4 C)
 Eseguo azione load-prod con costo 10 (transport van 4 MI 3 city MI 1 C 0 A 0 B)
 Eseguo azione shift con costo 138 (transport van 4 TO 3 city TO 0 B 0 A 4 C city MI 1 C 0 A 0 B)
 Eseguo azione unload-store con costo 40 (transport van 0 TO 3 city TO 0 B 0 A 0 C carries 3 4 A)
 Eseguo azione shift con costo 138 (transport van 0 MI 3 city MI 1 C 0 A 0 B city TO 0 B 0 A 0 C)
 Eseguo azione load-prod con costo 40 (transport van 4 MI 3 city MI 5 C 0 A 0 B)
```
 
`Esiste soluzione con costo 514`

- subproblem b.

```
Eseguo azione unload-need con costo 50 (transport plane 2 RM 6 city RM 0 A 5 C 0 B carries 6 5 C)
 Eseguo azione shift con costo 836 (transport plane 2 TO 6 city TO 0 B 0 A 0 C city RM 0 A 5 C 0 B)
 Eseguo azione load-store con costo 50 (transport plane 7 TO 6 city TO 0 B 0 A 5 C)
```
 
`Esiste soluzione con costo 936`

**9. BO**

```
Eseguo azione unload-need con costo 20 (transport van 2 BO 2 city BO 0 B 2 C 0 A carries 2 2 C)
 Eseguo azione shift con costo 158 (transport van 2 VE 2 city VE 0 C 0 B 0 A city BO 0 B 2 C 0 A)
 Eseguo azione load-prod con costo 20 (transport van 4 VE 2 city VE 2 C 0 B 0 A)
 Eseguo azione shift con costo 158 (transport van 4 BO 2 city BO 0 B 2 C 0 A city VE 2 C 0 B 0 A)
 Eseguo azione unload-need con costo 40 (transport van 0 BO 2 city BO 0 B 6 C 0 A carries 2 4 C)
 Eseguo azione shift con costo 158 (transport van 0 VE 2 city VE 2 C 0 B 0 A city BO 0 B 6 C 0 A)
 Eseguo azione load-prod con costo 40 (transport van 4 VE 2 city VE 6 C 0 B 0 A)
 Eseguo azione shift con costo 158 (transport van 4 BO 2 city BO 0 B 6 C 0 A city VE 6 C 0 B 0 A)
 Eseguo azione unload-need con costo 40 (transport van 0 BO 2 city BO 0 B 10 C 0 A carries 2 4 C)
 Eseguo azione shift con costo 158 (transport van 0 VE 2 city VE 6 C 0 B 0 A city BO 0 B 10 C 0 A)
 Eseguo azione load-prod con costo 40 (transport van 4 VE 2 city VE 10 C 0 B 0 A)
```
 
`Esiste soluzione con costo 990`


For each subproblem, once the program find the solution, it asserts an ordered fact named `file_total_cost` that contains the cost for that solution and write it in the `projectCosts.fct` file.
The total cost is then calculated, by a function named `sum_up_costs` contained in `functions.clp` file. 
This function sums all over the `file_total_cost` facts and calculate the overall cost in this way:

<p align="center">
  <img src="http://latex.codecogs.com/gif.latex?cities%20%3D%20%5Cleft%20%5C%7BTO%2C%20%5C%3A%20MI%2C%5C%3A%20VE%2C%5C%3A%20GE%2C%5C%3A%20BO%2C%5C%3A%20RM%2C%5C%3A%20NA%2C%5C%3A%20PA%2C%5C%3A%20BA%2C%5C%3A%20RC%20%5Cright%20%5C%7D%20%5C%5C%20%5C%5C"/>
</p>

<p align="center">
  <img src="http://latex.codecogs.com/gif.latex?overall%5C_%20%5C%20cost%5C%20%3D%20%5Csum_%7Bcity%3A%5C%3Acities%7Dsubproblem%5C_%20%5C%20cost%28city%29"/>
</p>

```
Il costo totale è : 31144
```

# Post exam

Some critical issues emerged, so we reflected on the appropriate changes that could remedy and implemented them.

## **Status comparison**

Each state of the space is identified by the set of facts having the same id.

- **Current approach**: the comparison takes place in the following steps:
1) Concatenation of strings related to the same state
2) Comparison of single strings to check for duplicates

- **Implemented alternative**: comparison with rules.

If there is at least one fact which is not valid for the two states at the same time, then the states are different.
Otherwise they are the same.

## **Iteration between the sub-problems of the algorithm**

- **Current approach**: the knowledge necessary to solve every single sub-problem is written in a file related to it.
The algorithm, for each sub-problem, carries out the following steps:
1) Read the facts necessary to solve the iteration of the current sub-problem
2) Resolve the sub-problem
3) Removes all the facts.

- **Implemented alternative**: at the end of the resolution of a single sub-problem, the knowledge necessary to solve the sub-problem of the next iteration is saved in the unordered fact `state`, which acts as a container of useful information to the subsequent iteration of the algorithm. Every other fact is then retracted.
`state` is so used as beginning of the following iteration and its content act as the initial state of the next sub-problem.
In this way, the writing of the individual knowledge bases of each sub-problem is no longer **manual**, but is delegated to the resolution algorithm itself starting from the initial state, and thus becomes **automatic**.

## **Removal of the hard coding related to the knowledge bases of the sub-objectives**

- **Current approach**: the individual sub-objectives have been hardcoded to solve the problem of the enormous branch factor of the state space graph.
Correcting this inelegant approach would require, due to the specific modeling adopted for the domain and the algorithm, a non-indifferent effort in terms of changes to large parts of the project, therefore only a hypothetical workaround in words will be presented below.

- **Not implemented alternative**: the problem should be decomposed into objectives such as "satisfy (bring to 0 needs) a given city *c*". Each of these sub-objectives could further be broken down through the use of some heuristics into the following sub-objectives:
1) identify the city with the greatest quantity of the necessary good to the city *c*
2) identify the means of transport that takes less road to transport that good between the two cities and use it to do it
3) repeat (1) and (2) until the resources of the city that exports are exhausted
4) return to (a) if *c* has not been satisfied yet 

## Authors

* **Cesare Pio Iurlaro** - [CesareIurlaro](https://github.com/CesareIurlaro)
* **Giuseppe Gabbia**  - [beppe95](https://github.com/beppe95)
* **Lamberto Basti**  - [lamba92](https://github.com/lamba92)
