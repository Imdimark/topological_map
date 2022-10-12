# Interact With an Ontology Through the aRMOR Service
**An ROS-based tutorial for the Experimental Robotics Laboratory course held at the University of Genoa**  
Author: *Luca Buoncompagni*

---

## Introduction

This tutorial shows how to use the `topological_map.owl` ontology in ROS. Such an ontology 
represents indoor locations and a mobile robot with surveillance purposes.

You can find such an ontology in this repository, where the `topological_map_box.owl` is a copy 
of the `topological_map.owl` file but without the Abox, i.e., without some individuals 
representing a specific environment for showing purposes. You might want to open the 
`topological_map_abox.owl` ontology with the Protégé editor (see installation below) for 
knowing better its structure, while the `topological_map.owl` ontology should be used by the 
robot within a ROS architecture.

You can know more about OWL ontology through [this](https://drive.google.com/file/d/1A3Y8T6nIfXQ_UQOpCAr_HFSCwpTqELeP/view) 
tutorial.

## A Very Brief OWL-DL Premier

An OWL ontology is based on the Description Logic (DL) formalism, and it encompass three sets 
made by entities:
 - the TBox encodes a tree of *classes*,
 - the RBox encodes a tree of *objects* or *data properties*, 
 - the ABox encodes a set of *individuals*. 
For simplicity, we will refer to a class with a capitalized name (e.g., `LOCATION`), to a data 
or object property with camel case notation (e.g., `isIn`), while individuals will have the first
letter capitalized (e.g., `Robot1`).
 
More in particular, the individuals are related within each other through object properties, and 
with literals (e.g., integer, strings, etc.) through data properties. 

Individuals can be classified into some classes, which are hierarchical arranged in an 
implication tree, i.e., if an individual belongs to a class, such an individual would always  belongs to its super-classes as well. In other words, the child class always implies its parents.

Object and data properties has a domain and a range, and they can be specified in the definitions
of the classes for classifying individuals.

Note that the OWL formalism relies on the *Open Word Assumption*, which states that an unknown 
information is not necessary false. Also, be aware that two entities (i.e., classes, individuals
or properties) might be equivalent between each other if not explicitly stated as *disjoint*.

Given some logic axioms in the ontology, a *reasoner* can be invoked to infer implicit knowledge.
We will also exploit SWRL rules, which are if-then statements. Since not all reasoners can 
process SWRL rules, we will use the Pellet reasoner. Also, since OWL reasoning is a time 
computing task, you should invoke it only if strictly required.

## Software Tools

Download the [Protégé](https://protege.stanford.edu/) for your operative system, and unzip it in 
you home folder. Open a terminal, go into the unzipped folder, and open Protégé by entering `./
run.sh`. When Proégé starts, a plugins installation popup should appear (that popup can always 
be found at `File -> Check for Plugins`). From such a popup select `Pellet Reasoner Plugin` and 
`SWRL Tab Protege 5.0+ Plugin`. Then, install them and restart Protégé. When Protégé is running
again, go in `window -> tabs` end enable the `SWRLTab`. Now you can open an ontology, update the 
reasoner from the dedicated tab (on top of the window), and see the results.

To use an OWL ontology and the related reasoner within ROS, we will use the [aRMOR](https://github.com/EmaroLab/armor). 
If necessary, follow the instruction to install it in your workspace , and also check the 
[commands](https://github.com/EmaroLab/armor/blob/master/commands.md) 
documentation for knowing how to use aRMOR. In addition, you might want to check the 
[armor_py_api](https://github.com/EmaroLab/armor_py_api), which simplifies the calls to aRMOR,
but it can only be used from a python-based ROS node. 

## An Ontology for Topological Maps

...picture class

The ontology provided in this repository encodes the classes showed in the picture above, where 
each `LOCATION` can be a `ROOM`, if it has only one `DOOR`, and a `CORRIDOR`, if it has more 
doors. Each door is associated to a location with the object property `hasDoor`. In addition, 
each `LOCATION` has the data property `visitedAt` which represent the more recent timestamp (in 
seconds) when the robot visited such a location.

...picture robot

The `ROBOT` class contains only one individual (i.e., `Robot1`), which specifies some 
properties, as shown in the figure above. In particular, the `isIn` property specifies in which 
`LOCATION` the robot is, while the `now` property specify the last time stemp (in seconds) when
the robot change location. In addition, the property `urgencyThreshold` represents a parameter
to identify `URGENT` location to be visited (see more details below).

A new `LOCATION` can be defined by creating a new individual with some properties `hasDoor`, 
which are related to other individuals that might be created to represent doors. The `hasDoor` 
relation allows the reasoner to infer each individual of type `DOOR` and `LOCATION`, which can 
be of sub-type `ROOM` and `CORRIDOR`. For instance, the example shown in the environment 
described into the `topological_map.owl` can be seen in the figure below.

...picture environment

The ontology also implement three SWRL rules to infer 
 - the locations that are connected to another location through a door,
 - the locations that the robot can reached by passing through a door,
 - the locations that the robot should visit urgently, i.e., it did not visit them for a number 
   of seconds greater than the value associated to the `urgencyThreshold` property.

After having invoked the reasoner, it infers several logic axioms, and the more interesting for
our example are:
 - properties `canReach`, which are generated to represent the locations that a robot can reach,
 - the type of each `LOCATION`, which can be a `ROOM` and, eventually, a `CORRIDOR` as well
   (note that a `CORRIDOR` is represented as a specific type of `ROOM`, i.e, an individual of 
   type `CORRIDOR` is always of type `ROOM` as well).
 - locations of type `URGENT`, i.e., the robot should visit them urgently.

Such a data representation should be used in order to make the robot visiting a location, and 
stay there for some time before to move to another location. In particular, the visiting policy 
should be such that the robot mainly moves across corridors but, when a near location becomes
*urgent*, then it should go visiting it.

Please check the `topological_map_abox.owl` file with Protégé to see how the concepts above are 
implemented in an OWL ontology. You can also start the Pellet reasoner to see the its results 
(highlighted in yellow in Protégé) and change the individuals to appreciate the differences.

## Use the Ontology within a ROS-based Architecture

### The aRMOR service

We exploit the aRMOR server for using OWL ontologies in a ROS architecture. The first time you
run this server you should use the following commands to create a ROS executable based on Java.
```bash
roscd armor
./gradlew deployApp
```

If you have a `roscore` running, you can launch the aRMOR server with 
```bash
rosrun armor execute it.emarolab.armor.ARMORMainService
```

Note, that if you want to run aRMOR from launch file you should use the following configuration.
```xml
<node pkg="armor" type="execute" name="armor_service" args="it.emarolab.armor.ARMORMainService"/>
```

aRMOR can perform three types of operations: manipulations, queries, and ontology management.
Ontology management involves several utilities, such as loading or saving an ontology as well as
invoke the reasoner, etc. Manipulations are operations that change the logic axioms asserted in 
an ontology, e.g., add an individual, replace an object property, removing a class, etc. 
Finally, queries allow retrieving knowledge from the ontology, e.g., know the classes in which 
an individual belongs to, the properties applied to an individual, etc. 

By default, the entities (i.e., classes, individuals, properties) involved in an 
manipulation are created if not already available in the ontology. Also, by default, queries 
involve the knowledge inferred by the reasoner, which should have been synchronized with recent 
manipulations, if any.

aRMOR share the same message to perform each operation, namely each request involves the 
following fields.
 - `client_name`: It is an optional identifier used to synchronize different client's requests. 
   In particular, when a client with a certain name requests an aRMOR operation, the request of 
   another client with the same name is refiused with an error.
 - `reference_name`: It is the name of an ontology to work with.
 - `command`: It specifies the command to execute e.g., `ADD`, `LOAD`, `QUERY`, etc.
 - `primary_command_spec`: It is a specifier of the given `command` and it can be optional, 
    e.g. IND, FILE, etc.
 - `secondary_command_spec`: It is a further specifier of the given `command`, and it can be 
    optional.
 - `args`: It is a list of arguments used to perform the given `command` with the `primary` and 
   `secondary` `specifier`, e.g. the name of a class, a list of individuals, etc.

Please, check the [documentation](https://github.com/EmaroLab/armor/blob/master/commands.md) to 
see the possible `command` and the relative `primary_command_spec`, `secondary_command_spec`, 
and `args`.

On the other hand, the response of aRMOR involve the following fields.
 - `success`: It is a boolean specifying if the service was successfully computed.
 - `exit_code`: It is an integer specifying the possible source of errors.
 - `error_description`: It a string specifying a possible error.
 - `is_consistent`: It is a boolean specifying if the ontology is consistent.
 - `timeout`: It is an optional filed use while performing SPARQL queries.
 - `queried_objects`: It is an optional list of strings representing the queried entities.
 - `sparql_queried_objects` It is an optional list of QueryItem (key-value pairs) given as the
   result of a SPARQL query.

Please, see the aRMOR [package](https://github.com/EmaroLab/armor) for more documentation.

For showing purposes, the following sections assume that aRMOR is running, and they make request
directly from the terminal, i.e., by using `rosservice call ...` commands.

### Load an Ontology

Make the following request to load an ontology from file.
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'LOAD'
  primary_command_spec: 'FILE'
  secondary_command_spec: ''
  args: ['<path_to_file>/topological_map.owl', 'http://bnc/exp-rob-lab/2022-23', 'true', 'PELLET', 'false']"
```

Where we set a `client_name` which will be shared among all the clients since we do not need to
synchronize them. Also, we set a `reference_name` that will be used to refer alwasys to the same
ontology loaded on memoty.

Then, we configure the command such to load from file with the following `args`.
 1. The path to the file containing the ontology to load.
 2. The IRI, which is specified into the ontology itself.
 3. A Boolean set to `true` to specify that the manipulation should be automatically applied. If 
    `false`, then the `APPLY` command should be explicitly state to apply some manipulations.
 4. The name of the reasoner to use.
 5. A Boolean set to `true` to specify that the reasoning process should be explicitly stated. If
    `false`, then the reasoning process automatically runs after each manipulation; this would 
    lead to an expensive computation.

Note that, in this example, the 1st and 2nd fields of the request above do not changes in the 
message below.

### Add a Location

An individual `L` that represents a new location can be added in the ontology if it is paired 
with one or more doors `D` through the object property `hasDoor`. This can be done with the 
following request made for each door of `L`.
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'ADD'
  primary_command_spec: 'OBJECTPROP'
  secondary_command_spec: 'IND'
  args: ['hasDoor', 'L', 'D']"
```

When all locations (e.g., `L1....Ln`) and doors (e.g., `D1...Dm`) has been added, it is 
important to set them as different individuals, which can be done with the following request.
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'DISJOINT'
  primary_command_spec: 'IND'
  secondary_command_spec: ''
  args: ['L1', 'L2', ..., 'Ln', 'D1', 'D2', ..., 'Dm']" 
```

These logic axioms are enough to make the reasoning inferring that `D` is a `DOOR`, and if `L` is
a `ROOM` or a `CORRIDOR`.

### Robot's Movements 

The `ROBOT` named `Robot1` is always in a location, and this should be consistently represented
in the ontology with through the `isIn` property, over time. To represent the location of the
robot (e.g., `Robot1 isIn L1`) you should use the following request.
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'ADD'
  primary_command_spec: 'OBJECTPROP'
  secondary_command_spec: 'IND'
  args: ['isIn', 'Robot1', 'L1']" 
```

Since the location of the robot should be unique, when the robot moves to another location 
(e.g., `L2`) we should remove the old `isIn` value. This can be done through the command
```
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'REPLACE'
  primary_command_spec: 'OBJECTPROP'
  secondary_command_spec: 'IND'
  args: ['isIn', 'Robot1',  'L2', 'L1']" 
```

### Timestamps Updating

The instances representing the robot and locations are associated with a Unix timestamp, 
respectively through the `now` and `visitedAt` data properties, which are used to compute the 
urgency. 

Such a timestamps might be represented in seconds, and they should be unique for each 
individuals. Therefore, when a timestamp it is introduced for the fist time, the `ADD` command 
should be used. Then, similarly to above, the `REPLACE` command should be used for updating such 
a timestamp.

Note that the `REPLACE` command also requires to specify the old value, which might be retrieved
from the ontology. In order to do so we should invoke the reasoner first, with the request
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'REASON'
  primary_command_spec: ''
  secondary_command_spec: ''
  args: ['']"
```

Then, we can query the timestamp associated with the robot, which represents the last time the robot changed location. Hence, use the following request, which returns `1665579740`.
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'QUERY'
  primary_command_spec: 'DATAPROP'
  secondary_command_spec: 'IND'
  args: ['now', 'Robot1']" 
```

Now, we can update the timestamp associated with the robot (e.g., to `1665579750`) with:
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'REPLACE'
  primary_command_spec: 'DATAPROP'
  secondary_command_spec: 'IND'
  args: ['now', 'Robot1', 'Long', '1665579750', '1665579740']" 
```

This procedure concerns the logic axiom involving the `Robot1` individual, the `now` data 
property and a `Long` timestamp, and it should be done each times the robot moves to a new 
location. The same procedure should also be used for each individuals of type 
`LOCATION`, when the robot enter in that specific location. For instance, let `Robot1` go in the
location `L` at timestamp `1665579750`, then the `visitedAt` property of `L` should be replaced
through the following request; where `1665579710` is the lat time `Robot1` was in `L`.
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'REPLACE'
  primary_command_spec: 'DATAPROP'
  secondary_command_spec: 'IND'
  args: ['visitedAt', 'L', 'Long', '1665579750', '1665579710']" 
```

### Retrieving Inferred Knowledge Through Queries

To retrieve inferred knowledge, update the reasoner with the `REASON` command above if some 
manipulation has been from the last time the reasoner has been updated. The query all the 
`canReach` value associated with `Robot1`, i.e.,
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'QUERY'
  primary_command_spec: 'OBJECTPROP'
  secondary_command_spec: 'IND'
  args: ['canReach', 'Robot1']" 
```
Which would return a set of locations that the robot can reach by passing through a `DOOR`, e.g., `L1`, `L2`, etc.

Than, we can query information about these locations, e.g., 
```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'QUERY'
  primary_command_spec: 'CLASS'
  secondary_command_spec: 'IND'
  args: ['L1', 'false']" 
```
Which returns a set of classes that might involve `ROOM`, `CORRIDOR`, and `URGENT`.

### Save the Ontology

For debugging purposes, you can save the ontology manipulated so far in order to open it with
Protégé and inspect its contents. Tho save an ontology use

```bash
rosservice call /armor_interface_srv "armor_request:
  client_name: 'example'
  reference_name: 'ontoRef'
  command: 'SAVE'
  primary_command_spec: ''
  secondary_command_spec: ''
  args: ['/root/Desktop/topological_map2.owl']" 
```

---
