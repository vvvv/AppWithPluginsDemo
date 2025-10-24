# PluginHostDemo
Demonstrates how to build an application in VL that can host plugins.

## Overview
- The main application: `myApp.vl` references `host.vl`. This is what is being exported to deploy the app.
- A separate document for developing plugins: `myApp-plugindevelop.vl`. The same as above but: Also references any plugin .vl documents directly to work on them
- The core app: `host.vl`: Must not be referenced by plugins. In this example it will show a combo box to select the plugin which should run.
- Plugin Interfaces: `Layer-plugins.vl`, `TexFX-plugins.vl`, `Audio-plugins.vl`

The `\plugins` folder with example plugins:
- `pluginA`, `pluginB` contain Skia Layer plugins
- `Ana` simple TextureFX
- `Posterize` a TextureFX with custom shaders
- `TexFXWithAssets` a TextureFX with an asset folder
- `TAL-NoiseMaker` an Audio plugin that uses a VST instrument

## Running the demo
- Export `myApp.vl` is intended to be exported. The resulting executable will look beside its `plugins` folder for plugin dlls.
- Open the individual plugins from the `\plugins` folder and export them 
  - Notice how they are set-up to export into `MY_REPOS\VL.PluginHostDemo\exported\myApp\plugins`
  - Notice how `TexFXWithAssets` includes a .targets file that makes sure the assets folder is also copied over

Sidenote: Check the [vvvv commandline compiler](https://thegraybook.vvvv.org/reference/hde/exporting.html#the-commandline-compiler) to simplify your deployment scenario.

## Guidelines
- Keep your interfaces as minimal as possible. Make sure they only reference packages that they actually need. This helps to keep the output folder small. If you have multiple plugin interfaces using different technologies think about placing them in seperate VL files.
- Don't add a reference to an exported plugin.dll to your `myApp-plugindevelop` environment. Because this plugin.dll would require a specific interfaces.dll which during dev time is not available.
- Don't mix vvvv versions: APIs provided by vvvv and used by your patches could change in between versions and break compatibility. You most likely will see `MissingMethodException` or `TypeLoadException` in case that happens.
- Don't mix package versions across plugins. Build a list of versions every plugin should work with.
- You may consider stripping plugins from duplicated dlls, but be aware: if you know the host application, it should be safe to remove all dlls the host already provides. But in case of multiple hosts this needs to be coordinated.

### Dealing with paths
Imagine the following project structure:
```
src/myPlugin/myPlugin.vl
src/myPlugin/assets/foo.jpg
```

When exported the resulting structure would be:
```
bin/myApp/myApp.exe
bin/myApp/plugins/myPlugin/myPlugin.dll
bin/myApp/plugins/myPlugin/assets/foo.jpg
```

Or in case plugins end up in USER wide storage it could be:
```
USER/AppData/Local/myCompany-plugins/myPlugin/myPlugin.dll
USER/AppData/Local/myCompany-plugins/myPlugin/assets/foo.jpg
```
We have a node called `PatchPath` which always points `myPlugin` and can be used to build file paths.

## Known Issues
- A plugin needs to have a dummy entry point in order to show up in the export dialog. A simple comment in the application patch is enough.
- Untested: A plugin that makes use of native dll

## Future work
Explore loading the plugins via a separate [assembly load context ](https://learn.microsoft.com/en-us/dotnet/core/dependency-loading/understanding-assemblyloadcontext). This could allow updating and removing plugins at runtime in the exported application. And it could allow different plugins to use different versions of the same package

## Related Discussions
This is a slightly modified version as was outlined [in the forum](http://forum.vvvv.org/t/export-and-load-dlls-from-gamma/22278/4).

### Instructions for vvvv dev build (for devvvvs only)
Compile vvvv itself with
```
cd MY_REPOS\vvvv
build EditorAndPackages
```
Compile myApp.vl with
```
cd bin\win-x64\vvvv_gamma_MY_VERSION
vvvvc MY_REPOS\VL.PluginHostDemo\myApp.vl --export-package-sources MY_REPOS\vvvv\bin\win-x64\packages
```
