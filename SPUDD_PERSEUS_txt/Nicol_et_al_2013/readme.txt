# please refer to https://sites.google.com/site/adaptivemanagementeaa/home
# we have copied across the problems but the website provides more information

Overview of the structure of the symbolic Perseus files included with this data challenge
---------------------------------------
Author:

Sam Nicol Sam.Nicol@csiro.au

-----------------------------------------
CONTENTS:
* POMDP input files (SYMBOLIC PERSEUS)
* file structure
	- preamble
	- action decision diagrams (dds)
	- reward, discount, tolerance
* definition of breeding population states for each species	

-----------------------------------------	
POMDP input files
-----------------------------------------
The following problems are provided with this documentation:

curlew_sandpiper.txt
grey-tailed_tattler.txt
red_knot_p.txt
red_knot_r.txt
terek_sandpiper.txt
bar-tailed_godwit_m.txt
bar-tailed_godwit_b.txt
great_knot.txt
far_eastern_curlew.txt
lesser_sand_plover.txt

Files can be downloaded from:
https://sites.google.com/site/adaptivemanagementeaa/downloads/pomdp_input_files

Each input file represents a network of internationally important staging sites for the migratory 
bird species in the filename. An image showing the structure of the network for each species can be
downloaded from https://sites.google.com/site/adaptivemanagementeaa/downloads/network-images. 

Some of the files are also provided in Cassandra's format. More information on this is in the 
document README_CASSANDRA.txt, which can also be downloaded from:
https://sites.google.com/site/adaptivemanagementeaa/downloads/pomdp_input_files.

File structure
----------------------------------------------
Each file is formatted for Symbolic Perseus (available from
https://cs.uwaterloo.ca/~ppoupart/software.html).

File structure is detailed below.
PREAMBLE: 
---------
variables: a list of the variables. These are:
		- pop: 	 	the population state at the breeding node. There are 4 states. s1 represents the 
					smallest population and s4 represents the largest population. The exact populations 
					that they represent are different for each species and not required for Symbolic 
					Perseus to run.	In case these do become useful, I have included a code snippet at the 
					bottom of this document showing the population states.
					
		- nodex:	the protection status of node x, categorized as low, medium or high. These levels represent
					protection against sea level rises of 0m, 1m, 2m respectively. If more models (see
					next bullet point) are	included in the problem then more protection levels are also 
					required
		- model:	scenarios of sea level rise effects on the network nodes. Three models are 
					included: 0m,1m and 2m (models rise0cm, rise100cm, rise200cm respectively).
					
observations: 	observable variables. In this problem all variables are completely 
				observable except the hidden model variable. 
		- pobs:		observable variable for pop. possible observations are o1,o2,o3,o4.	
		- obsx:		observable variables for nodex. possible observations are lo_, med_, hi_
		
initial belief: 	specifies the initial belief for all variables. All input files start from a state
					with maximum population (s4) and no protection of nodes (low). The model is unknown
					so there is an equal belief in each model at time zero.
				
unnormalized:		variables are unnormalized. For more info, see symbolic Perseus documentation.

decision diagrams: 	These ADDs give transition probabilities between variables. Refer to the influence 
					diagram for an image of the relationships between variables, available at:
					https://sites.google.com/site/adaptivemanagementeaa/downloads/influence_diagram.

	- actiondds: 	Transitions between protection levels are influenced only by the current protection
					level, so these can be specified once at this point in the file then referred to
					later. Refer to the description of P(v'|v) in the submission for details of 
					ddnoactionx and ddactionx matrices. For	reference: 
					P_action=[0.01 0.99 0; 0 0.01 0.99;0 0 1];      	%prob action is successful
					P_decline= [1 0 0; 0.075 0.925 0; 0 0.075 0.925]; 	%prob state declines/threat level increases
					
	- modeldd:		Transition between models, influenced only by the current model. Models transition on
					average every 20 years and cannot return to a lower	sea level rise model. This is how
					we model non-stationarity as a stationary system. 
					Refer to the description of P(m'|m) in the submission for details of modeldd.
					For reference: P(m'|m)= [0.95, 0.05 0; 0 0.95 0.05; 0 0 1];
					
	- popOF			Observation function for observable variables-- in this case all variables are 
					perfectly observable so this is the identity matrix.
	
ACTION DECISION DIAGRAMS:
-------------------------
These dds give the full transition matrices for all possible state transitions. Factored states can refer
to the dds that are described in the previous section.
Action names are self-explanatory, e.g. 'action a2' means act at node 2. Each species has an 'action DN' 
action, which is a do nothing action. 
The matrices are structured in the same way for each action. P(v'|v) and P(m'|m) are factored and already
described. P(x'|x,v,m,a) is a very large DD and returns the probability that the number of birds 
returning to the breeding node is close to state x'. This is computed from the maximum flow through the 
network given protection level v and model m. The effect of each sea level rise (sea_level_rise_by_model.tar),
the capacities of each network with 0cm sea level rise (network_capacities.tar) and the parameters for 
the stochastic Gompertz model (gompertz_model_parameters.csv) can be downloaded from 
https://sites.google.com/site/adaptivemanagementeaa/downloads/raw-network-data-files,
however they are not necessary to run the POMDP file.

At the bottom of each action's DD are the observation function for each node (the identity) 
and the cost of action. Cost is determined by the amount of habitat lost if sea level rise exceeds
the protection level of the action node. The habitat lost is converted to 000's of birds lost by assuming
the that proportion of habitat lost is equivalent to the proportion of birds initially using the habitat
that is lost. This is computed using the proportion of habitat lost under each scenario assuming 
intertidal areas can migrate inland if a node is protected (see manuscript for details). We 
multiply this number by 10 to increase differentiation between states.

REWARD, DISCOUNT, TOLERANCE:
----------------------------
Reward is equal to the population of birds in state x minus the cost of action defined in each action dd.
The population of birds (x10^{3}) is multiplied by 10 to increase differentiation between states. 
i.e. Reward= 10* (population of birds in state x - cost of action during timestep).
In the input file, reward gives 10* population of birds in state x.

A discount factor of 0.95 is applied to all species to ensure convergence of the algorithm
A tolerance of 0.05 is used to specify when the algorithm should terminate.

----------------------------------------------------------------------------------------------------
Definition of breeding population states for each species:

The following MATLAB code snippet defines:
flyway_eq: 	total number of birds ('000s) currently using the flyway (i.e. 0cm sea level rise)
states: 	definition of the number of birds at the breeding site (in '000s) used to define the states 
			pop= {s1,s2,s3,s4} 	.
action_nodes: specifies which nodes it is possible to act on-- the numbers refer to the row of the 
			  network structure  matrices in the raw data files (downloadable from 
			  https://sites.google.com/site/adaptivemanagementeaa/downloads/raw-network-data-files).
tot_nodes: 	total number of nodes in the network. This count includes outward and inward bound journey, so is 
			~2x the number of nodes reported in the manuscript. Table 1 of the manuscript only reports the 
			number of action nodes (these may or may not be revisited on the return journey) plus 
			the breeding node, since these are the relevant metrics for computing the state space 
			in table 1.
			
CODE SNIPPET DEFINING BREEDING POPULATION STATES FOR EACH SPECIES:			
			
switch species
    case 'bar-tailed_godwit_m'
        flyway_eq= 190;
        states= [120 140 160 180];
        action_nodes= [2,3,4,5,6];
        tot_nodes= 11;
    case 'bar-tailed_godwit_b' 
        flyway_eq= 185;
        states= [80 120 140 160]; 
        action_nodes= [3,4,5,2];
        tot_nodes= 9;
    case 'far_eastern_curlew'
        flyway_eq= 38;  %flyway abundance at equilibrium in '000s 
        states= [14,20,26,32];
        action_nodes=[2,4,5,6,7,8,3,9];
        tot_nodes= 15;
    case 'curlew_sandpiper'
        flyway_eq= 180;  %flyway abundance at equilibrium in '000s 
        states= [100 120 140 160];
        action_nodes= [2,3,4,5,6,7,8,9];
        tot_nodes= 18;
    case 'great_knot'
        flyway_eq= 353; %flyway abundance at equilibrium in '000s 
        states= [150,200,250,300];
        action_nodes= [3,4,6,7,8,9,5,2];
        tot_nodes= 15;
    case 'grey-tailed_tattler'
        flyway_eq= 50;  %flyway abundance at equilibrium in '000s 
        states= [15 25 35 45];
        action_nodes= [2,3,4,5,6];
        tot_nodes= 12;
    case 'lesser_sand_plover'
        flyway_eq= 40;  %flyway abundance at equilibrium in '000s 
        states= [20 25 30 35];
        action_nodes= [2,3];
        tot_nodes= 6;
    case 'red_knot_p'
        flyway_eq= 44.9;  %flyway abundance at equilibrium in '000s 
        states= [27 32 37 42];
        action_nodes= [3,4,5,6,7,8];
        tot_nodes= 13;
    case 'red_knot_r'
        flyway_eq= 57;  %flyway abundance at equilibrium in '000s
        states= [30 37 44 51];
        action_nodes= [2,3,4,5,6,7,8];
        tot_nodes= 13;
    case 'terek_sandpiper'
        flyway_eq= 50;  %flyway abundance at equilibrium in '000s 
        states= [25 30 35 40];
        action_nodes= [2,3,4,5,6];
        tot_nodes= 11;        
end

