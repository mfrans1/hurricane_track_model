=========================================================
Introductory Information
=========================================================

Title: STORM IBTrACS present climate synthetic tropical cyclone tracks

Description: Each seperate .txt-file contains 1,000 years of synthetic tropical cyclone tracks, generated using the STORM algorithm.

Format: for every array (time step of a tropical cyclone), the entries are comma-seperated. 

File naming: The files are named STORM_IBTRACS_PRESENT_CLIMATE_(Basin ID)_1000_YEARS_(index).txt. The Basin IDs can be found in the description. "index" ranges from 0-9. 

Contact Information: nadia.bloemendaal@vu.nl

=========================================================
Methodological Information
=========================================================
Method description: the synthetic tracks were generated using the fully statistical STORM algorithm (see Bloemendaal et al, Generation of a Global Synthetic Tropical Cyclone Hazard Dataset 
using STORM, in review). Further details on the algorithm can be found in the manuscript. 

Software: All data was produced in Python 3.6.1 on the SURFsara Lisa Computer Cluster. The file format of the datasets has been chosen such that they can be opened and 
analyzed in a wide variety of computer programs.


Validation: The dataset has been validated in the manuscript. 

=========================================================
Data Specific Information
=========================================================

Column names: The entries of each of the .txt-files is as follows (copied from Bloemendaal et al - Generation of a Global Synthetic Tropical Cyclone Hazard Dataset using STORM),
for every time step of a tropical cyclone:


Entry	Variable name			Unit		Notes on variable
1	Year				-		Starts at 0
2	Month				-	
3	TC number			-		For every year; starts at 0. 
4	Time step			3-hourly	For every TC; starts at 0.
5	Basin ID			-		0=EP, 1=NA, 2=NI, 3=SI, 4=SP, 5=WP
6	Latitude			Deg		Position of the eye.
7	Longitude			Deg		Position of the eye. Ranges from 0-360°, with prime meridian at Greenwich.
8	Minimum pressure		hPa	
9	Maximum wind speed		m/s	
10	Radius to maximum winds		km	
11	Category			-		.
12	Landfall			-		0= no landfall, 1= landfall
13	Distance to land		km	
