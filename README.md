# PluginHostDemo
Sketches ideas how to build host with plugins in VL

This is a slightly modified version as was outlined [in the forum](http://forum.vvvv.org/t/export-and-load-dlls-from-gamma/22278/4).

The main files are
- `interfaces.vl` - this is the one file all others reference. It contains the plugin interface (called `IPlugin` in this example, which has an `Update` operation returning a Skia layer).
- `pluginA.vl` contains a class `PluginA` implementing `IPlugin` by rendering "Hello from A".
- `pluginB.vl` contains a class `PluginB` implementing `IPlugin` by rendering "Hello from B".
- `host.vl` contains host related logic and must not be referenced by plugins. In this example it will show a combo box to select the plugin which should run.
- `entrypoint-production.vl` references `host.vl` and is intended for export.
- `entrypoint-develop.vl` like above but also references any plugins directly to be able to work on them - this is needed as long as we lack dynamic loading of VL files during runtime.

## Export
On the hosting side only `entrypoint-production.vl` is intended to be exported. The resulting executable will look beside its `plugins` folder for plugin dlls.

On the plugin side the `plugin*.vl` get exported and their resulting dll needs to be manually copied to the before mentioned `plugins` folder.

## Future work
Expore loading the plugins via a separate [assembly load context ](https://learn.microsoft.com/en-us/dotnet/core/dependency-loading/understanding-assemblyloadcontext).
If that works we could add a file watcher on the plugins directory.