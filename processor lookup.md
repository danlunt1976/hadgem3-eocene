Identify geographical domain based on processor ID
- from Charlie: PEs are ordered left-to-right from the bottom-left (the SW corner of the grid to the NE corner) according to domain decomposition, so check number of PEs in each row and column respectively 
- can be found in GUI at `> suite conf > Domain Decomposition > Atmosphere`
### example for [[cf290]] crash:
- crash in atmosphere with 36 East-West and 28 North-South processors
- this means we have 36 PE columns and 28 rows with a total of 1008 processors:

	| | | | | | | |
	|-|-|-|-|-|-|-|
	|972|973|974|975|976|...|1007|
	|...|...|...|...|...|...|...|
	|72|73|74|75|76|...|107|
	|36|37|38|39|40|...|71|
	|0|1|2|3|4|...|35|
- need to find exact x,y location of PE in this table and then convert this indices to a longitude/latitude based on grid spacing
- grid spacing can be calculated by 360 divided by number of East-West processes and 180 divided by number of North-South processes, i.e. 360 / 36 = 10 and 180 / 28 = 6.4
- grid starts at 0E/W,-90S, so use this to get longitude/latitude of the SW corner of PE
- error from processor 475, i.e. in row 14 ($=475/36+1$) and column 8 ($=475-36*13+1$)
- this means we have x=8 and y=14 and can calculate $lon = (x-1) * dx = (8-1)*10 = 70E$ and $lat = -90 + (y-1) * dy = -90 + (14-1)*180/28 = 6S$
