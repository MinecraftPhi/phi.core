# phi.core
The core module of Phi.

# Objectives
- phiglobal
    - Stores global state
    - DO NOT CLEAR
    - Typical usage is on fakeplayers
- phiversion
    - Stores version numbers
    - Completely cleared on load then written with version numbers of loaded modules
    - Only needed during load
- phitemp
    - Stores temporary data
    - There is no guarantee that values will be kept beyond the current function context
    - Typical usage is on fakeplayers
- phiconst
    - stores numerical constants
    - Typical usage is on fakeplayers with the same name as the value, however this is not required and a normal fakeplayer name is also allowed
    - e.g.
    ```
    #load
    scoreboard players set 2 phiconst 2

    #usage
    scoreboard players operation ... *= 2 phiconst
    ```

# Standards
## Fakeplayer
When using a fakeplayer to store data, either temporarily or permanently, the fakeplayer name must be "namespaced" in order to prevent conflicts.

The format of a fakeplayer follows this pattern: `$<namespace>.<name>` where:
- `$` is used to ensure the name is an invalid real player name while still being a valid fakeplayer name.  
`#` is not used because it hides the fakeplayer from the sidebar, which is undesirable for debugging purposes
- `<namespace>` is the namespace. This will typically be the same as the namespace being used by the datapack
- `.` is just a separator between the segments of the name
- `.<name>` is a `.` separate name of the fakeplayer.  
For `phitemp` the name typically follows `.<function>.<name>` where `<function>` is the name of the function. This prevents conflicts between fakeplayers in the function and in the chain of caller functions.

Example: `$phi.rng.seed` is in the `phi` namespace, and the name is `rng.seed`.  
This can also be seen as being in the `phi.rng` namespace, and with a name of `seed`. Both options are valid, and this ambiguity does not matter.

## Storage
When using `storage` for storing arbitrary NBT the namespaced id should have only the namespace with no key, which can be done by writing the namespace followed by a colon with nothing after it. e.g. `phi.core:`  
The reasoning for this is two-fold:
- This puts the storage for every pack into separate files, making removal of an individual pack easier
- Keys in the namespaced ID cannot be removed from within game. By moving the key into the path instead of the ID allows for removal using `data remove`

## Entity Tags
Entity tags (`/tag`) should also be namespaced to prevent conflicts. The namespace is separated from the tag name with a `.`

# Versioning and Dependency Management
Phi uses [Lantern Load](https://github.com/LanternMC/Load) for version handling and dependency management. Please read the documentation for Lantern Load before using Phi. (once it has been written)

Version numbers in Phi follow [Semantic Versioning](https://semver.org/), and on load the latest version of each module that has been loaded will be stored in the `phiversion` objective on the following fakeplayers:
- `$phi.<module>.version.major`: Major version number, incremented when breaking changes are made.  
This includes **any** changes to existing datapack elements apart from functions
- `$phi.<module>.version.minor`: Minor version number, incremented when new features are added, reset when the major version is incremented.  
Adding any new (public/documented) datapack elements counts as adding a new feature.
- `$phi.<module>.version.patch`: Patch version number, incremented when bug fixes are applied, reset when the minor or major version is incremented.  
Unfortunately, this is only possible for functions, by using function tags and version score checks

Note that the version numbers are incremented assuming that packs do not depend on internal/undocumented features.

Note: the following should be moved to Lantern Load documentation
> Packs/modules should not put their tick functions in the `#minecraft:tick` tag, and should instead schedule their tick functions on load using `schedule` and to reschedule at the end of their tick functions.  
This solves two problems:
> - The vanilla `#minecraft:tick` tag runs **before** `#minecraft:load`, and as such the loading process will not be complete when the tick function runs for the first time after loading
> - `#minecraft:tick` always runs all its functions every tick, with no way to disable a function apart from wrapping it in a function that checks a condition. This wastes performance when a tick function is not needed.
>
> Using `schedule` for tick functions is also useful for version handling. It is recommended that on load the scheduled tick function(s) should be cleared, and then the version numbers of all dependencies should be compared for compatibility with the loading pack, and only if the dependencies are compatible should the tick function(s) be scheduled.  
This has the effect of disabling the pack when incompatible depenedencies are loaded, or if the dependencies are not loaded at all.
>
>Under normal circumstandces, as per Semantic versioning, a module is compatible if the `major` version is equal to the expected version, and the `minor` and `patch` versions are greater than or equal to the expected version.

## Creating a pack with versioning
When creating a pack it must follow the constraints described in the Lantern Load documentation.

The pack must also ensure that the correct version numbers are assigned to the approriate fakeplayers. When loading, the version stored on the fakeplayers should be checked, and if the version of the pack is later than the version on the fakeplayers the version should be overwritten.

### Functions
To prevent conflicts between versions, function paths should be made such that the namespaced id follows this pattern:
```
<namespace>:<major>/<minor>/<patch>/<name>
```
There should also be a function tag the follows the following pattern: `#<namepace>:<major>/<minor>/<name>`  
This tag should call the function for the latest patch as of the time of writing the tag. Previous patch versions would add previous versions of the function.  
This tag should also be added to the the tag for the previous minor version

Overall this means that all loaded versions of the function that are compatible with the specified version will run, so the function should check that the version is the exact version so that only the latest will do anything.

The point of having tags at the minor version level is to reduce the number of functions that need to run, since `#<namepace>:<major>/<minor>/<name>` will only contain that version and later versions.

#### Example
A function called `func` for version 2.3.4 of a pack called `pack` would have the following:

`datapacks/pack/data/pack/functions/2/3/4/func.mcfunction`:
```
execute if score $pack.version.major matches 2 if score $pack.version.minor matches 3 if score $pack.version.patch matches 4 run ...
```

`datapacks/pack/data/pack/tags/functions/2/3/func.json`:
```json
{
    "replace": false,
    "values": [
        "pack:2/3/4/func"
    ]
}
```

`datapacks/pack/data/pack/tags/functions/2/2/func.json`:
```json
{
    "replace": false,
    "values": [
        "#pack:2/3/func"
    ]
}
```

Now any of the following will result in the same function being called: `pack:2/3/4/func`, `#pack:2/3/func`, `#pack:2/2/func`.  
If a later version is also loaded the tags will cause the later version to be called, and if an earlier version is also loaded the older tags will get the newer tags added to them, resulting in the newer version running even when the older tags are run.

## Phi version 0
Phi version 0 was written before Lantern Load was conceived, and as such does not follow the versioning standards. Version 0 is deprecated, and new features will not be added to it, however critical bug fixes will be back-ported if applicable. There is also no way to check how recent the loaded version of Phi version 0 actually is

It is highly discouraged to depend on version 0, however if you wish to you still can, but checking for version 0.0.0 will not work as the fakeplayer will not exist in that case. To check for version 0 you need to instead do:
```
execute unless score $phi.core.version.major phiversion matches 1..
```