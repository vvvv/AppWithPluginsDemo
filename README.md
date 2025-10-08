# PluginHostDemo
Sketches ideas how to build host with plugins in VL

This is a slightly modified version as was outlined [in the forum](http://forum.vvvv.org/t/export-and-load-dlls-from-gamma/22278/4).

The main files are
- `interfaces.vl` - this is the one file all others reference. It contains the plugin interface (called `IPlugin` in this example, which has an `Update` operation returning a Skia layer).
- `pluginA.vl` contains a class `PluginA` implementing `IPlugin` by rendering "Hello from A".
- `pluginB.vl` contains a class `PluginB` implementing `IPlugin` by rendering "Hello from B".
- `host.vl` contains host related logic and must not be referenced by plugins. In this example it will show a combo box to select the plugin which should run.
- `entrypoint-production.vl` references `host.vl` and is intended for export.
- `entrypoint-develop.vl` like above but also references any plugins directly to be able to work on them

## Export
On the hosting side only `entrypoint-production.vl` is intended to be exported. The resulting executable will look beside its `plugins` folder for plugin dlls.

On the plugin side the `plugin*.vl` get exported and their resulting dll needs to be manually copied to the before mentioned `plugins` folder.

## Known Issues
- You can not add a reference to an exported plugin.dll to your `entrypoint-develop` environment. Because this plugin.dll would require a specific interfaces.dll which during dev time is not available.
- Stride asets?
- VST?
- Native dlls?
- A plugin most likely consist of more than just the plugin.dll. ie. consider having plugin folders instead of just the .dlls
- Path references (relative to doc, vs. .exe) -> consider AssemblyPath a la ApplicationPath

## Do's and Don'ts
- how to handle issues with dependencies from different vvvv versions (one that exports the host, one that plugin-developers are using)
- also: different versions of nugets that plugins are using 

## Future work
Expore loading the plugins via a separate [assembly load context ](https://learn.microsoft.com/en-us/dotnet/core/dependency-loading/understanding-assemblyloadcontext). This could allow updating and removing plugins at runtime in the exported application.
