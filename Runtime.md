# Runtime / In-Game Editing (Digger PRO only)

Digger PRO allows you to modify the terrain at runtime (in-game).

## Setup

1. **Tools > Digger > Setup for runtime**.
   - This adds the `DiggerMasterRuntime` component to your scene.
2. Ensure your terrain has at least one layer with a texture and normal map.

## Scripting API

The `DiggerMasterRuntime` component provides methods to modify the terrain.

### Getting Started

```csharp
using Digger.Modules.Core.Sources;
using Digger.Modules.Runtime.Sources;

public class MyDiggerController : MonoBehaviour
{
    private DiggerMasterRuntime digger;

    void Start()
    {
        digger = FindObjectOfType<DiggerMasterRuntime>();
    }
}
```

### BrushType

Available brush shapes:

| Value | Description |
|-------|-------------|
| `BrushType.Sphere` | Spherical brush |
| `BrushType.HalfSphere` | Half-sphere (dome) |
| `BrushType.RoundedCube` | Cube with rounded edges |
| `BrushType.Stalagmite` | Pointed stalactite/stalagmite shape |
| `BrushType.Custom` | Custom brush from a 3D texture |

### ActionType

Available actions:

| Value | Description |
|-------|-------------|
| `ActionType.Dig` | Remove terrain (carve holes/caves) |
| `ActionType.Add` | Add terrain (fill holes, build) |
| `ActionType.Paint` | Paint texture without modifying geometry |
| `ActionType.PaintHoles` | Paint holes in terrain |
| `ActionType.Reset` | Reset terrain to original state |
| `ActionType.Smooth` | Smooth terrain surface (Sphere brush only) |
| `ActionType.BETA_Sharpen` | Sharpen terrain surface (beta) |

### Synchronous Modification

The `Modify` method performs the edit immediately. The game will freeze until the operation is complete.

```csharp
// Simple call
digger.Modify(position, BrushType.Sphere, ActionType.Dig, textureIndex: 0, opacity: 0.5f, size: 4f);
```

### Asynchronous Modification

To prevent frame drops, use asynchronous methods.

**`ModifyAsync`**:
Starts an async modification.
> **Note**: You cannot call this if another async operation is already running. Check `digger.IsRunningAsync`.

```csharp
// Check if we can start a new async operation
if (!digger.IsRunningAsync)
{
    await digger.ModifyAsync(hit.point, BrushType.Sphere, ActionType.Dig, 0, 0.5f, 4f);
}
```

**`ModifyAsyncBuffured`** (Recommended):
Adds the modification to a queue. Digger processes the queue one by one.

```csharp
if (Physics.Raycast(transform.position, transform.forward, out var hit))
{
    digger.ModifyAsyncBuffured(hit.point, BrushType.Sphere, ActionType.Dig, 0, 0.5f, 4f);
}
```

### ModificationParameters

For full control, use `ModificationParameters`:

```csharp
var parameters = new ModificationParameters
{
    Position = hit.point,
    Brush = BrushType.Sphere,
    Action = ActionType.Dig,
    TextureIndex = 0,
    Opacity = 0.5f,
    Size = 4f,
    StalagmiteUpsideDown = false,
    OpacityIsTarget = false,
    PaintWhileDigging = true,
    BypassDestructability = false,
    IsIndestructible = false,
    Callback = OnModificationComplete
};

digger.ModifyAsyncBuffured(parameters);
```

**Parameter Reference**:

| Parameter | Type | Description |
|-----------|------|-------------|
| `Position` | `float3` | World position of the modification |
| `Brush` | `BrushType` | Shape of the brush |
| `Action` | `ActionType` | Type of modification |
| `TextureIndex` | `int` | Texture index (0-7) |
| `Opacity` | `float` | Strength/intensity (0-1) |
| `Size` | `float3` | Size of the brush |
| `StalagmiteUpsideDown` | `bool` | Flip stalagmite brush |
| `OpacityIsTarget` | `bool` | Set texture weight directly to opacity value |
| `PaintWhileDigging` | `bool` | Apply texture while digging |
| `BypassDestructability` | `bool` | Ignore indestructible zones |
| `IsIndestructible` | `bool` | Make added terrain indestructible |
| `Callback` | `Action<ModificationResult>` | Called when operation completes |

### ModificationResult

All modification methods can return a `ModificationResult` struct containing statistics about the operation. This is useful for gameplay mechanics like resource gathering or building costs.

**Properties**:
- `RemovedMatterQuantity` (double): Sum of voxel values removed during digging.
- `AddedMatterQuantity` (double): Sum of voxel values added during building.
- `TotalModifiedVoxels` (int): Total number of voxels that were modified.
- `AverageChangePerVoxel` (double): Average magnitude of change per modified voxel.

```csharp
void OnModificationComplete(ModificationResult result)
{
    // Example: Award resources based on terrain removed
    int oreGained = (int)(result.RemovedMatterQuantity * 10);
    playerInventory.AddOre(oreGained);

    Debug.Log($"Removed: {result.RemovedMatterQuantity:F2}");
    Debug.Log($"Added: {result.AddedMatterQuantity:F2}");
    Debug.Log($"Voxels modified: {result.TotalModifiedVoxels}");
}
```

### Common Examples

**Digging a tunnel:**
```csharp
digger.ModifyAsyncBuffured(position, BrushType.Sphere, ActionType.Dig, 0, 0.5f, size: 3f);
```

**Building/filling terrain:**
```csharp
digger.ModifyAsyncBuffured(position, BrushType.Sphere, ActionType.Add, textureIndex: 2, 0.5f, size: 4f);
```

**Painting texture only:**
```csharp
digger.ModifyAsyncBuffured(position, BrushType.Sphere, ActionType.Paint, textureIndex: 1, 0.8f, size: 5f);
```

**Smoothing terrain:**
```csharp
// Note: Smooth only works with Sphere brush
digger.ModifyAsyncBuffured(position, BrushType.Sphere, ActionType.Smooth, 0, 0.3f, size: 6f);
```

**Creating a stalagmite:**
```csharp
digger.Modify(position, BrushType.Stalagmite, ActionType.Add, textureIndex: 0, opacity: 1f,
    size: 2f, stalagmiteHeight: 8f, stalagmiteUpsideDown: false);
```

**Creating a stalactite (upside-down):**
```csharp
digger.Modify(ceilingPosition, BrushType.Stalagmite, ActionType.Add, textureIndex: 0, opacity: 1f,
    size: 2f, stalagmiteHeight: 6f, stalagmiteUpsideDown: true);
```

**Resetting an area to original terrain:**
```csharp
digger.ModifyAsyncBuffured(position, BrushType.Sphere, ActionType.Reset, 0, 1f, size: 5f);
```

### Buffer Management

Control how many modifications can be queued:

```csharp
// Set buffer size (default is 1)
digger.BufferSize = 10;

// Check current queue size before adding
bool wasAdded = digger.ModifyAsyncBuffured(position, BrushType.Sphere, ActionType.Dig, 0, 0.5f, 4f);
if (!wasAdded)
{
    Debug.Log("Buffer full, modification discarded");
}
```

## Persistence

Changes made at runtime are not automatically saved to disk.

> **Note**: Persistence must be enabled in the `DiggerMasterRuntime` inspector.

**Save all changes:**
```csharp
digger.PersistAll();
```

**Delete all saved data:**
```csharp
digger.DeleteAllPersistedData();
```

**Clear all modifications (without deleting save):**
```csharp
digger.ClearScene();
```

**Multiple save slots:**
```csharp
// Set save slot before loading/saving
digger.SetPersistenceDataPathPrefix("SaveSlot1");

// Later, switch to another slot
digger.SetPersistenceDataPathPrefix("SaveSlot2");

// Or use player-specific saves
digger.SetPersistenceDataPathPrefix($"Player_{playerId}");
```

Data is saved to: `Application.persistentDataPath/DiggerData/{prefix}/`

## Procedural Terrains

If you generate terrains at runtime, you can add Digger to them dynamically.

> **Requirement**: You must have at least one "template" terrain with Digger setup in the editor so Digger knows which materials/textures to use.

**Basic setup:**
```csharp
// 'terrain' is your newly generated terrain
digger.SetupRuntimeTerrain(terrain);
```

**With persistence support:**
```csharp
// Use a GUID to persist data for procedurally generated terrains
string terrainGuid = $"terrain_{chunkX}_{chunkZ}";
digger.SetupRuntimeTerrain(terrain, terrainGuid);
```

**After removing a terrain:**
```csharp
Destroy(terrain.gameObject);
digger.RefreshTerrainList(); // Update Digger's terrain list
```

## Performance Tips

- Keep **Brush Size** small.
- Use **Async** methods (`ModifyAsyncBuffured`).
- Disable **LOD generation** in Settings (it is expensive to recompute LODs at runtime).
- Set **Chunk Size** to 16 in Settings.
- Increase `BufferSize` if you need to queue many rapid modifications.

