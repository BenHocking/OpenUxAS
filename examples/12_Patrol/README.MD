#	Collision Dectection in UxAS using NASA's DAIDALUS code

This example is intended to informally test UxAS integration with NASA's DAIDALUS code.

This example simulates a patrol scenario that consists of three line searches.  One line search is performed around a polygon of intereset.  The other two line searches can be intepreted as surveilling two sides of a boundary.  This spacing, however, results in the entities/vehicles tasked with performing these two line searches being on a collision course. This directory contains a script that launches an instance of AMASE to drive simulation of the scenario. The script also launches three instances of UxAS, each configured for one of the three entities.   

## Input Files
* `cfgPatrol_1000.xml` - UxAS configuration file for Entity 1000
* `cfgPatrol_2000.xml` - UxAS configuration file for Entity 2000
* `cfgPatrol_4208.xml` - UxAS configuration file for Entity 4208
* `MessagesToSend/Patrol.xml` - AMASE configuration file for scenario
* `runUxAS_Patrol.sh` - script that launches AMASE and UxAS instances for the scenario

## Running the Example
1. Open a terminal window in this directory `12_Patrol`
2. Run the command `sh runUxAS_Patrol.sh`
3. Press play in AMASE after three additional terminals running UxAS open

## Notes
1. The entity/vehicle configuration files may need to be edited to correctly identify the network device on line 9.
2. Three terminals will open after AMASE launches. Each runs an instance of UxAS configured for one of the three entities. Each instance of UxAS runs the DAIDALUS_Processing service. This service runs DAIDALUS, implements avoidance maneuvers, and prints information on DAIDALUS processing as it runs.
3. UxAS will log messages sent to AMASE, including messages sent from the DAIDALUS_Processing. For each entity, these messages are stored in a db3 SQLite database file within a subdirectory named according to entity number, e.g. 09_Collision/UAV_1000/datawork/SavedMessages/messageLog_1_0.db3. To view the database file, we recommend DB Browser for SQLite. On most Linux distributions, you can get this browser by executing in a terminal window the commands `sudo apt-get update` and `sudo apt-get install sqlitebrowser`.
4. The current script file assumes a version of AMASE that includes a DAIDALUS plugin to diplay visualizations of the guidance processing bands generated by the DAIDALUS code. To get this version, build AMASE from the daidalus_integration branch of the AMASE repository (https://github.com/afrl-rq/OpenAMASE).
5. This is a work in progress and is subject to change.

## Things to try
1. Changing the configuration will change the way the vehicles respond to detections of projected loss of wellclear.
	-See DAIDALUS_Processing.h for changable parameters and default values.
	-These parameters can be changed for each instance of UxAS in the corresponding XML configuration file.
