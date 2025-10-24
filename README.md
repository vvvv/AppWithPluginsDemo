# PluginHostDemo
Sketches ideas how to build applications with plugins in VL

The main files are
- info: `myApp.vl` references `host.vl` and is intended for export.
- info: `myApp-develop.vl` like above but also references any plugins directly to be able to work on them
- info: `host.vl` contains host related logic and must not be referenced by plugins. In this example it will show a combo box to select the plugin which should run.
- Plugin Interfaces: `Layer-plugins.vl`, `TexFX-plugins.vl`, `Audio-plugins.vl` - These contain the plugin interfaces.

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

## Guidelines
- Keep your interfaces as minimal as possible. Make sure they only reference packages that they actually need. This helps to keep the output folder small. If you have multiple plugin interfaces using different technologies think about placing them in seperate VL files.
- Don't add a reference to an exported plugin.dll to your `myApp-develop` environment. Because this plugin.dll would require a specific interfaces.dll which during dev time is not available.
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
