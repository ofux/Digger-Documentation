# Advanced Operations

Digger includes advanced tools to speed up complex tasks.

## Spline Operation

The Spline operation allows you to Dig or Add terrain along a Bezier curve. Perfect for roads, tunnels, or rivers.

### 1. Create a Spline

1. Go to **Tools > Digger > Create Bezier Spline**.
2. Select the new `BezierSpline` object in the hierarchy.

### 2. Edit the Spline

- **Add Point**: Click "Add Point" in the inspector.
- **Move Point**: Select a point in the Scene view and move it.
- **Curve**: Select the control handles (little spheres) to adjust the curve.
- **Remove Point**: Select a point and click "Remove" in the inspector.

### 3. Apply to Terrain

1. Open **Digger Master**.
2. Select **Spline** in the **Operation** dropdown.
3. Assign your `BezierSpline` object to the **Spline** field.
4. Select the sub-operation (Dig or Add).
5. Click **Perform operation along the spline**.

### Spline Settings

| Setting | Description |
|---------|-------------|
| **Width** | The width/radius of the tunnel or path |
| **Steps** | Number of points along the spline to process (more = smoother) |
| **Texture Index** | Texture to apply when adding terrain |

## Procedural Cave Generation

Digger includes a `CaveGenerator` class for creating procedural cave systems using splines.

### Using CaveGenerator

```csharp
using Digger.Modules.AdvancedOperations.Splines;
using Digger.Modules.AdvancedOperations.Splines.ProceduralGeneration;
using UnityEngine;

public class ProceduralCaveExample : MonoBehaviour
{
    public BezierSpline spline;

    void Start()
    {
        var generator = new CaveGenerator(
            step: 4f,                        // Distance between points
            stepCount: 100,                  // Number of points
            minY: -20f,                      // Minimum altitude variation
            maxY: 20f,                       // Maximum altitude variation
            altitudeVariationFrequency: 0.03f,
            horizontalVariationFrequency: 0.05f,
            seed1: 1337,                     // Seeds for reproducible generation
            seed2: 13,
            seed3: 17
        );

        // Generate points along the spline
        generator.GeneratePoints(transform.position, spline);

        // Now use the spline with Digger's spline operation
    }
}
```

### CaveGenerator Parameters

| Parameter | Description |
|-----------|-------------|
| `step` | Distance between generated points |
| `stepCount` | Total number of points to generate |
| `minY` / `maxY` | Vertical range for altitude variation |
| `altitudeVariationFrequency` | How quickly altitude changes (lower = smoother) |
| `horizontalVariationFrequency` | How much the path wanders horizontally |
| `seed1`, `seed2`, `seed3` | Random seeds for reproducible results |

## Spline Decorator

The `SplineDecorator` component spawns objects along a spline at regular intervals. Useful for placing torches, rocks, or other decorations along cave paths.

### Setup

1. Create a Bezier Spline (see above).
2. Create an empty GameObject and add the **Spline Decorator** component.
3. Configure the decorator:

| Setting | Description |
|---------|-------------|
| **Spline** | Reference to the BezierSpline |
| **Frequency** | How many times to repeat the item set |
| **Look Forward** | Orient objects to face along the spline direction |
| **Items** | Array of prefabs to spawn |

### Example: Torch-lit Tunnel

1. Create a torch prefab with a point light.
2. Add SplineDecorator to an empty GameObject.
3. Set **Frequency** to 10.
4. Enable **Look Forward**.
5. Add the torch prefab to **Items**.
6. Objects spawn at runtime when the scene starts.

```csharp
// SplineDecorator spawns items automatically in Awake()
// Items are instantiated as children of the decorator GameObject
```

## Easy Overhangs

Easy Overhangs is a specialized tool for creating overhangs on cliffs without manual sculpting.

1. Select **Easy Overhangs** in the **Operation** dropdown.
2. Adjust **Brush Size** and **Opacity**.
3. Select the **Texture** you want to apply.
4. Click on a steep slope or cliff face.
5. Digger will automatically extrude the terrain to create an overhang.

### Tips for Easy Overhangs

- Works best on steep terrain (cliffs, hillsides).
- Multiple clicks build up the overhang.
- Use lower opacity for subtle overhangs, higher for dramatic ones.
- Combine with Paint tool to add different textures to the underside.

## Combining Operations

For complex cave systems, combine multiple techniques:

1. **Plan the layout**: Use CaveGenerator to create the main tunnel spline.
2. **Carve the main tunnel**: Apply the spline operation with Dig.
3. **Add chambers**: Use manual Dig with large Sphere brush.
4. **Add details**: Use Stalagmite brush for formations.
5. **Decorate**: Use SplineDecorator for lighting and props.
6. **Paint**: Use different textures for variety.

