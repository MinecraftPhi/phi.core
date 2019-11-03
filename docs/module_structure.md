# The Phi Module Packer
The Phi Module Packer is designed to help automate cross version compatibility and dependency management, as well as performing other processing on the module before it can be run.

## Folder Structure
Modules follow this folder structure:
```
├── src
│   ├── bundled
│   │   ├── datapacks
│   │   └── resourcepacks
│   ├── datapack
│   └── resourcepack
├── dist
│   └── output name
│       ├── datapack
│       └── resourcepack
└── module.json
```

## `module.json`
This file defines the metadata for the module.
```json5
{
    "id": "unique namespaced id",
    "name": "human readable name",
    "description": "short description",
    "version": "semver",
    "internalFolder": "folder name",
    "load": [
        "namespaced function/function tag id"
    ],
    "tick": [
        "namespaced function/function tag id"
    ],
    "preprocessors": [
        "pre-defined preprocessor id",
        {
            "custom": false,
            "id": "pre-defined preprocessor id",
            "params": {
                // Preprocessor specific parameters
            },
            "outputs": {
                "output name": "folder name", // "output name" is variable
                "output name": {
                    "folder": "folder name",
                    "dependencies": [
                        // See "dependencies" in the root
                    ],
                    "optionalDependencies": [
                        // See "optionalDependencies" in the root
                    ],
                    "preprocessors": [
                        // Recursive structure
                    ]
                }
            }
        },
        {
            "custom": true,
            "name": "human readable name",
            "url": "link to custom preprocessor"
        }
    ],
    "dependencies": [
        {
            "type": "module",
            "id": "module id",
            "version": "semver"
        },
        {
            "type": "datapack",
            "source": "", // "git" or "file"
            "url": "url"
        },
        {
            "type": "resourcepack",
            "source": "", // "git" or "file"
            "url": "url"
        },
        {
            "type": "mod",
            "name": "Human readable mod name",
            "url": "link to mod"
        }
    ],
    "optionalDependencies": [
        {
            "type:":"", // Same as a normal dependency
            // ...
            "description": "Description of what changes if this dependency is found"
        }
    ]
}
```

## Pre-Processing
The `"preprocessors"` list defines what preprocessors need to be run on the `src` folder before the resource/data pack can be run by minecraft.

Currently there are no pre-defined preprocessors, however there are plans for a Phi transpiler and custom language. Pre-defined preprocessors will be able to be run automatically by the Phi module packer.

Custom pre-processors can't be run by the Phi module packer, but are still included in the metadata to indicate that the user needs to run it manually, and to prevent the packer from creating an invalid module distribution.

The preprocessors are run in order from top the bottom, and the outputs are put into folders inside the `dist` folder named according to `"outputs"`  
If `"outputs"` is not defined the preprocessor defines the default output folder(s). When the `"preprocessors"` list is empty the output goes directly into the `dist` folder. Most preprocessors use this same default for a single output.

If a preprocessor creates multiple outputs the nested `"preprocessors"` list is run on the associated output, and then any preprocessors after this preprocessor will run on each output individually, with the output being nested inside the output folder.

An example preprocessor with multiple outputs would be one that outputs a version of the pack using only vanilla features, and a version that takes advantage of mod commands to improve performance or add new functionality.

## Dependencies
The `"dependencies"` list specifies the required dependencies of the module.

There are multiple types of supported dependencies:
- `"type": "module"`  
  A dependency on another module following the Phi module format. The Phi module packer will be able to automatically download and bundle the latest compatible version of the module from the Phi website. This bundling will also attempt to automatically add the version check for that module to the load function(s).
- `"type": "datapack"` and `"type": "resourcepack"`  
  Bundles a plain datapack or resource pack from a URL. The url can either point to a git repository, or a zip file, based on the `"source"` key
- `"type": "mod"`  
  Specifies that the module depends on a mod to minecraft. For example, the module may depend on the additions to the command system made by Phi-Modded

The nested `"dependencies"` list in preprocessor outputs merges with the root list. In the case of two of the same dependency existing, the latest of the two specified versions is used.

The `"optionalDependencies"` list follows almost the same format as `"dependencies"`, but the dependencies are not automatically bundled, and with the addition of a description that explains what happens if the dependency is also loaded.

## Automatic Versioning and Dependency Management
The `"load"` and `"tick"` lists act as replacements for the load and tick function tags, with the added functionality of automatically performing the version checks and declarations before running the specified functions.

Any datapack elements in folders that do not follow the folder structure for versioning they will be automatically converted to match the version number and perform the necessary checks.  
Files inside the folder specified by `"internalFolder"` will be considered not part of the public API and will still be moved to the versioned folders, but will not perform any version checks and will not generate the function tags.