# Adding your rooms to the game

When you finish editing your room and press the "Save UAsset" at the top you should get two files in `assets/`: YourRoom.uasset and YourRoom.uexp. These files are the room assets that need to be packed into a mod for the game to be able to see your custom room. However, making the room is only the first step: you will need to modify a bunch of other files for the game to even know your room exists.

**This section is a lot easier to understand if you have modded the game before, but I will try my best.**

## Choosing the room behaviour in your mod

For every mission type, you have to decide:

* Do you want your custom rooms to be mixed with the vanilla rooms?
* Or do you want your custom rooms to be exclusive, or in other words, guaranteed to appear?

Let's see each option separately.

## Custom rooms mixed with vanilla rooms 
This is the easiest option, as it doesn't require to modify the DNA files.

### The Tag System

To decide which rooms to spawn on each mission the game uses a system of tags, or names. Each vanilla room has one or more tags associated to it. Another file called the DNA file (located in `Content/Content/GameElements/Missions`, where you will find one file per every mission type) has a list of tags to look for when building the mission map. For example, the file for a 225 morkite mission is 'DNA_2_03.uasset'. If you open it with UAssetGUI or a similar program you will see that it contains a query with three tags:

1. Rooms.Linear.Small to choose the first room 
2. Rooms.Linear.Medium to choose the second room 
3. Rooms.Linear.Small to choose the third room

Morkite missions are the easiest to understand as they are linear and the tags that appear in the DNA file match the rooms as in the first tag corresponds to the first room, the second tag to the second room, etc. Other mission types are not so straightforward to understand because we can see which tags the game uses but not HOW they are used. 

The following spreadsheet contains all the tags used by every mission type: [DRG Mission Tags](https://docs.google.com/spreadsheets/d/e/2PACX-1vSrIEAbg8iWAKqYcxuMYua-lhWncY8UWThzlHun5L2CQ9oNeZt9AzvHHO7xRRDmHY51jN16MnnDerua/pubhtml).

Once you know the tags relevant to the mission type you're interested in, you will need to add the corresponding tags to your rooms with the editor and save the uassets. For example, if you tag your room `Rooms.Linear.Small`, your room will be available in the smaller morkite missions, 4 and 6 eggs, medium complexity salvages, etc. To make playability better, I recommend being honest with the size of the rooms and trying to follow the `Small`, `Medium` and `Big` system used by the devs in the vanilla game. 

#### Special cases: PE and starting rooms

Point Extraction doesn't use the tag system and instead it has a list of valid rooms in its PLS file:

* `Content/Landscape/ProceduralLevelSetups/Alpha02/PLS_Motherlode_Star.uasset`.

so if you want your room to appear in PE, edit the list of rooms you can find inside this file. PE maps are made of a single room, so I recommend making special maps for PE only.

#### Special cases: start rooms

The first room in some missions like Eggs or Mining is not picked with the tag system. Instead, their PLS files have a list of valid start rooms. If you want to add rooms also to the start of missions, you will have to edit their corresponding PLS files.

### The RMG file

Once your rooms have the correct vanilla tags the rooms need to be added to the room pool by modifying a special file inside `Content/Maps/Rooms/RoomsGroups/EarlyAccess`: `RMG_ExtractionLinear.uasset`. This file contains a list of all existing room names and their paths, usually `Content/Maps/Rooms/RoomGenerators`. If you only have a handful of rooms you can edit the file manually with UAssetGUI. For bigger changes, I created [a small Python CLI tool](https://github.com/vonacht/drg-rmg-patcher/) that can be used, please see the instructions in the repository.

### The mod structure 
Once you have your room uassets with vanilla tags and the patched RMG file, you can create a mod pak with the following structure:

```
Content/
    Maps/
        Rooms/
            RoomGenerators/
                Room01.uasset
                Room01.uexp 
                Room02.uasset
                Room02.uexp
                ...
            RoomsGroups/
                EarlyAccess/
                    RMG_ExtractionLinear.uasset
                    RMG_ExtractionLinear.uexp
```

where `RMG_ExtractionLinear.uasset` is your patched RMG file that contains entries for Room01, Room02, etc. If you have edited the PLS files for PE or Deep Scan they need to be included also with their corresponding folder structure. To pak the mod you can use any tool you want, I personally use [repak](https://github.com/trumank/repak).

## Rooms exclusive to a mission type

If you don't want vanilla rooms to appear alongside your custom rooms, you will need to create custom tags and substitute the vanilla tags in the DNA files. The step explained before about modifying the RMG file is still needed.

### Modifying the DNA files with custom tags 

As we saw before, the game uses a tag system to decide which rooms to spawn in each mission type. A certain morkite mission might use `Rooms.Linear.Small` and `Rooms.Linear.Medium`: if you can modify these tags to, say, `Rooms.Linear.CustomSmall` and `Rooms.Linear.CustomMedium` and you create a few rooms with these tags, the mission will only pick from your custom rooms as there will be no vanilla room tagged with these. 

To modify the DNA files, you can just change the tags inside the TagDictionaries with UAssetGUI. Similar to the RMG patcher, I have a [simple CLI tool](https://github.com/vonacht/drg-dna-patcher) that helps automating it. Once you have your patched DNA files, the structure of your mod will look like:

```
Content/
    GameElements/
        Missions/
            DNA_2_01.uasset
            DNA_2_01.uexp
            DNA_2_02.uasset
            DNA_2_02.uexp
            ...
    Maps/
        Rooms/
            RoomGenerators/
                Room01.uasset
                Room01.uexp 
                Room02.uasset
                Room02.uexp
                ...
            RoomsGroups/
                EarlyAccess/
                    RMG_ExtractionLinear.uasset
                    RMG_ExtractionLinear.uexp
```
