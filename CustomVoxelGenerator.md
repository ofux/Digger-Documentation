# Custom Voxel Generator Documentation

## Overview

Digger's voxel generation system allows you to create custom generators that control how voxels are created from the terrain heightmap. This provides flexibility to add procedural variations, custom textures, destructibility settings, and other properties to your voxels.

The system uses Unity's Job System and Burst compiler for high-performance parallel processing of voxel generation.

## When to Create Custom Generators

Consider creating a custom voxel generator when you want to:

- Apply procedural noise or patterns to voxel properties
- Assign textures based on depth, position, or other criteria
- Control voxel destructibility for different gameplay effects (e.g., indestructible rock layers)
- Implement biome-based voxel generation
- Add custom logic for specific game mechanics

## Built-in Generators

Digger comes with two built-in generators:

### Simple Voxel Generator
The default generator that creates voxels based purely on the terrain heightmap. It calculates the signed distance field (SDF) value for each voxel without any additional processing.

### Advanced Voxel Generator
A sophisticated layered generator that supports:

**Depth Layers**:
- Assign textures and destructibility based on depth below terrain surface
- Each layer specifies: minimum depth, texture index, and whether voxels are destructible
- Automatically sorted by depth (deepest first)

**Noise Layers**:
- Add procedural variation using Perlin noise
- Each noise layer has: scale, octaves, persistence, destructibility, texture override, blend mode, and threshold
- Multiple layers can be stacked and combined
- Two blend modes:
  - **Replace**: Fully replaces the destructibility when noise influence is high
  - **Add**: Only makes voxels indestructible (if layer is indestructible), never makes them destructible

The generator first applies depth layers, then applies noise layers in order, allowing for complex procedural voxel generation.

## Architecture

The voxel generation system consists of three main components:

1. **IVoxelGenerator Interface**: Defines the contract for voxel generators
2. **IJobParallelFor Job Struct**: Performs the actual voxel generation using Unity's Job System
3. **AVoxelGeneratorEditor**: Provides custom inspector UI for the generator

## Step-by-Step Guide

### 1. Create a ScriptableObject Class Implementing IVoxelGenerator

Create a new C# file in your project (e.g., `MyCustomVoxelGenerator.cs`):

```csharp
using Digger.Modules.Core.Sources.Generators;
using Unity.Collections;
using Unity.Jobs;
using Unity.Mathematics;
using UnityEngine;

namespace MyProject.Digger.Generators
{
    [CreateAssetMenu(fileName = "MyCustomVoxelGenerator", 
                     menuName = "Digger/Voxel Generators/My Custom Generator", 
                     order = 10)]
    public class MyCustomVoxelGenerator : ScriptableObject, IVoxelGenerator
    {
        [Header("Custom Settings")]
        [Range(0f, 1f)]
        public float customParameter = 0.5f;

    public JobHandle GenerateVoxels(
        float[] heightArray,
        int3 chunkPosition,
        int sizeVox,
        float3 heightmapScale,
        NativeArray<float> heights,
        NativeArray<Voxel> voxels,
        bool refreshOnly)
    {
        var jobData = new MyCustomVoxelGenerationJob
        {
            ChunkPosition = chunkPosition,
            Heights = heights,
            Voxels = voxels,
            SizeVox = sizeVox,
            SizeVox2 = sizeVox * sizeVox,
            HeightmapScale = heightmapScale,
            CustomParameter = customParameter
        };

        return jobData.Schedule(voxels.Length, 64);
    }
    }
}
```

### 2. Create the IJobParallelFor Struct

Create the job struct that will be executed in parallel (e.g., in `MyCustomVoxelGenerationJob.cs`):

```csharp
using Unity.Burst;
using Unity.Collections;
using Unity.Jobs;
using Unity.Mathematics;
using Digger.Modules.Core.Sources;

namespace MyProject.Digger.Generators
{
    [BurstCompile(CompileSynchronously = true, FloatMode = FloatMode.Fast, OptimizeFor = OptimizeFor.Performance)]
    public struct MyCustomVoxelGenerationJob : IJobParallelFor
    {
        public int3 ChunkPosition;
        public int SizeVox;
        public int SizeVox2;
        public float3 HeightmapScale;
        public float CustomParameter;

        [ReadOnly] [NativeDisableParallelForRestriction]
        public NativeArray<float> Heights;

        [WriteOnly] 
        public NativeArray<Voxel> Voxels;

        public void Execute(int index)
        {
            // Convert linear index to 3D coordinates
            var pi = Utils.IndexToXYZ(index, SizeVox, SizeVox2);
            
            // Get height from terrain
            var height = Heights[Utils.XYZToHeightIndex(pi, SizeVox)];
            
            // Convert to world position
            var p = Utils.ChunkVoxelToUnityPosition(ChunkPosition, pi, HeightmapScale);
            
            // Create voxel with SDF value
            var voxel = new Voxel(p.y - height, HeightmapScale.y);
            
            // Apply custom logic here
            // For example, set destructibility based on your parameter
            // Use SetMaxValue to make voxels indestructible (maxValue >= 32 means indestructible)
            if (CustomParameter > 0.5f)
                voxel.SetMaxValue(HeightmapScale.y, HeightmapScale.y); // Makes voxel indestructible
            
            // You can also set textures
            voxel.FirstTextureIndex = 0;
            voxel.SecondTextureIndex = 0;
            voxel.NormalizedTextureLerp = 0f;
            
            Voxels[index] = voxel;
        }
    }
}
```

### 3. Create the Custom Editor

Create an editor class in an Editor folder (e.g., `MyCustomVoxelGeneratorEditor.cs`):

```csharp
using Digger.Modules.Core.Editor.Generators;
using Digger.Modules.Core.Sources.Generators;
using UnityEditor;
using UnityEngine;

namespace MyProject.Digger.Generators.Editor
{
    [VoxelGeneratorAttr("My Custom Generator", 10)]
    public class MyCustomVoxelGeneratorEditor : AVoxelGeneratorEditor
    {
        private MyCustomVoxelGenerator customGenerator;
        private SerializedObject serializedGenerator;

        public override void OnEnable()
        {
            customGenerator = generator as MyCustomVoxelGenerator;
            if (customGenerator != null)
            {
                serializedGenerator = new SerializedObject(customGenerator);
            }
        }

        public override void OnDisable()
        {
            serializedGenerator?.Dispose();
        }

        public override void OnInspectorGUI()
        {
            if (customGenerator == null || serializedGenerator == null)
            {
                EditorGUILayout.HelpBox("No generator selected.", MessageType.Warning);
                return;
            }
            
            serializedGenerator.Update();
            
            EditorGUILayout.HelpBox(
                "My Custom Voxel Generator\n\n" +
                "Description of what this generator does.",
                MessageType.Info);
            
            EditorGUILayout.Space();
            
            var customParamProperty = serializedGenerator.FindProperty("customParameter");
            EditorGUILayout.PropertyField(customParamProperty, 
                new GUIContent("Custom Parameter", "Tooltip for your parameter"));
            
            serializedGenerator.ApplyModifiedProperties();
            
            if (GUI.changed)
            {
                EditorUtility.SetDirty(customGenerator);
            }
        }
    }
}
```

### 4. Add the VoxelGeneratorAttr Attribute

The `VoxelGeneratorAttr` attribute is used to display your generator in the inspector dropdown:

- **name**: Display name in the UI
- **order**: Sort order (lower numbers appear first)

```csharp
[VoxelGeneratorAttr("My Custom Generator", 10)]
public class MyCustomVoxelGeneratorEditor : AVoxelGeneratorEditor
{
    // ...
}
```

### 5. Create an Asset Instance

Once your generator is implemented:

1. Right-click in the Project window
2. Select **Create > Digger > Voxel Generators > My Custom Generator**
3. Name your asset (e.g., "MyCustomGenerator")
4. Select the Digger Master GameObject
5. In the Inspector, go to the **Generation** tab
6. Assign your newly created generator asset to the "Voxel Generator" field

## Understanding Voxel Properties

The `Voxel` struct has several important properties you can set:

### Value (SDF)
```csharp
voxel.SetValue(value, maxAbsValue);
// or use constructor:
var voxel = new Voxel(value, maxAbsValue);
```
The signed distance field value. Negative = inside volume, Positive = outside volume, Zero = on surface.

### Textures
```csharp
voxel.FirstTextureIndex = 0;      // First texture (0-31)
voxel.SecondTextureIndex = 1;     // Second texture for blending (0-31)
voxel.NormalizedTextureLerp = 0.5f; // Blend between textures (0-1)
```

### Destructibility
```csharp
// Voxels can be made indestructible using SetMaxValue
// When MaxValue >= 32 (normalized ~0.5), the voxel becomes indestructible
voxel.SetMaxValue(maxAbsValue, maxAbsValue);  // Makes voxel indestructible

// Check if voxel is indestructible
bool isIndestructible = voxel.IsIndestructible;
```

### Alteration State
```csharp
voxel.Alteration = Voxel.OnSurface;  // See constants in Voxel struct
```
Possible values:
- `Voxel.Unaltered` (0): Not altered, no mesh generated
- `Voxel.OnSurface` (1): On terrain surface
- `Voxel.NearBelowSurface` (2): Altered, near surface, below
- `Voxel.NearAboveSurface` (3): Altered, near surface, above
- `Voxel.FarBelowSurface` (4): Altered, far from surface, below
- `Voxel.FarAboveSurface` (5): Altered, far from surface, above
- `Voxel.Hole` (6): Hole cut in terrain

## Performance Tips

### Use Burst Compiler
Always add the `[BurstCompile]` attribute to your job struct for optimal performance:

```csharp
[BurstCompile(CompileSynchronously = true, FloatMode = FloatMode.Fast, OptimizeFor = OptimizeFor.Performance)]
public struct MyCustomVoxelGenerationJob : IJobParallelFor
```

### Avoid Managed Objects
Jobs cannot access managed objects. Use only:
- Primitive types (float, int, etc.)
- Unity.Mathematics types (float3, int2, etc.)
- NativeArray and other native collections

### Dispose Native Arrays Properly
If you create temporary NativeArrays, dispose them after the job completes:

```csharp
var tempArray = new NativeArray<float>(size, Allocator.TempJob);
var handle = job.Schedule(count, 64);
handle = tempArray.Dispose(handle); // Dispose after job
```

### Batch Size
The second parameter in `Schedule()` is the batch size. Common values:
- **64**: Good default for most cases
- **512**: For very simple jobs
- **32**: For more complex jobs

```csharp
return jobData.Schedule(voxels.Length, 64);
```

## Common Use Cases

### Example 1: Height-Based Texture
```csharp
public void Execute(int index)
{
    // ... calculate voxel position and SDF ...
    
    var worldHeight = voxelAltitude;
    
    if (worldHeight < 50f)
        voxel.FirstTextureIndex = 0; // Sand
    else if (worldHeight < 100f)
        voxel.FirstTextureIndex = 1; // Grass
    else
        voxel.FirstTextureIndex = 2; // Rock
    
    Voxels[index] = voxel;
}
```

### Example 2: Procedural Noise Destructibility
```csharp
public void Execute(int index)
{
    // ... basic setup ...

    var noiseValue = noise.cnoise(p / NoiseScale);

    // Make voxels indestructible in certain noise regions
    if (noiseValue > 0.3f)
        voxel.SetMaxValue(HeightmapScale.y, HeightmapScale.y); // Indestructible

    Voxels[index] = voxel;
}
```

### Example 3: Biome-Based Generation
```csharp
public void Execute(int index)
{
    // ... basic setup ...

    // Use 2D noise for biome selection
    var biomeNoise = noise.cnoise(new float2(p.x, p.z) / 200f);

    if (biomeNoise < -0.3f)
    {
        // Desert biome - destructible sand
        voxel.FirstTextureIndex = 0;
        // Voxel remains destructible (default)
    }
    else if (biomeNoise < 0.3f)
    {
        // Plains biome - destructible dirt
        voxel.FirstTextureIndex = 1;
        // Voxel remains destructible (default)
    }
    else
    {
        // Mountain biome - indestructible rock
        voxel.FirstTextureIndex = 2;
        voxel.SetMaxValue(HeightmapScale.y, HeightmapScale.y); // Indestructible
    }

    Voxels[index] = voxel;
}
```

## Using the Advanced Voxel Generator Layer System

The Advanced Voxel Generator provides a powerful layer-based approach to voxel generation without writing code.

### Depth Layers

Depth layers assign properties based on how deep a voxel is below the terrain surface.

**Example Configuration**:
```
Layer 1: minDepth = 0,  texture = 0 (grass),  destructible = true
Layer 2: minDepth = 5,  texture = 1 (dirt),   destructible = true
Layer 3: minDepth = 10, texture = 2 (stone),  destructible = false
```

This creates:
- 0-5 units deep: Grass texture, can be dug
- 5-10 units deep: Dirt texture, can be dug
- 10+ units deep: Stone texture, indestructible (cannot be modified by digging)

### Noise Layers

Noise layers add procedural variation and can override or modify the properties set by depth layers.

**Example: Large-Scale Indestructible Patches**
```
scale = 20
octaves = 2
persistence = 0.5
destructible = false
blendMode = Add
threshold = 0.3
textureIndex = -1 (no override)
```

This creates scattered indestructible patches across the terrain where noise exceeds the threshold.

**Example: Indestructible Rocky Veins**
```
scale = 8
octaves = 3
persistence = 0.6
destructible = false
blendMode = Replace
threshold = 0.5
textureIndex = 3 (rock texture)
```

This creates indestructible rocky veins that appear where noise exceeds 0.5. The effect smoothly fades in around the threshold to prevent holes in the surface.

### Blend Modes Explained

**Replace Mode**:
- Fully replaces the destructibility setting when noise influence is high
- Use for distinct features like ore veins or indestructible rock formations

**Add Mode**:
- Only makes voxels indestructible (if the layer's destructible = false), never makes them destructible
- Use when you want to add indestructible regions without affecting already indestructible areas

### Layer Application Order

1. **Depth layers** are evaluated first to set base texture and destructibility
2. **Noise layers** are then applied in the order they appear in the list
3. Each noise layer can:
   - Modify destructibility based on its blend mode
   - Override texture if textureIndex >= 0
   - Only activate if noise value exceeds threshold

### Tips for Layer Design

**For Realistic Terrain**:
1. Start with 2-3 depth layers for basic material stratification
2. Make deeper layers indestructible to simulate bedrock
3. Add a noise layer with Add mode for scattered indestructible rocks

**For Stylized/Fantasy Terrain**:
1. Use fewer depth layers with distinct textures
2. Add noise layers with high thresholds (0.3-0.7) for dramatic, sparse features
3. Use Replace mode with texture overrides for magical veins or crystal formations

**For Performance**:
- Limit to 3-4 depth layers and 2-3 noise layers maximum
- Fewer octaves = better performance (1-2 is often sufficient)
- Higher thresholds mean fewer calculations (layers only activate when threshold is met)

## Debugging

To debug your generator:

1. Remove `[BurstCompile]` temporarily to enable breakpoints
2. Use `UnityEngine.Debug.Log()` (only works without Burst)
3. Test with a small chunk size for faster iteration
4. Use Unity Profiler to measure performance

## Additional Resources

- [Unity Job System Documentation](https://docs.unity3d.com/Manual/JobSystem.html)
- [Burst Compiler Documentation](https://docs.unity3d.com/Packages/com.unity.burst@latest)
- [Unity Mathematics Package](https://docs.unity3d.com/Packages/com.unity.mathematics@latest)

## Support

If you need help creating custom voxel generators:
- Check the included examples (SimpleVoxelGenerator and AdvancedVoxelGenerator)
- Join the Digger Discord community: https://discord.gg/C2X6C6s
- Visit the documentation: https://ofux.github.io/Digger-Documentation/

