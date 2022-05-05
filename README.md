## Data Extracted From AIS Signals (Baltic Sea 2020)

This repository contains supply and idle data in zones of the Baltic Sea as a result of historical vessel movements from 2020 based on automatic identification system (AIS) data provided by the Danish Maritime Authority (DMA).

### Data Description
The processed data is located within the data folder of this repository. The folder contains three different files.

1. The `supply.csv` file shows supply in tons on a weekly basis per zone. The `zone` column corresponds to the respective zone, the `calender_week` column to the calendar week and the `supply_tons` column to `the amount of supply in metric tons (generalized over all different types of cargo).
2. The `idle.csv` file shows idle times in hours on a weekly basis per zone. The `zone` column corresponds to the respective zone, the `calendar_week` column to the calendar week and the `idle_hours` column to the average idle time in hours.
3. The `zones.csv` file contains a list of all zones.
4. The `baltic_sea_coastal_line_polygons.pkl` file is a pickle file of a dataframe containing definitions of polygons for the zones found in file `zones.csv`.

### Data Processing
The supply and idle data is derived from AIS signals provided by the Danish Maritime Authority. Since the AIS signals, besides some specifications about the vessel itself, only contain information about the current geo-position of same, further preprocessing steps are necessary to calculate the supply and demand in the different zones of the Baltic Sea. The following parameters, given in AIS signals, are used for the calculations: `timestamp`, `latitude`, `longitude`, `imo`, `mmsi`, `draught`, `destination`. Below screenshot shows the zones, in which we geographically have split up the Baltic Sea. Since we are interested in the supply flow between landmass, we only consider polygons on the coastal line. To get the polygon definition, please load the `polygones.pkl` in Python using pickle.

To calculate supply and idle in zones based on AIS, following steps have been performed:
1. Get all AIS signals related to one vessel. The `imo` or `mmsi` is a unique identifier for a vessel and can be used to consider every vessel one by one.
2. The time wise ordered AIS data (use `timestamp`), more precisely, the geo-position based on `latitude` and `longitude` illustrate the route a vessel is taking.
3. To derive supply zones with its idle times, the zone in which the vessel is loading (supply zone) has to be determined. To do so, the vessels `draft` and `destination` give an indication. The point in time, i.e., the AIS signal, in which the `destination` changed compared to the previous signal, denotes the start of a new voyage. Further, if the vessels `draft` has increased, the vessel was loading cargo, thus, the vessel is in a supply zone and travels towards a new zone to discharge.
4. Using a vessel database, e.g. Vesselfinder (see below), we determine the vessels maximum capacity in metric tons, its so called deadweight (dwt).
5. Since the dwt denotes the capacity of a vessel in metric tons, we take the dwt of the vessel as the amount of cargo, the vessel was loading. Note, it may not reflect one hundred percent of reality.
6. Performing such steps for each vessel, yields voyages between supply zones including the amount of cargo transported.
7. Aggregating the data on a weekly and zone basis, gives data as provided in the `supply.csv` file.
8. To determine the idle time in a supply zone, one simply calculates the time from entering a zone until loading. Averaging idle times over all vessels for the same zone at the same time step gives the idle time reflected in the `idle.csv` file.

### Baltic Sea Zone Map

![Map](./map.PNG?raw=true "Map")

### References

- Raw AIS Data: https://dma.dk/safety-at-sea/navigational-information/ais-data
- Vessel Database: https://www.vesselfinder.com/de
