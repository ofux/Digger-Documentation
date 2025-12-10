# Frequently Asked Questions & Troubleshooting

## Common Issues

### When I dig, a hole appears but the cave mesh is invisible
- **Cause**: Missing texture/normal map on terrain layers.
- **Fix**: Ensure ALL your terrain layers have both a Texture and a Normal Map assigned.

### I changed terrain textures, but caves show old textures
- **Fix**: Go to Digger Master > Settings and click **Sync with terrain(s) & Refresh**.

### Digger meshes don't blend perfectly with the terrain
- **Lightmapping**: If using baked lighting, ensure you click "Prepare for lightmapping" in the Lighting tab before baking.
- **Draw Instanced**: In some cases, the "Draw Instanced" setting on the Terrain can cause lighting mismatches. Try disabling it in Terrain Settings.
- **Pixel Error**: Try lowering the "Pixel Error" value in Terrain Settings.

### Exporting Digger Data
- **Issue**: Exporting a Unity Package doesn't include Digger data.
- **Reason**: Digger stores data in hidden `.internal` folders which Unity ignores by default.
- **Solution**: Manually copy the `Assets/DiggerData` folder when moving projects.

### Performance is slow when editing
- Reduce **Brush Size**.
- Set **Chunk Size** to 16 in Settings.
- Reduce **Resolution** multiplier.
- The first edit after opening Unity may be slow due to Burst compilation.

## Gameplay Questions

### Does Digger support NavMesh?
Yes. Digger supports the built-in NavMesh. For runtime updates (Digger PRO), see the [Integrations](Integrations.md) page.

### Can players dig at runtime?
Yes, but only with **Digger PRO**. The standard version is for editor-time creation only.

### Does this affect collisions?
Yes, Digger automatically updates mesh colliders.

### Can I make certain areas indestructible?
Yes, use the **Advanced Voxel Generator** with depth layers or noise layers to create indestructible regions. See [Settings](Settings.md) for details.

At runtime, you can also use `IsIndestructible = true` in `ModificationParameters` when adding terrain.

## Multiplayer / Networking

Digger does not include built-in networking, but you can synchronize terrain modifications across clients.

### Basic Approach

1. **Send modification parameters** over the network instead of voxel data.
2. **Apply modifications** on all clients using the same parameters.
3. **Use persistence** to save/load terrain state.

### Example Architecture

```csharp
using Digger.Modules.Core.Sources;
using Digger.Modules.Runtime.Sources;
using UnityEngine;
// Using your networking solution (Mirror, Netcode, Photon, etc.)

public class NetworkedDigger : MonoBehaviour
{
    private DiggerMasterRuntime digger;

    void Start()
    {
        digger = FindObjectOfType<DiggerMasterRuntime>();
    }

    // Called by the local player when they dig
    public void LocalDig(Vector3 position, float size)
    {
        // Apply locally
        ApplyDig(position, size);

        // Send to server/other clients
        SendDigCommand(position, size);
    }

    // Called when receiving dig command from network
    public void OnNetworkDigReceived(Vector3 position, float size)
    {
        ApplyDig(position, size);
    }

    private void ApplyDig(Vector3 position, float size)
    {
        var parameters = new ModificationParameters
        {
            Position = position,
            Brush = BrushType.Sphere,
            Action = ActionType.Dig,
            Size = size,
            Opacity = 0.5f,
            TextureIndex = 0
        };
        digger.ModifyAsyncBuffured(parameters);
    }

    // Implement with your networking solution
    private void SendDigCommand(Vector3 position, float size)
    {
        // Your network code here
    }
}
```

### Synchronization Tips

- **Deterministic**: Same parameters produce same results, so sync parameters, not geometry.
- **Authority**: Have the server validate dig requests to prevent cheating.
- **Late joiners**: Save terrain state and send to new players, or replay modification history.
- **Persistence**: Use `PersistAll()` on the server and share save files with clients.

## Lighting

### Light Probes

Digger meshes work with Light Probes for dynamic lighting in caves.

**Setup:**
1. Place Light Probe Groups in your caves.
2. Ensure Digger meshes are not marked as static (or use Light Probe Proxy Volumes).
3. Bake lighting.

**Tips:**
- Dense probe placement in cave interiors provides better lighting.
- Use Light Probe Proxy Volumes for large caves.
- Probe lighting updates automatically as meshes change at runtime.

### Baked Lighting (Lightmapping)

1. Enable **Contribute GI** in Digger Settings.
2. In Digger Master, go to the **Lighting** tab.
3. Click **Prepare for lightmapping**.
4. Open the Lighting window and bake.

> **Note**: Lightmapping is for editor-created caves only. Runtime modifications won't be lightmapped.
> **Note**: Lightmapping doesn't work with the MicroSplat integration.

### Realtime Lighting

For runtime digging, use realtime lights:
- Point lights in caves
- Spot lights for focused illumination
- Light Probes for ambient lighting on characters/objects

## Build & Platform

### Stripping Settings

If you encounter errors in builds related to missing types:

1. Go to **Edit > Project Settings > Player**.
2. Under **Other Settings**, find **Managed Stripping Level**.
3. Set to **Minimal** or **Low** if you experience issues.

Digger uses reflection for some features which aggressive stripping can break.

### Platform Support

| Platform | Support |
|----------|---------|
| Windows | Full |
| macOS | Full |
| Linux | Full |
| iOS | Full (PRO) |
| Android | Full (PRO) |
| WebGL | Limited (no persistence) |
| Consoles | Contact support |

### Mobile Optimization

For mobile platforms:
- Set **Chunk Size** to 16.
- Use **Resolution** x1 or x2.
- Disable **LOD generation** if not needed.
- Keep brush sizes small.
- Limit concurrent async operations.

### Build Size

Digger data is stored in `Assets/DiggerData`. This folder is included in builds.

To reduce build size:
- Delete unused scene data from `Assets/DiggerData`.
- Use **Auto save meshes as Assets** to avoid bloating scene files.

## Debugging

### How can I see the meshes generated by Digger?
1. Go to Digger Master > Settings.
2. Enable **Show underlying objects**.
3. You will see `Chunk` objects appear as children of your Terrain in the Hierarchy.

### Digger creates too many chunks
- Increase **Chunk Size** in Settings.
- This reduces the number of chunks but makes each one larger.

### Memory usage is high
- Reduce **Resolution** multiplier.
- Reduce **Chunk Size** to 16 (uses less memory per chunk).
- Call `ClearScene()` to remove all modifications if testing.

### Burst compilation errors
- Ensure Burst package is up to date.
- Try **Jobs > Burst > Clear Compile Cache** from the menu.
- Check for conflicting Burst versions from other assets.

