
modified version of [Engima's Traffic script v1.30](https://forums.bistudio.com/topic/186976-engima39s-traffic-script-release/) for Arma3

# DESCRIPTION

Engima's Traffic is a set of scripts that adds traffic to an Arma 3 mission.

Vehicles of different types are spawned in out of sight for the player(s). They get a waypoint to a random road segment
on the map, but are removed at a certain distance from the nearest player.

Script can run different instances of traffic, with different customized behaviors, simultaneously. For example, you
can run one script that puts a lot of civilians on the entire map, and one more that simultaneously control a few enemy armored
vehicles in one area and friendly vehicles in another (see USING MORE THAN ONE INSTANCE below).

Script works in singleplayer, multiplayer, hosted, dedicated for JIPs, and on any map (at least official if it has roads).


# CUSTOMIZATION

## Config

The script can be customized for different behaviors. Customize a script by creating a mission config class "EngimaTraffic" in `description.ext`

This is optional, this is what the default values would amount to:

```sqf
class EngimaTraffic {
    vehicleSets[] = {"A3"}; // RDS_CIV, RHS_GREF may be added if you've gothose mods
    VEHICLES = []; // vehicle class names, default to everything included in A3
    SIDE = civilian;
    VEHICLES_COUNT = 10;
    MIN_SPAWN_DISTANCE = 800;
    MAX_SPAWN_DISTANCE = 1200;
    MIN_SKILL = 0.3;
    MAX_SKILL = 0.7;
    AREA_MARKER = "";
    HIDE_AREA_MARKER = true; // FIXME there's no boolean in Config
    ON_SPAWN_CALLBACK = {}; // FIXME we need
    ON_REMOVE_CALLBACK = {};
    DEBUG = false;
};
```

Here is a complete list of the parameters and what they do:

* SIDE (Side): Which side the spawned vehicles will be. Can be east, west, independent or civilian.

* VEHICLES (Array): Array of vehicle classes that may be spawned. If you want to see more of one vehicle than another,
  then have it occur a couple of more often in the array. The following example will spawn traffic where 75% of vehicles
  are quadbikes and 25% is transports:
  Example: ["C_Quadbike_01_F", "C_Quadbike_01_F", "C_Quadbike_01_F", "C_Van_01_transport_F"]

* VEHICLES_COUNT (Number): Number of vehicles that exists on the map for the current traffic instance.
  Example: If VEHICLES_COUNT is set to 10 and MAX_SPAWN_DISTANCE is set to 1000, then there will be 10 vehicles on an area
  of 3142 square meters (1000 * pi).

* MIN_SPAWN_DISTANCE (Number): Minimum spawn distance in meters from nearest human player on the map. Should be at least 100
  meters less than MAX_SPAWN_DISTANCE.
  Example: 800

* MAX_SPAWN_DISTANCE (Number): Maximum spawn distance in meters from nearest player on the map. Vehicles beyond this
  distance will be removed. Should be at least 100 meters greater than MIN_SPAWN_DISTANCE.
  Example: 1200

* MIN_SKILL (Number): Vehicle crew's minimum skill. Must be between 0 and 1 and less than or equal to MAX_SKILL. Actual
  skill of each spawned vehicle (and crew) will be a random number between MIN_SKILL and MAX_SKILL.
  Example: 0.4

* MAX_SKILL (Number): Vehicle crew's maximum skill. Must be between 0 and 1 and greater than or equal to MIN_SKILL. Actual
  skill of each spawned vehicle (and crew) will be a random number between MIN_SKILL and MAX_SKILL.
  Example: 0.6

* AREA_MARKER (String): Name of a marker that sets bounds for the traffic. The marker needs to be of shape rectancle or
  ellipse (not icon for obvious reasons), and it needs to contains road segments. All vehicles for the current traffic
  will spawn inside this area, and all waypoints set to these vehicles will also be inside the area. However, it is Arma
  that routes the vehicle to the destination, and so it can come to decide to use roads that are outside of the marker. Be
  aware of this when you are planning the marker positions. Default value is an empty string ("") which means "the entire
  map".

* HIDE_AREA_MARKER (Boolean): Wether the area marker should be hidden or not. If true then the marker will be hidden on
  the map for the players, otherwise it will be visible. Default value is true.

* ON_SPAWN_CALLBACK (Code): Code that is executed when a vehicle has spawned. Parameter _this is an array: 0: created
  vehicle (Object), 1: all crew (Array of Objects), 2: vehicle's group (Group). Default value is {}.
  Example: { hint ("Vehicle of type " + typeOf (_this select 0) + " created!")


* ON_REMOVE_CALLBACK (Code): Code that is executed just before a vehicle is removed. Vehicle is sent in as a parameter _this.
  Default value is {}.
  Example: { hint "Vehicle of type " + typename _this + " removed!"; }

* DEBUG (Boolean): Whether script is running in debug mode or not. In debug mode all vehicles are marked as dots on the map.
  Can be true or false. Default value is false.

## Event Handlers

event handlers may be pushed to:
* the array ENGIMA_TRAFFIC_spawnHandler - will be called on spawning civs, with the unit as first parameter
* the array ENGIMA_TRAFFIC_vehicleSpawnHandler - will be called on spawning cars, with the vehicle as first parameter

## USING MORE THAN ONE INSTANCE

To excecute more instances of the script simultaneously, create a CBA hash with parameters like in the Config, and spawn
`ENGIMA_TRAFFIC_startTraffic` several times - all in all within two seconds (!)
Otherwise the map's road segments will not be initialzied as
needed.
