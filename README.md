# Location Manager

Makes your custom location spawn during world generation in Valheim.

## How to add locations

Copy the asset bundle into your project and make sure to set it as an EmbeddedResource in the properties of the asset bundle.
Default path for the asset bundle is an `assets` directory, but you can override this.
This way, you don't have to distribute your assets with your mod. They will be embedded into your mods DLL.

### Merging the precompiled DLL into your mod

Add the following three lines to the bottom of the first PropertyGroup in your .csproj file, to enable C# V10.0 features and to allow the use of publicized DLLs.

```xml
<LangVersion>10</LangVersion>
<Nullable>enable</Nullable>
<AllowUnsafeBlocks>true</AllowUnsafeBlocks>
```

Download the LocationManager.dll from the release section to the right.
Including the dll is best done via ILRepack (https://github.com/ravibpatel/ILRepack.Lib.MSBuild.Task). You can load this package (ILRepack.Lib.MSBuild.Task) from NuGet.

If you have installed ILRepack via NuGet, simply create a file named `ILRepack.targets` in your project and copy the following content into the file

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
    <Target Name="ILRepacker" AfterTargets="Build">
        <ItemGroup>
            <InputAssemblies Include="$(TargetPath)" />
            <InputAssemblies Include="$(OutputPath)\LocationManager.dll" />
        </ItemGroup>
        <ILRepack Parallel="true" DebugInfo="true" Internalize="true" InputAssemblies="@(InputAssemblies)" OutputFile="$(TargetPath)" TargetKind="SameAsPrimaryAssembly" LibraryPath="$(OutputPath)" />
    </Target>
</Project>
```

Make sure to set the LocationManager.dll in your project to "Copy to output directory" in the properties of the DLL and to add a reference to it.
After that, simply add `using LocationManager;` to your mod and use the `Location` class, to add your locations.

## Example project

```csharp
using BepInEx;
using LocationManager;

namespace CustomLocation;

[BepInPlugin(ModGUID, ModName, ModVersion)]
public class CustomLocation : BaseUnityPlugin
{
	private const string ModName = "CustomLocation";
	private const string ModVersion = "1.0.0";
	private const string ModGUID = "org.bepinex.plugins.customlocation";

	public void Awake()
	{
		_ = new LocationManager.Location("guildfabs", "GuildAltarSceneFab")
		{
			MapIcon = "portalicon.png",
			ShowMapIcon = ShowIcon.Explored,
			Biome = Heightmap.Biome.Meadows,
			SpawnDistance = new Range(500, 1500),
			SpawnAltitude = new Range(10, 100),
			MinimumDistanceFromGroup = 100,
			Count = 15,
			Unique = true
		};
	}
}
```

```csharp
//  Location Configurable Properties
            // MapIcon                      Sets the map icon for the location.
            // ShowMapIcon                  When to show the map icon of the location. Requires an icon to be set. Use "Never" to not show a map icon for the location. Use "Always" to always show a map icon for the location. Use "Explored" to start showing a map icon for the location as soon as a player has explored the area.
            // MapIconSprite                Sets the map icon for the location.
            // CanSpawn                     Can the location spawn at all.
            // SpawnArea                    If the location should spawn more towards the edge of the biome or towards the center. Use "Edge" to make it spawn towards the edge. Use "Median" to make it spawn towards the center. Use "Everything" if it doesn't matter.</para>
            // Prioritize                   If set to true, this location will be prioritized over other locations, if they would spawn in the same area.
            // PreferCenter                 If set to true, Valheim will try to spawn your location as close to the center of the map as possible.
            // Rotation                     How to rotate the location. Use "Fixed" to use the rotation of the prefab. Use "Random" to randomize the rotation. Use "Slope" to rotate the location along a possible slope.
            // HeightDelta                  The minimum and maximum height difference of the terrain below the location.
            // SnapToWater                  If the location should spawn near water.
            // ForestThreshold              If the location should spawn in a forest. Everything above 1.15 is considered a forest by Valheim. 2.19 is considered a thick forest by Valheim.
            // Biome
            // SpawnDistance                Minimum and maximum range from the center of the map for the location.
            // SpawnAltitude                Minimum and maximum altitude for the location.
            // MinimumDistanceFromGroup     Locations in the same group will keep at least this much distance between each other.
            // GroupName                    The name of the group of the location, used by the minimum distance from group setting.
            // Count                        Maximum number of locations to spawn in. Does not mean that this many locations will spawn. But Valheim will try its best to spawn this many, if there is space.
            // Unique                       If set to true, all other locations will be deleted, once the first one has been discovered by a player.
```