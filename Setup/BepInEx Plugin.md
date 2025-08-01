# Setting up a Wobbly Life BepInEx plugin

## Prerequisites

Before you start you need 4 things:

- **The .NET SDK**

  Download and install the latest recommended one from [the .NET downloads page](https://dotnet.microsoft.com/download)

- **An IDE**

  It's best to use an IDE such as [JetBrains Rider](https://www.jetbrains.com/rider/), [Visual Studio Community](https://visualstudio.microsoft.com/), or code editor like [Visual Studio Code](https://code.visualstudio.com/)

- **BepInEx installed**

  Make sure you installed BepInEx on Wobbly Life. It's recommended to enable the console window for debugging by editing your config at `<Wobbly Life>/BepInEx/config/BepInEx.cfg` like so:

  ```cfg
  [Logging.Console]
  
  ## Enables showing a console for log output.
  # Setting type: Boolean
  # Default value: false
  Enabled = true
  ```

- **BepInEx plugin templates**

  Install the templates by running this in the terminal:

  ```
  dotnet new install BepInEx.Templates::2.0.0-be.4 --nuget-source https://nuget.bepinex.dev/v3/index.json
  ```

## Creating the project

Create a new project using the templates installed before.

If your IDE can create a project using the template make sure you use `2020.3.44` as the Unity version and `net46` as TFM. If not you can also create a project in the current folder by running this command in the terminal:

```
dotnet new bepinex5plugin -n <Plugin Name> -T net46 -U 2020.3.44
```

Just make sure to replace <Plugin Name> with whatever name you want your plugin to have.

If your IDE didn't let you select the TFM you can also change it in the `.csproj` like so:

```csproj
<TargetFramework>net46</TargetFramework>
```

## Installing Wobbly Life libraries

Right now the plugin can't directly use Wobbly Life's code, to fix this we need to import the dlls (libraries) into our project. This can be done manually as outlined [here](https://docs.bepinex.dev/articles/dev_guide/plugin_tutorial/2_plugin_start.html#referencing-from-local-install) but it's simpler to just import them from NuGet. Either use your IDE to import the package [WobblyLife.GameLibs](https://www.nuget.org/packages/WobblyLife.GameLibs) or run this command in your project folder:

```
dotnet add package WobblyLife.GameLibs -v *-*
```
