# PluginHostDemo
Sketches ideas how to build host with plugins in VL

This is a slightly modified version as was outlined [in the forum](http://forum.vvvv.org/t/export-and-load-dlls-from-gamma/22278/4).

The main files are
- `interfaces.vl` - this is the one file all others reference. It contains the plugin interface (called `IPlugin` in this example, which has an `Update` operation returning a Skia layer).
- `pluginA.vl` contains a class `PluginA` implementing `IPlugin` by rendering "Hello from A".
- `pluginB.vl` contains a class `PluginB` implementing `IPlugin` by rendering "Hello from B".
- `host.vl` contains host related logic and must not be referenced by plugins. In this example it will show a combo box to select the plugin which should run.
- `myApp.vl` references `host.vl` and is intended for export.
- `myApp-develop.vl` like above but also references any plugins directly to be able to work on them

## Export
On the hosting side only `myApp.vl` is intended to be exported. The resulting executable will look beside its `plugins` folder for plugin dlls.

On the plugin side the `plugin*.vl` get exported and their resulting dll needs to be manually copied to the before mentioned `plugins` folder.

To export from command line use
```
cd PROGRAM_FILES\vvvv_gamma_MY_VERSION_
vvvvc MY_REPOS\VL.PluginHostDemo\myApp.vl
vvvvc MY_REPOS\VL.PluginHostDemo\pluginA.vl
vvvvc MY_REPOS\VL.PluginHostDemo\pluginB.vl
```

## Known Issues
- You can not add a reference to an exported plugin.dll to your `myApp-develop` environment. Because this plugin.dll would require a specific interfaces.dll which during dev time is not available.
- For now `PatchPath` needs to be copied from plugin to plugin, see TODO below
- A plugin needs to have a dummy entry point
- All plugins have basic dlls duplicated in their build output folder
- Host and plugin need to be build with same version of vvvv

## Dealing with paths
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

We have a node called `PatchPath` which always points to `myPlugin` and can be used to build file paths.

TODO: In its current form however it needs to be copied from plugin to plugin to work correctly!

## Do's and Don'ts
- Do: Keep your interfaces as minimal as possible. Make sure they only reference packages that they actually need. This helps to keep the output folder small. If you have multiple plugin interfaces using different technologies think about placing them in seperate VL files.
- Don't: Do not mix vvvv versions. APIs provided by vvvv and used by your patches could change in between versions and break compatibility. You most likely will see `MissingMethodException` or `TypeLoadException` in case that happens.
- Don't: This one is a bit hard - do not mix package versions across plugins. Build a list of versions every plugin should work with. This point might get better by loading plugins in different load contexts (see below).
- If you consider stripping plugins from duplicated dlls be aware: if you know the host application it should be safe to remove all dlls the host already provides. But in case of multiple hosts this needs to be coordinated.

## Future work
Expore loading the plugins via a separate [assembly load context ](https://learn.microsoft.com/en-us/dotnet/core/dependency-loading/understanding-assemblyloadcontext). This could allow updating and removing plugins at runtime in the exported application.

## Notes for devvvvs
### TODOs
- `PatchPath` node needs to be copied to each plugin to work correctly
- Build plugin which makes use of native dll
- Review internal code of Stride asset bundle loading

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
