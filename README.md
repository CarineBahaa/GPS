# GPS Module

We need to understand how a GPS module works and how to use it to successfully navigate the vehicle to the 3 waypoints during the autonomous phase.

## What is GPS? 

GPS (Global Positioning System) is a navigating system based on receiving signals from satellites that orbit Earth. 

It’s important not to confuse GPS with GNSS, GNSS(Global Navigation satellite system) is the general term for different navigation systems, while GPS is only a type of GNSS.
GPS is the United States’ satellite system, but there are other multiple systems that exist such as BeiDou, Galileo, etc…
This satellite system (constellation) consists of 24 satellites that are separated in 6 planes, each with 4 satellites.

*** 

## How does a GPS module work?

 A **GPS receiver** located in the module receives signals from 3 different satellites to produce a location, and from a 4th one to validate the information sent from the other three satellites. This is also called a lock or a fix.
A signal sent by a satellite consists of :
 a **timestamp** (when the signal was sent), and data on **where this satellite is located in the sky**.

GPS works through a technique called **"Trilateration"**

>   this is the process where the GPS receiver considers the satellite as the center of a sphere, then it calculates the distance traveled by the signal from the satellite to the receiver, the receiver then proceeds to find the intersection point of the 3 spheres to calculate its location.

### How does it calculate the distance between itself and the satellite?

How can we use trilateration if we can't physically measure the distance from the satellite to the module?
The trick lies in the fact that satellites are always sending out signals.

In GPS positioning, the rate is how fast the radio signal travels, which is equal to the speed of light (299,792,458 meters per second). Time is determined by how long it takes for a signal to travel from the satellite to a GPS receiver on earth (using timestamp of the signal).
With a known rate and a known time we can calculate the distance between satellite and receiver 
```text
Distance = time × speed
```

*** 

## GPS Outputs and Data Formats

We need to understand what values the module is going to give us and their formats to know how to use those values in our task (navigating to the 3 way points).
Data is displayed in a Serial terminal using UART protocol, a standard GPS module can output latitude, longitude, altitude, timestamp, etc..

Nearly all GPS receivers output data using a format called NMEA.
**ΝMEΑ (National Marine Electronics Association)**

>   is a common data format that most GPS modules use. NMEA data is displayed using something called a "**sentence**".
The NMEA sentence contains several fields of data that are separated by commas to make it easier to read and parse by computers and microcontrollers.

This data is sent out on the serial port at an interval called the "**update rate**", most receivers update this information once per second (1Hz), but more advanced receivers are capable of multiple updates per second.

There are different types of NMEA messages that a module can output : GGA, GLL, GSA, RAC, etc..
 Let's take for example a **GAA sentence (Global Positioning System Fix Data)**  to clearly understand the structure of a sentence : 

```text
$$GPGGA,181908.00,3404.7041778,N,07044.3966270,
W,4,13,1.00,495.144,M,29.200,M,0.10,0000*40
```

-181908.00 is the time stamp: time in hours, minutes and seconds.

-3404.7041778 is the latitude:  in the DDMM.MMMMM format (degrees.decimal minutes)

-N denotes north latitude.

-07044.3966270 is the longitude in the DDDMM.MMMMM format (degrees.decimal minutes)

-W denotes west longitude.

-4 denotes the Quality Indicator:
+0 =  Fix not available or invalid
+1 = Uncorrected coordinate
+2 = Differentially correct coordinate (e.g., WAAS, DGPS)
+4 = RTK Fix coordinate (centimeter precision)
+5 = RTK Float (decimeter precision

-13 denotes number of satellites viewed by the module

-1.0 denotes the HDOP (horizontal dilution of precision)

-495.144 denotes altitude of the antenna

-M denotes units of altitude (eg. meters or feet)

-29.200 denotes the geoidal separation 

-M denotes the units used by the geoidal separation

-1.0 denotes the age of the differential correction (if exists)

-0000 denotes the differential correction station ID (if exists) 

-40 denotes the checksum

*** 

## How to use those outputs to navigate the vehicle?

There are two things we need to calculate: the **distance** between our current position and the set waypoint, and the turning **angle** needed to face the waypoint.

There are two ways we can calculate the distance and the needed angle:

1. Haversine Formula
2. Euclidean Geometry

***

### Haversine Formula

Used to calculate the great circle distance between two points (the shortest path between 2 points on a sphere).

Because of the spherical shape of the Earth, it's more useful and accurate than Euclidean geometry over **large distances**.

#### Haversine Formula

Given:

* Current position:

  * Latitude: $\phi_1$
  * Longitude: $\lambda_1$

* Target waypoint:

  * Latitude: $\phi_2$
  * Longitude: $\lambda_2$

First, convert all latitude and longitude values from degrees to radians.

Calculate:

```math
\Delta \phi = \phi_2 - \phi_1
```

```math
\Delta \lambda = \lambda_2 - \lambda_1
```

Then calculate:

```math
a = \sin^2\left(\frac{\Delta \phi}{2}\right)
+ \cos(\phi_1)\cos(\phi_2)\sin^2\left(\frac{\Delta \lambda}{2}\right)
```

```math
c = 2\operatorname{atan2}(\sqrt{a},\sqrt{1-a})
```

The distance between the current position and the waypoint is:

```math
d = R \cdot c
```

Where:

* `d` = distance between the two points (meters)
* `R` = Earth's radius ≈ 6,371,000 m
* `c` = angular distance between the two points

To calculate the waypoint angle (initial bearing):

```math
y = \sin(\Delta \lambda)\cos(\phi_2)
```

```math
x = \cos(\phi_1)\sin(\phi_2)
-
\sin(\phi_1)\cos(\phi_2)\cos(\Delta \lambda)
```

```math
\theta = \operatorname{atan2}(y,x)
```

Convert the result to degrees and normalize it to the range:

```math
0^\circ \le \theta < 360^\circ
```

This angle represents the direction from the current position to the waypoint.

***

### Euclidean Geometry

#### Euclidean Formula

For relatively short distances, we can approximate the Earth as a flat plane and convert latitude and longitude into local Cartesian coordinates `(x,y)`.

First calculate the differences:

```math
\Delta \text{lat} = \text{lat}_2 - \text{lat}_1
```

```math
\Delta \text{lon} = \text{lon}_2 - \text{lon}_1
```

Convert them to meters:

```math
x = \Delta \text{lon} \cdot \cos(\text{lat}_{avg}) \cdot R
```

```math
y = \Delta \text{lat} \cdot R
```

Where:

```math
\text{lat}_{avg} = \frac{\text{lat}_1 + \text{lat}_2}{2}
```

and all latitude and longitude values are expressed in radians.

The distance is then:

```math
d = \sqrt{x^2+y^2}
```

The waypoint angle is:

```math
\theta = \operatorname{atan2}(y,x)
```

Where:

* `d` = distance between the current position and waypoint
* `x` = East-West displacement
* `y` = North-South displacement
* `θ` = direction toward the waypoint

***

### Distance Error and Angle Error
Using either way, this information will be used to calculate:

1. **distance error**: the difference between the distance from the set waypoint with respect to the origin point, and the distance from the current position with respect to the origin point.

2. **angle error**: the difference between the waypoint angle and the current angle of the robot.

Those variables will be given to **PID** to control our motion going to the waypoint.

> **Important Note:** there can be two ways to calculate the waypoint angle:
>
> * **ENU (East-North-Up):** used by mathematicians, robotics, and ROS, the axes map to Earth as X=East, Y=North, and Z=Up, where the angle is calculated **counterclockwise**.
>
> * **NED (North-East-Down):** used by aviation, marine, and some GNSS packages, the axes map to X=North, Y=East, and Z=Down, where the angle is calculated **clockwise**.
