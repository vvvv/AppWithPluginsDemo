# PluginHostDemo
Sketches ideas how to build applications with plugins in VL

The main files are
- `Layer-plugins.vl`, `TexFX-plugins.vl`, `Audio-plugins.vl` - These contain the plugin interfaces.
- `myApp.vl` references `host.vl` and is intended for export.
- `myApp-develop.vl` like above but also references any plugins directly to be able to work on them
- `host.vl` contains host related logic and must not be referenced by plugins. In this example it will show a combo box to select the plugin which should run.

`plugins` folder:
- `pluginA`, `pluginB` contain Skia Layer plugins
- `Ana` simple TextureFX
- `Posterize` a TextureFX with custom shaders
- `TexFXWithAssets` a TextureFX with an asset folder
- `TAL-NoiseMaker` an Audio plugin that uses a VST instrument

## Export
On the hosting side only `myApp.vl` is intended to be exported. The resulting executable will look beside its `plugins` folder for plugin dlls.

Plugins need to be exported into `MY_REPOS\VL.PluginHostDemo\exported\myApp\plugins`.

To export myApp from command line use
```
cd PROGRAM_FILES\vvvv_gamma_MY_VERSION_
vvvvc MY_REPOS\VL.PluginHostDemo\myApp.vl
```
`exported\myApp` folder now contains `myApp.exe`  

To export layer plugins from command line use
```
cd PROGRAM_FILES\vvvv_gamma_MY_VERSION_
vvvvc MY_REPOS\VL.PluginHostDemo\plugins\pluginA\pluginA.vl
vvvvc MY_REPOS\VL.PluginHostDemo\plugins\pluginB\pluginB.vl
```

To export TextureFX plugins from command line use
```
cd PROGRAM_FILES\vvvv_gamma_MY_VERSION_
vvvvc MY_REPOS\VL.PluginHostDemo\plugins\Ana\Ana.vl
vvvvc MY_REPOS\VL.PluginHostDemo\plugins\Posterize\Posterize.vl
vvvvc MY_REPOS\VL.PluginHostDemo\plugins\TexFXWithAssets\TexFXWithAssets.vl
```

Copy the Asset folder `MY_REPOS\VL.PluginHostDemo\plugins\TexFXWithAssets\` to `MY_REPOS\VL.PluginHostDemo\exported\myApp\plugins\TexFXWithAssets`

To export Audio plugins from command line use
```
cd PROGRAM_FILES\vvvv_gamma_MY_VERSION_
vvvvc MY_REPOS\VL.PluginHostDemo\plugins\TAL-NoiseMaker\TAL-NoiseMaker.vl
```

all plugins are now in `MY_REPOS\VL.PluginHostDemo\exported\myApp\plugins`


## Known Issues
- You can not add a reference to an exported plugin.dll to your `myApp-develop` environment. Because this plugin.dll would require a specific interfaces.dll which during dev time is not available.
- For now `PatchPath` needs to be copied from plugin to plugin, see TODO below
- Third party libraries need to be referenced by both host and plugin as of now, see TODO below. For example currently the host needs to reference `VL.Audio.VST` for the `TAL-NoiseMaker` plugin to work. Ideally this reference shouldn't exist.
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

## Credits
This is a slightly modified version as was outlined [in the forum](http://forum.vvvv.org/t/export-and-load-dlls-from-gamma/22278/4).

## Notes for devvvvs
### TODOs
- Add assembly resolver for plugins so they can have reference third partly dlls the host does not have
- Make sure node factories get notified about new plugins at runtime
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
