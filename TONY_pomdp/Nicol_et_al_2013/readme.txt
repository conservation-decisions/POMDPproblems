# please refer to https://sites.google.com/site/adaptivemanagementeaa/home
# we have copied across the problems but the website provides more information

README_CASSANDRA	

Provides an overview of the structure of the Cassandra input files included with this data challenge

Author:

Sam Nicol Sam.Nicol@csiro.au

-----------------------------------------
CONTENTS:
* POMDP input files (CASSANDRA FORMAT)
* how to generate the files from Symbolic Perseus

-----------------------------------------	
POMDP input files
-----------------------------------------
The following problems are provided with this documentation:

grey-tailed_tattler_Cassandra.txt
terek_sandpiper_Cassandra.txt
bar-tailed_godwit_m_Cassandra.txt
bar-tailed_godwit_b_Cassandra.txt
lesser_sand_plover_Cassandra.txt

Files can be downloaded from:
https://sites.google.com/site/adaptivemanagementeaa/downloads/pomdp_input_files

Files were converted to Cassandra format automatically from the symbolic Perseus input files.
Networks with 8-9 nodes (i.e. red_knot_p, red_knot_r, curlew_sandpiper, great_knot, far_eastern_curlew) had 
state spaces that were too large to convert and caused out of memory errors on our machine. For more information,
refer to the readme files for symbolic Perseus, available at: https://cs.uwaterloo.ca/~ppoupart/software.html.

Generating Cassandra formatted files from Symbolic Perseus
----------------------------------------------------------
Pascal Poupart provides a MATLAB function to convert input files from Symbolic Perseus to Cassandra format.
His function, convertPOMDP2CassandraFormat.m, assumes that all variables are observable. Because the EAA 
flyway problem contains a hidden model variable, the convertPOMDP2CassandraFormat has to be edited in order
to run. The edits are minor but the m-file can be downloaded from: 
https://sites.google.com/site/adaptivemanagementeaa/downloads/pomdp_input_files
The edited m-file is called convertPOMDP2CassandraFormat_MO.m. It takes the same input and returns the 
same output as convertPOMDP2CassandraFormat.m. 

Large networks could not be generated as Cassandra input files because they caused out of memory errors. 
From the help files for symbolic Perseus:
	"Note that the Cassandra format requires the enumeration of all states
	(flat format) and therefore may not be suitable for large problems.
	Depending on the amount of RAM available it may not be possible to
	convert problems with more than 10,000 states."
