# Editing Guide

Digger offers a variety of tools to shape your terrain. All editing tools are located in the **Edit** tab of the **Digger Master** inspector.

## Tools and Actions

### Dig
Removes matter from the terrain to create caves, tunnels, or entrances.

### Add (Raise Overhangs)
Adds matter to the terrain. Useful for creating overhangs, covering holes, or building organic structures.

### Reset
Restores the terrain to its original heightmap state in the brush area.
> **Note**: This does not restore terrain details (grass/trees) that were removed by Digger.

### Paint
Paints textures on the Digger meshes (caves/overhangs). Digger uses the same texture layers as your terrain. Select the texture you want to paint from the list.

> **Note for MicroSplat users**: If you use MicroSplat, the Paint tool allows you to paint Wetness, Puddles, and Streams (if enabled in your MicroSplat config).

### Paint Holes
Used to manually cut holes in the terrain surface without generating cave geometry. Digger usually handles holes automatically when you Dig, but this gives you manual control.

### Smooth
Smooths the terrain surface, softening sharp edges and creating more natural-looking transitions.

> **Note**: Smooth only works with the **Sphere** brush.

### Sharpen (Beta)
Sharpens terrain edges, creating more defined features. This is a beta feature.

> **Note**: Sharpen only works with the **Sphere** brush.

## Brushes

### Sphere
Standard spherical brush. Creates smooth, organic shapes.

**Best for**:
- Tunnels and cave passages
- Organic, natural-looking holes
- Smoothing operations

### HalfSphere
A dome-shaped brush (half of a sphere).

**Best for**:
- Cave entrances
- Alcoves and niches
- Domed ceilings

### RoundedCube
A cube with rounded edges.

**Best for**:
- Rectangular rooms with soft corners
- Mine shafts
- Architectural spaces that need defined walls but soft edges

### Stalagmite
Special brush to create pointed stalagmites or stalactites.

**Best for**:
- Cave decorations
- Pointed rock formations
- Use negative opacity or hold `Shift` to create stalactites (pointing down)

### Custom
Use any mesh as a brush shape. See [Custom Brushes](#custom-brushes) below.

**Best for**:
- Stamping specific shapes
- Architectural details
- Repeatable patterns

## Brush Properties

- **Brush Size**: The radius or extent of the brush.
- **Opacity**: The strength/speed of the modification.
- **Depth**: Adjusts the position of the brush relative to the terrain surface. Negative values push the brush underground.
- **Auto-remove terrain details**: When enabled, grass and trees are automatically removed as you dig. This operation is irreversible.

## Custom Brushes

Custom brushes allow you to use any 3D mesh as a brush shape.

### Creating a Custom Brush

1. Create or import a mesh into your project.
2. Add the mesh to your scene (it doesn't need to be near the terrain).
3. Add the **Custom Brush** component to the mesh GameObject:
   - Select the mesh object
   - Click **Add Component > Digger > Custom Brush**
4. The mesh will be converted to voxels automatically.

### Using a Custom Brush

1. In **Digger Master**, set **Brush** to `Custom`.
2. Assign your Custom Brush object to the **Custom Brush** field.
3. Click on the terrain to stamp the shape.

### Custom Brush Settings

| Setting | Description |
|---------|-------------|
| `Auto Refresh` | When enabled, the brush updates automatically when you modify the mesh's transform (rotation/scale). |

### Tips for Custom Brushes

- **Scale**: Adjust the mesh's scale to change the brush size independently of the Brush Size setting.
- **Rotation**: Rotate the mesh to change the orientation of the stamp.
- **Closed meshes**: For best results, use closed (watertight) meshes.
- **Complexity**: Simpler meshes work better. Very complex meshes may produce unexpected results.

```csharp
// Example: Using custom brush at runtime
var parameters = new ModificationParameters
{
    Position = hit.point,
    Brush = BrushType.Custom,
    CustomBrush = myCustomBrushComponent, // Reference to CustomBrush component
    Action = ActionType.Dig,
    TextureIndex = 0,
    Opacity = 0.5f,
    Size = 2f
};
digger.ModifyAsyncBuffured(parameters);
```

## Shortcuts

If shortcuts are enabled in the **Settings** tab:

| Key | Action |
| --- | --- |
| **B** | Change Brush type |
| **N** | Change Action |
| **Keypad +** | Increase Brush Size |
| **Keypad -** | Decrease Brush Size |
| **Keypad *** | Increase Opacity |
| **Keypad /** | Decrease Opacity |
| **Shift + Click** | Invert action (e.g., Add instead of Dig) |

