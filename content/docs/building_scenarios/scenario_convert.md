---
title: Scenario Convert
linktitle: Scenario Convert
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false
menu:
  docs:
    parent: building_scenarios
    weight: 2
    
gallery_item:
  - album: gallery
    image: osm_uncleaned.png
    caption: Uncleaned OSM-file
  - album: gallery
    image: osm_cleaned.png
    caption: Cleaned OSM-File
---

{{% alert note %}}
The tool **scenario-convert** is available in MOSAIC Extended and can be used for free for research and academical purposes.
{{% /alert %}}

Scenario-convert is a useful tool, which can be used to create new scenarios or import and export data from
external sources like OpenStreetMap, SUMO etc into your existing scenarios.  
It will create a database, which is the basis for all map-related tasks performed by Eclipse MOSAIC (e.g. navigation,
route calculation...).  
Based on a MOSAIC database, scenario-convert can export the data to SUMO-conform formats.
Furthermore one can choose, whether to use routes generated by `scenario-convert`, use existing
routes or  build own routes via route generation tools (e.g. DUAROUTER by SUMO).

This chapter intends to highlight the most common workflows for the work with `scenario-convert`.
We will be using <a href="/docs/building_scenarios/files/steglitz.osm" download>`this`</a> OSM-file for most of the
use cases So feel free to follow along with the steps to get a better understanding on how the `scenario-convert`-script functions.
For a complete reference of the script please check [here](#reference-documentation-for-scenario-convert).

{{< figure src="../images/osm_uncleaned.png" title="OSM-File of Steglitz" numbered="true" >}}

## Creating a complete Eclipse MOSAIC-scenario from an OSM-file with one command

This is the most straight forward way to create a scenario from your OSM-file.
We will use the option `--osm2mosaic`, which is a combination of the options `--osm2db`
and `--db2mosaic`.  
Let's start off by showing you how a complete call could look like:

```bash
java -jar scenario-convert.jar --osm2mosaic -i steglitz.osm
```

{{% alert note %}}
Change `mosaic.version` to the current version you are using.  
In this section we use the scenario name `steglitz.*` as synonym for `path/to/steglitz.*`.
{{% /alert %}}

This achieves a couple of things. First off the script is going to create a SQLite-database,
which is used by Eclipse MOSAIC. Furthermore, a directory will be created, which should look like this:

```FOLDER
└─ <working-directory>
   ├─ steglitz.osm
   ├─ application
   |  └─ steglitz.db
   ├─ cell2
   |  ├─ cell2_config.json
   |  ├─ network.json
   |  └─ regions.json
   ├─ environment
   |  └─ environment_config.json
   ├─ mapping
   |  └─ mapping_config.json
   ├─ ns3
   |  ├─ ns3_config.json
   |  └─ ns3_federate_config.xml
   ├─ omnetpp
   |  ├─ omnetpp_config.json
   |  └─ omnetpp.ini      
   ├─ output
   |  └─ output_config.xml
   ├─ sns
   |  └─ sns_config.json
   ├─ sumo
   |  ├─ steglitz.net.xml
   |  └─ steglitz.sumocfg
   └─ scenario_config.json .................. Basic configuration of the simulation scenario
```

Let's walk through all these files:
1. First the `steglitz.db` will be created using the `steglitz.osm`-file.
2. The `steglitz.db` will be used to create `steglitz.con.xml`, `steglitz.edg.xml` and `steglitz.nod.xml`, which are files used by SUMO.
3. [SUMO Netconvert](https://sumo.dlr.de/wiki/NETCONVERT) is used to create `steglitz.net.xml`, which is a [network representation](https://sumo.dlr.de/wiki/Networks/SUMO_Road_Networks) in SUMO.
4. Now the SUMO-configuration `steglitz.sumo.cfg` is written.
5. Afterwards the `mapping_config.json` and `scenario_config.json` are created and all files are moved to the right place.
In the `scenario_config.json` values like the center coordinate will automatically be set using data from the SUMO related files.

While this is technically sufficient to start working on your scenario there are a couple of other things
you can do to achieve better results.

__Clean the OSM-file using Osmosis__  
Osmosis will automatically be used to create a new OSM-file with the suffix `_cleaned`. The created
file will contain much less clutter and usually is better suited for simulation purposes.
Check the images below to see the difference the clean-up process can make.

```bash
java -jar scenario-convert.jar --osm2mosaic -i steglitz.osm
```
<div class="row">
  {{< figure src="../images/osm_uncleaned.png" title="Uncleaned OSM-file" numbered="true" >}}
  {{< figure src="../images/osm_cleaned.png" title="Cleaned OSM-file" numbered="true" >}}
</div>
<div class="row">
  {{< figure src="../images/netfile_uncleaned.png" title="Uncleaned Net-file" numbered="true" >}}
  {{< figure src="../images/netfile_cleaned.png" title="Cleaned Net-file" numbered="true" >}}
</div>

{{< gallery album="gallery" >}}
To avoid "cleaning" the OSM-file, please use the option "skip-osm-filter".
```bash
java -jar scenario-convert.jar --osm2mosaic -i steglitz.osm --skip-osm-filter
```

{{% todo %}}Modify 'gallery' shortcode to display images correctly{{% /todo %}}

{{< gallery album="cleaning-maps" >}}

__Generating Routes__  
The scenario-convert also offers the option `--generate-routes`, which will generate
a route-file, given some additional information. For example purposes we will run the
following command:
```bash
java -jar scenario-convert.jar --osm2mosaic -i steglitz.osm --generate-routes
--route-begin-latlon 52.4551693,13.3193474 --route-end-latlon 52.4643101,13.3206834 --number-of-routes 3
```
This will calculate three routes between the two given coordinates. 

Alternatively you can use the following command in order to generate routes with node-id's as start and end point, which can be found in the `steglitz.nod.xml` file.

```bash
java -jar scenario-convert.jar --osm2mosaic -i steglitz.osm -o --generate-routes
--route-begin-node-id 267350668 --route-end-node-id 313139970 --number-of-routes 3
```
see [below](#reference-documentation-for-scenario-convert) for all command 
line options.

__Exporting Traffic Lights__  
Another feature of the scenario-convert script is the ability to export traffic lights from the osm-file to
be used by SUMOs netconvert. The extended call would look like this:
```bash
java -jar scenario-convert.jar --osm2mosaic -i steglitz.osm --generate-routes
--route-begin-latlon 52.4551693,13.3193474 --route-end-latlon 52.4643101,13.3206834 --number-of-routes 3
--export-traffic-lights
```

__Conlusion__  
This wraps up one of the main workflows with the scenario-convert-script.
A quick reminder on what we achieved:
- Cleaned up an OSM-file to only contain relevant data.
- Converted that OSM-file to formats that Eclipse MOSAIC/SUMO can handle.
- Created the project structure for a scenario.
- Calculated routes between two coordinates.

With all of this you can now start further developing your scenario. For a more detailed description on the next steps
please have a look [here (Simulation Scenarios)](/docs/building_scenarios/scenarios/) and 
[here (Application Development)](/docs/building_scenarios/application_development/).  
While this is the 'happy world' workflow it is often necessary to manually adapt routes and
insert them into your scenario. The following workflow
will explain how that is done and you will also get a more detailed overview of the scenario-convert-functions.

## Importing Routes to your scenario

As mentioned above your routes won't always be created by the scenario-convert script. We generated some routes 
for the steglitz-scenario using SUMO's [duarouter](http://sumo.dlr.de/wiki/DUAROUTER), which you can find
[here](/docs/building_scenarios/files/steglitz.rou.xml). We'll start with only the `steglitz.osm` and 
`steglitz.rou.xml` files:

```FOLDER
└─ <working-directory>
   ├─ steglitz.osm
   └─ steglitz.rou.xml
```

__Creating the database__  
We'll start off by solely creating the database and applying OSMOSIS with the following command:
```bash
java -jar scenario-convert.jar --osm2db -i steglitz.osm
```
The directory should look like this:

```FOLDER
└─ <working-directory>
   ├─ steglitz.db
   ├─ steglitz.osm
   ├─ steglitz.rou.xml
   └─ steglitz_cleaned.osm
```

__Importing the route-file__  
> This is the interesting part of this workflow. :thumbsup:

Let's import our routes into the database.  
We achieve this by calling:
```
java -jar scenario-convert.jar --sumo2db -i steglitz.rou.xml -d .\steglitz.db
```
Now all new routes are imported into our database. The following image shows a visualization of one of
the created routes.

{{< figure src="../images/steglitz_route.png" title="Visualization of one of the routes" numbered="true" >}}

__Creating the scenario__  
The final step is to create the scenario from the files we created so far.

```
java -jar scenario-convert.jar --db2mosaic -d .\steglitz.db
```
Instead of copying our SUMO-files this will generate all necessary files and move them into a Eclipse MOSAIC-conform
folder structure:

```FOLDER
└─ <working-directory>
   ├─ steglitz.osm
   └─ steglitz
      ├─ application
      |  └─ steglitz.db
      ├─ mapping
      |  └─ mapping_config.json
      ├─ sumo
      |  ├─ steglitz.net.xml
      |  └─ steglitz.sumocfg
      └─ scenario_config.json
```

As you can see the resulting folder structure looks just like the final output from the first workflow described.

__Conclusion__  
You should now know how you can manually add routes to your scenario and have a deeper understanding of the way that
some of the script parameters work.

### Attached Files
A list of all attached files in this chapter:
- <a href="/docs/building_scenarios/files/steglitz.osm" download>Steglitz OSM-file</a>
- <a href="/docs/building_scenarios/files/steglitz.rou.xml" download>Steglitz Route-file</a>

## Reference documentation for scenario-convert
### Usage of scenario-convert

The following listing shows an overview for the usage of scenario-convert:

{{% todo %}}New shortcode for embedding documents{{% /todo %}}

<a href="/docs/building_scenarios/files/ScenarioConvertFunctions.txt" download>scenarioConvertFunctions</a>

### Configuration-files for scenario-convert

Scenario-convert offers a way to safe your conversion-parameters in a `JSON` configuration file using
the option `-c` or `--config-file`.  
The following listing shows how to save the options used in the example above:

<a href="/docs/building_scenarios/files/steglitz_config.json" download>steglitzConfigFile</a>

### Speed-files
Below you can find a properties file which can be used during the import of OSM data
in order to define speeds for ways, which do not have a maxspeeds-tag defined. For this purpose use the
option `--osm-speeds-file <FILE>`. In the speed properties file, for each way type a speed value can
be defined, according to the OSM [`highway`](http://wiki.openstreetmap.org/wiki/Key:highway) key.

<a href="/docs/building_scenarios/files/car-speeds.properties" download>speed-properties</a>
