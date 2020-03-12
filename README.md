# Vehicle-Tracking-System

Running the Vehicle Tracking System 
1.	Unzip the Zip file containing solution 
2.	Open the solution in Visual studios.
3.	Restore the database DB_Tracking in MS SQL 2019 from the backup file.
4.	Update the connection file in web.Config.
5.	Run the code.

Introduction
The Vehicle Tracking System is based on onion architecture. It contains 4 architecture layers:
•	Data: At the center part of the Onion Architecture, the domain layer exists; this layer represents the business and behavior objects.
•	Repository: This layer creates an abstraction between the domain entities and business logic of an application
•	Service: This layer is used to communicate between the UI layer and repository layer.
•	WebAPI: It's the outer-most layer and keeps the Web API.

The database on MS SQL 2019. To maintain data of 10000 vehicle which will send location every 30 second. It may send 30,000,000 data for tracking every day. I used Sharding of Vehicle Tracking entity to manager this big data, to achieve better performance and fast querying. Table master_Registered_Database will manage all the sharding tables of vehicle tracking and its number of vehicle count. Every time new Vehicle request comes, it will register to the Vehicle tracking table have minimum count in table master_Registered_Database.

Working:
1)	Recording vehicle location
The Restful WebApi VehicleTrackingController accepts standard NMEA’s RMC (Recommended minimum specific GPS/Transit data) sentence to track the location appended with apiKey and Chip_SN of device. 
Example: $GPRMC,220516,A,5133.82,N,00042.24,W,173.8,231.8,130694,004.2,W*70
              1    2    3    4    5     6    7    8      9     10  11 12

1.	220516     Time Stamp
2.	A          validity - A-ok, V-invalid
3.	5133.82    current Latitude
4.	N          North/South
5.	00042.24   current Longitude
6.	W          East/West
7.	173.8      Speed in knots
8.	231.8      True course
9.	130694     Date Stamp
10.	004.2      Variation
11.	W          East/West
12.	*70        checksum

On request following validation will perform:
•	If the received RMC in sentence string have valid checksum. The checksum of RMC is an XOR of all the bytes between the $ and the * (not including the delimiters themselves), and written in hexadecimal. Checksum of RMC string is last 2 char of string after ‘*’.
Example RMC: $GPRMC,081836,A,3751.65,S,14507.36,E,000.0,360.0,130120,011.3,E*69
•	 If the received sentence have “$GPKEY,apiKey,Chip_SN”

If both conditions success the system look if the vehicle registered and get its VehicleID a and ShardMapID.
If 1st time request, Vehicle will registered on any  VehicleMaster. The shardMapID will be get from master_Registered_Database which have minimum vehicle. This is based on Sharding technique.
Parameter receivedMessage: 
•	$GPKEY,12,SN00012$GPRMC,081836,A,3751.65,S,14507.36,E,000.0,360.0,130120,011.3,E*69

For Ex: http://localhost:55532/api/VehicleTracking/$GPKEY,30,SN00030$GPRMC,123519,A,4807.038,N,01131.000,E,022.4,084.4,230120,003.1,W*67/
 
Note: As passing decimal values char ‘/’ at last is required

2)	Getting current location of vehicle

To get current location need to pass 3 parameters:
Username: admin
Password: 0000
Chip_SN (for example): 
•	SN00012,
•	SN00020
•	SN00025
•	SN00030
•	SN00030
•	Or any which is recorded before

For Ex: http://localhost:55532/api/VehicleTracking/admin;0000;SN00030 /
 









3)	Getting location of vehicle in Duration (FromDateTime- ToDateTime).
It requires 5 parameters 
•	Username: admin
•	Password: 0000
•	Chip_SN (for example): 
a.	SN00012,
b.	SN00020
c.	SN00025
d.	SN00030
e.	SN00030
f.	Or any which is recorded before
•	FromDateTime: (format: ddMMyy hh:mm:ss)
•	ToDateTime : (format: ddMMyy hh:mm:ss)
 
For Ex: http://localhost:55532/api/VehicleTracking/admin;0000;SN00030;230120%20003000;230120%20023000/

 

4)	Nearby Hotels (Google map Api). 
Nearby Places of the current place of Vehicle have four parameters:
a.	Username: admin
b.	Password: 0000
c.	Chip_SN (for example): 
i.	SN00012,
ii.	SN00020
iii.	SN00025
iv.	SN00030
v.	SN00030
vi.	Or any which is recorded before
d.	Search Keyword: Hotel, School, etc

For Ex: http://localhost:55532/api/VehicleTracking/NearByPlaces/admin;0000;SN00030;hotels/ 

 
Note: To use Google Api key, I need to add the  server IP addresses. For now, It is working for my IP.
