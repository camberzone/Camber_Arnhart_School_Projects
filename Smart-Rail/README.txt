## Smart-Rail
Smart-Rail project for CS351

## Program information

This program allows the user to control a train system composed of tracks, trains, stations, switches, lights, and buffered stops. The user can choose from 6 pre-made rail layouts (detailed below), or they can create their own layout by editing one of the configuration files found in the resources file. Once the configuration file is loaded, the user can create trains by selecting origin and destination stations.

The game is initialized in Main, where the object manager and GUI controller are created. Once the user selects their desired configuration file from the starting screen, the file is loaded into the object manager where it creates and starts all the threads corresponding to each object in the rail. These threads communicate with their designated neighbors (see Communication), as well as sending messages to the GUI controller and object manager.

Routing: The routing for trains is comprised of the four-method recursion (described in Communication), and is activated by the GUI-ObjectManager relationship directing any specified station to dispatch a train. Only one reservation request may be active at a time by means of a lock variable stored in the Manager in order to avoid any race conditions or deadlock, and while routing the algorithm will pause upon encountering an already-reserved track in order to ensure a possible route is not ignored. The base aggressive recursive algorithm is naturally slow in some instances, depending on the number of possible branches and the order in which they are checked, and always prefers a straight pathing in order to reduce the likelihood of obtuse routing.

Communication: The communication of this program as dictated by two sectors, the GUI-Manager relationship overhead, and the active Messenger objects comprising the rail lines and trains. Each train and component of the rail (per specification) operates on its own thread, only knowing its own information, the addresses of its neighbors, and communicating via message passing. Basic Messengers (Train) can only force a neighbor to change their state as a preemptive measure, which the neighbor will respond to on their own thread, ReserveableMessengers ("RM"s, rail components) utilize a four-part "handle in-flag/flag-in next, wait/handle out-flag/flag-out previous, wake previous" system to alert an RM that it has to handle an incoming or outgoing call and allows it to handle that call on its own thread.


Hints and Tips:  
*The user must click and origin station and then a destination station to spawn a train  
*The user cannot select the same station to be both the origin and destination station. (This will not break the game. An error message will be shown in the error message box)  
*If the train cannot reach the destination station from the origin station, the train will not spawn. (This will not break the game. An error message will be shown in the error message box)  

## Pre-Made Config File Descriptions

config1: General/Full demonstration of all parts (Stations, Tracks, Lights, Switches, Buffered Stops)

config2: Lowest Level demonstrations: 1. "empty" line, 2. Short line, 3. Max length line, 4-5. Basic switch, 6-7. Switch with lights, 8 Station to buffered stop (no dispatches can be made on this rail as there is only one station)

config3: Switch demonstrations, see routing under program information for explanation of algorithm

config4: Buffered Stop demonstrations, note that buffered stops cannot be chosen as destinations, and so only affect routing in such situations as 5->4 and 2->1, or 1->(2|3) and 4->(3|5)

config5: Light demonstration, note in-line lights never have need of being red, only those bordering switches for route protection

config6: Left with an intentionally basic usage for ease of grader editing

## How to Customize Your Own Config File

# Character-to-Object Key
'S' = station
'b' = buffered stop
'=' = track
's', followed by an ID number between 1-99 (ex: 's1', 's2', 's42') = switch
'l' = light

# Example of a config file:
S = l s1 l  = =  S
S = l s1 l s2 l  = = S
S = = =  l s2 l s3 l = l s4 l S
S = = =  = =  l s3 l = l s4 l = = l s5 l = = = =  S
b = = =  = =  = =  = = = =  = = = l s5 l = = l s6 l = = = S
S = = =  = =  = =  = = l s8 l = = = =  = = = l s6 l = S
S = = =  l s7 l =  = = l s8 l = = = =  = = = = =  = = = = S
S = = =  l s7 l =  = = = =  = = = = =  = = = = =  = b

# Config file rules:
* Characters can be separated by spaces. This will not affect the contents of the rails.
* Each line (a rail) must begin and end with a station OR a buffered stop. A buffered stop acts as an end to the rail, and the train cannnot travel to, or be dispatched from, it.


* No line can contain a station or buffered stop in the middle of the line.

             BAD            GOOD
  ex:   S = = S = = b    S = = = = b


* Lights are not allowed to be placed next to stations as stations would not dispatch a train into a situation where they would be forced to immediately stop.

             BAD            GOOD
  ex:   S l = = = = b    S = = = = b


* Switches must have an ID number after the character 's'. The ID number allows the config scanner to see which switches are paired. Two switches with the same ID number are considered paired.

            BAD               GOOD
 ex:   S = l s1 l = b    S = l s1 l = b
 ex:   S = l s2 l = b    S = l s1 l = b
 
 
* Switches must always have a pair

            BAD               GOOD
 ex:   S = l =  l = b    S = l s1 l = b
 ex:   S = l s1 l = b    S = l s1 l = b


* Switches must always have a light, station, or buffered stop as their right and left neighbors. These represent route safety, multi-rail exits, and curves, respectively 

            BAD               GOOD             GOOD          GOOD
 ex:   S = = s1 = = b    S = l s1 l = b    S s1 l = = b     S s1 S
 ex:   S = = s1 = = b    S = l s1 l = b    S s1 l = = S     S s1 b
 
 
* The same ID number cannot be used for more than one pair of switches

            BAD               GOOD
 ex:   S s1 l = l s1 b    S s1 l = l s2 b
 ex:   S s1 l = l s1 b    S s1 l = l s2 b


* With default window size, each rail cannot contain more than 30 items. This is calculated based on the width of the window.
* With default window size, there cannot be more than 8 rails. This is calculated based on the height of the window.

## Debugging Options

In GameConstants, there are three debug options:
*Set PRINT_OBJECTS to true in order to print debugging information regarding the Messenger components. Importantly: ?><M> representing an incoming request call, !(T|F)><M> representing an incoming response call
*Set PRINT_GUI to true in order to print debug information regarding the GUI  
*Set PRINT_THREAD_NOTICE to true in order to print Thread.wake() and Thread.notify() calls, which are wrapped internally to allow for debugging

## Credits

Code is written by:  
*Kage Wiess
  - Designed and created base logic and most code for all rail component objects, as well as the organization hierarchy Messenger->Reserveable-><Objects>
  - Created all aspects of messaging and routing for rail components
  - Created project-representative config files, acted as canary

*Camber Arnhart  
  - Created all aspects of the GUI, both art and logic. 
  - Created configScanner, designed format of config files, and code to create/start objects from config file
  - Contributed code and design details for rail component objects (train, station, switch, etc)

All art assets created by Camber Arnhart

## Bugs
Attempting to route from a station that is in the process of routing will cause it to wait until it exceeds the allowed waiting period, and then dispatches the train. This train is dispatched along the original route, but with the subsequent destination as its goal, and so does not send a return train.
This program is not designed to be run on, and may display poorly on, devices with small screens or exceedingly large screens and/or dense resolutions, but instead formatted to fit the average current display.
