# Integrations

## Unity NavMesh (PRO only)

Digger PRO supports runtime NavMesh updates using Unity's `NavMeshComponents`.

### Setup
1. **Tools > Digger > Setup NavMeshComponents**.
   - Adds `DiggerNavMeshRuntime` to the scene.
2. Ensure you have a `NavMeshSurface` component in your scene.
   - **Do not** bake NavMesh using the old Navigation window. Use `NavMeshSurface`.

### Runtime Update
Call these methods from your scripts:

1. **Start**: Call `CollectNavMeshSources()` once at startup.
2. **Update**: Call `UpdateNavMeshAsync()` when you want to refresh the NavMesh (e.g., after a digging operation).

```csharp
var navMeshRuntime = FindObjectOfType<DiggerNavMeshRuntime>();
navMeshRuntime.CollectNavMeshSources();

// ... later, after digging ...
navMeshRuntime.UpdateNavMeshAsync();
```

### Automatic Update Example

```csharp
using Digger.Modules.Core.Sources;
using Digger.Modules.Runtime.Sources;
using UnityEngine;

public class DigWithNavMeshUpdate : MonoBehaviour
{
    private DiggerMasterRuntime digger;
    private DiggerNavMeshRuntime navMesh;

    void Start()
    {
        digger = FindObjectOfType<DiggerMasterRuntime>();
        navMesh = FindObjectOfType<DiggerNavMeshRuntime>();
        navMesh.CollectNavMeshSources();
    }

    void Dig(Vector3 position)
    {
        var parameters = new ModificationParameters
        {
            Position = position,
            Brush = BrushType.Sphere,
            Action = ActionType.Dig,
            Size = 3f,
            Opacity = 0.5f,
            Callback = OnDigComplete
        };
        digger.ModifyAsyncBuffured(parameters);
    }

    void OnDigComplete(ModificationResult result)
    {
        // Update NavMesh after terrain modification
        navMesh.UpdateNavMeshAsync();
    }
}
```

## MicroSplat

MicroSplat is fully supported and recommended for best visual results.

**Requirements:**
1. [MicroSplat](https://assetstore.unity.com/packages/tools/terrain/microsplat-96478)
2. [Triplanar module](https://assetstore.unity.com/packages/tools/terrain/microsplat-triplanar-uvs-96777)
3. [Digger integration module](https://assetstore.unity.com/packages/tools/terrain/microsplat-digger-integration-162840)

**Usage:**
- Digger automatically detects MicroSplat.
- If materials get out of sync, enable **Force MicroSplat Material Update** in Digger Settings.

## PolyTerrains Integration (PRO only)

Digger includes support for blocky, Minecraft-style terrain modifications through the `SetBlockOperation`.

### SetBlockOperation

The `SetBlockOperation` allows you to set voxels in a cubic region to specific values, useful for voxel-based building systems.

```csharp
using Digger.Modules.Core.Sources;
using Digger.Modules.PolyTerrainsIntegration.Sources;
using Digger.Modules.Runtime.Sources;
using Unity.Mathematics;
using UnityEngine;

public class BlockyTerrainExample : MonoBehaviour
{
    private DiggerMasterRuntime digger;

    void Start()
    {
        digger = FindObjectOfType<DiggerMasterRuntime>();
    }

    // Place a block at the given voxel position
    public void PlaceBlock(int3 voxelPosition, int textureIndex)
    {
        var operation = new SetBlockOperation
        {
            Position = voxelPosition,
            Size = new int3(1, 1, 1),      // Single block
            TextureIndex = (uint)textureIndex,
            TargetValue = -1f              // Negative = solid
        };

        digger.Modify(operation);
    }

    // Remove a block at the given voxel position
    public void RemoveBlock(int3 voxelPosition)
    {
        var operation = new SetBlockOperation
        {
            Position = voxelPosition,
            Size = new int3(1, 1, 1),
            TextureIndex = 0,
            TargetValue = 1f               // Positive = empty
        };

        digger.Modify(operation);
    }

    // Place a larger structure
    public void PlaceWall(int3 startPosition, int width, int height)
    {
        var operation = new SetBlockOperation
        {
            Position = startPosition,
            Size = new int3(width, height, 1),
            TextureIndex = 2,              // Stone texture
            TargetValue = -1f
        };

        digger.Modify(operation);
    }
}
```

### SetBlockOperation Properties

| Property | Type | Description |
|----------|------|-------------|
| `Position` | `int3` | Voxel position (not world position) |
| `Size` | `int3` | Size of the block region in voxels |
| `TextureIndex` | `uint` | Texture to apply (0-7) |
| `TargetValue` | `float` | Voxel value: negative = solid, positive = empty |

### Converting World Position to Voxel Position

```csharp
using Digger.Modules.Core.Sources;

// Convert world position to voxel position
int3 voxelPos = Utils.UnityToVoxelPosition(worldPosition, diggerSystem.HeightmapScale);

// Convert voxel position back to world position
Vector3 worldPos = Utils.VoxelToUnityPosition(voxelPos, diggerSystem.HeightmapScale);
```

## MapMagic 2 (PRO only)

To use Digger with MapMagic 2's infinite terrain generation:

1. Create a script to listen for tile generation events.
2. Call `SetupRuntimeTerrain` on new tiles.

```csharp
using Digger.Modules.Runtime.Sources;
using MapMagic.Terrains;
using UnityEngine;

public class MapMagicDiggerIntegration : MonoBehaviour
{
    private DiggerMasterRuntime digger;

    private void OnEnable()
    {
        digger = FindObjectOfType<DiggerMasterRuntime>();
        MapMagic.Terrains.TerrainTile.OnTileApplied += OnTileCompleted;
    }

    private void OnDisable()
    {
        MapMagic.Terrains.TerrainTile.OnTileApplied -= OnTileCompleted;
    }

    private void OnTileCompleted(TerrainTile tile, TileData tileData, StopToken stopToken)
    {
        if (tileData.isDraft) return;

        Terrain terrain = tile.GetTerrain(false) ?? tile.GetTerrain(true);
        if (terrain != null)
        {
            // Use tile coordinates for persistence
            string guid = $"tile_{tile.coord.x}_{tile.coord.z}";
            digger.SetupRuntimeTerrain(terrain, guid);
        }
    }
}
```

### Persisting MapMagic Terrain Data

To save and load Digger modifications on MapMagic terrains:

```csharp
// Set a path prefix based on save slot
digger.SetPersistenceDataPathPrefix($"SaveSlot1");

// Data will be saved to:
// Application.persistentDataPath/DiggerData/SaveSlot1/tile_X_Z/
```

## Gaia Integration

Digger works with Gaia-generated terrains. After generating terrain with Gaia:

1. Run **Tools > Digger > Setup terrains**.
2. Ensure terrain layers have both texture and normal maps.
3. Click **Sync with terrain(s) & Refresh** if you modify terrain after setup.

## Other Terrain Tools

Digger is compatible with most terrain generation tools. The general workflow is:

1. Generate your terrain with the external tool.
2. Ensure terrain layers have textures AND normal maps.
3. Run **Tools > Digger > Setup terrains**.
4. If the tool modifies terrain after Digger setup, use **Sync & Refresh**.

