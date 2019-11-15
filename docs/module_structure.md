# The Phi Module Packer
The Phi Module Packer is designed to help automate cross version compatibility and dependency management, as well as performing other processing on the module before it can be run.

## Folder Structure
Modules follow this folder structure:
```
├── src
│   └── ...
├── dist
│   └── output name
│       └── ...
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
    "preprocessors": [
        {
            "type": "verified",
            "id": "namespaced preprocessor id",
            "version": "semver",
            "params": {
                // Preprocessor specific parameters
            },
            "inputs": [
                "glob"
            ],
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
                    ],
                    "profiles": [
                        "profile name"
                    ]
                }
            },
            "profiles": [
                "profile name"
            ]
        },
        {
            "type": "nuget",
            "id": "nuget id",
            // ... same keys as a verified preprocessor
        },
        {
            "type": "local",
            "folder": "file location",
            // ... same keys as a verified preprocessor, without version
        },
        {
            "type": "custom",
            "name": "human readable name",
            "url": "url to preprocessor", // URL can point anywhere
            "profiles": [
                "profile name"
            ]
        },
        {
            "type": "split",
            "split name": { // "split name" is variable
                // Preprocessor
            },
            "split name": {
                // Preprocessor
            }
        }
    ],
    "profiles": [
        "profile name"
    ],
    "dependencies": [
        {
            "type": "module",
            "id": "module id",
            "version": "semver",
            "bundle": true
        },
        {
            "type": "datapack",
            "source": "", // "git" or "file"
            "url": "url",
            "bundle": true
        },
        {
            "type": "resourcepack",
            "source": "", // "git" or "file"
            "url": "url",
            "bundle": true
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
            "description": "Human readable description of what changes if this dependency is found"
        }
    ]
}
```

## Pre-Processing
The `"preprocessors"` list defines what preprocessors need to be run on the `src` folder before the resource/data pack can be run by minecraft.

There are 4 different types of pre-proccessors:
- `"type": "verified"`  
  A verified pre-processor has been manually verified, through source code analysis, to be safe to run. These types of pre-processors are stored on the Phi website and can be run automatically from the website or locally.  
  If a module uses only verified pre-processors it will be possible to download the module in a completely functional vanilla format from the website.
- `"type": "nuget"`  
  A nuget pre-processor is the same as a verified pre-processor, except has not yet been validated to be safe, and is stored on NuGet instead of the Phi website.  
  These pre-processors can still be run automatically by the Phi module packer on a local machine, but cannot be run on the Phi website for security reasons. Use with caution.
- `"type": "local"`  
  A local pre-processor is just like a verified or nuget pre-processor, but is stored locally on the machine rather than downloaded remotely.  
  This is designed for use during the development of a new pre-processor, and is not expected to be used in a release of any module.
- `"type": "custom"`  
  A custom pre-processor cannot be run by the Phi module packer and must be run manually by the user.  
  
  This exists for compatibility with existing tools that do not yet have a pre-processor available. If a module requires a custom pre-processor it will be shown on the website using the given name and will direct people to the specified URL. The URL can point anywhere, however it is recommended to point to the documenation of how to install and run the pre-processor
- `"type": "split"`  
  This allows for multiple independent preprocessor trees to be run. Each preprocessor tree gets the same inputs and produces outputs into folders matching the `"split name"` for that tree.  

  Examples of possible use cases:
  - Perform debugging without disrupting the flow of data
  - Only generating documentation when requested rather than on all builds
  - Automatically copying the final output to a world during development but not on final release

The preprocessors are run in order from top the bottom, and the outputs are put into folders inside the `dist` folder named according to `"outputs"`  
If `"outputs"` is not defined the preprocessor defines the default output folder(s). When the `"preprocessors"` list is empty the output goes directly into the `dist` folder. Most preprocessors use this same default for a single output.

If a preprocessor creates multiple outputs the nested `"preprocessors"` list is run on the associated output, and then any preprocessors after this preprocessor will run on each output individually, with the output being nested inside the output folder.  
Independent outputs may be processed in parallel.

An example preprocessor with multiple outputs would be one that outputs a version of the pack using only vanilla features, and a version that takes advantage of mod commands to improve performance or add new functionality.

The `"inputs"` lists specifies a list of files that the pre-processor should process, formatted as a *nix file glob relative to the root of the output of the previous pre-processor. This list is optional and the default is pre-processor defined. Any files that the pre-processor doesn't take as input will be just directly copied to the output with no processing, prior to any other files being processed so that the output of the pre-processor can replace such files without its own output being corrupted.

The `"profiles"` list specifies which profiles this preprocessor or output will run during. If the list is empty or missing then the it will run for all profiles, otherwise if the running profile is not in the `"profiles"` list, the pre-processor, all its children, and any pre-processors following it in the list, will not run.  
It is an error to specify a profile that is not listed in the global `"profiles"` list

## Standard Profiles
Profiles may have any name, but there are some profile names that are commonly used and are recommended:
- `dev` for development
- `release` for producing the final distribution
- `doc` for documentation

## Dependencies
The `"dependencies"` list specifies the required dependencies of the module.

There are multiple types of supported dependencies:
- `"type": "module"`  
  A dependency on another module following the Phi module format. The Phi module packer will be able to automatically download and bundle the latest compatible version of the module from the Phi website. This bundling will also attempt to automatically add the version check for that module to the load function(s).
- `"type": "datapack"` and `"type": "resourcepack"`  
  Bundles a plain datapack or resource pack from a URL. The url can either point to a git repository, or a zip file, based on the `"source"` key
- `"type": "mod"`  
  Specifies that the module depends on a mod to minecraft. For example, the module may depend on the additions to the command system made by Phi-Modded

The `"bundle"` boolean key determines whether the dependency will be bundled together in the final output or not. Defaults to `true`. Not applicable to `mod` dependencies or optional dependencies.

The nested `"dependencies"` list in preprocessor outputs merges with the root list. In the case of two of the same dependency existing, the latest of the two specified versions is used.

The `"optionalDependencies"` list follows almost the same format as `"dependencies"`, but the dependencies are not automatically bundled, and with the addition of a description that explains what happens if the dependency is also loaded.

## Phi Pre-Processors
### `phi:versioning` - Automatic Versioning and Dependency Management
```json5
"params": {
    "internalFolder": "folder name",
    "load": [
        "namespaced function/function tag id"
    ],
    "tick": [
        "namespaced function/function tag id"
    ]
}
```
The `"load"` and `"tick"` lists act as replacements for the load and tick function tags, with the added functionality of automatically performing the version checks and declarations before running the specified functions.  
For bundled dependencies only the major version of the dependency will be checked, as the loaded version is guaranteed to be at least at the required version but may be higher.  
For dependencies that are not bundled, the full version check will be performed.  
Optional dependencies do not perform any automatic version checks, as it is up to the pack to determine what to do when the dependency isn't loaded.

Any datapack elements in folders that do not follow the folder structure for versioning will be automatically converted to match the version number and perform the necessary checks.  
Files inside the folder specified by `"internalFolder"` will be considered not part of the public API and will still be moved to the versioned folders, but will not perform any version checks and will not generate the function tags.