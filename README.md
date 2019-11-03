# phi.core
This repository contains the core module for Phi, as well as defines the standards that Phi follows and that all Phi modules and packs that use Phi should follow.

- [Versioning](docs/versioning.md)

# Objectives
- phiglobal
    - Stores global state
    - DO NOT CLEAR
    - Typical usage is on fakeplayers
- phiversion
    - Stores version numbers
    - Completely cleared on load then written with version numbers of loaded modules
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
`$` is the preferred prefix, however if a fakeplayer value changes frequently it may be desirable to hide it from the sidebar. In this case, `#` may be used instead of `$`
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