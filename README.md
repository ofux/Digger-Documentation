# Digger Documentation

Digger is a simple yet powerful tool to create natural caves and overhangs on your Unity terrains directly from the Unity editor.

There are two versions of Digger: **Digger** and **Digger PRO**.
Digger PRO has all the features of Digger plus realtime/in-game editing support.

## Support

To get support, please join us on **Discord**: https://discord.gg/C2X6C6s

## Installation

1. Download and import Digger from the [Asset Store](https://assetstore.unity.com/publishers/11530).
2. **When Unity asks you if you want to install required packages, click *Yes*.**
   - Digger requires the `Burst` package to work efficiently.
3. From now on, Digger should be imported and there should not be any error in the console.
4. You should see a new menu: *Tools > Digger*.

### HDRP & URP Support

Digger supports both HDRP and URP. Shaders for these pipelines are included in `Assets/Digger/Shaders`.

- **URP 17+**: Import `Digger-URP17-shaders.unitypackage`.
- **HDRP 17+**: Import `Digger-HDRP17-shaders.unitypackage`.

## Version Control (Git, Unity Version Control, etc.)

**IMPORTANT:** make sure to include the `Assets/DiggerData` folder (**including hidden `.internal` sub-folders** that are present in each scene sub-folders) in your VCS and in your manual backups.

You might also want to add a `.gitignore` file in the `Assets/DiggerData` folder containing this:

```
*.vom_v*
*.vox3_v*
*.asset_*
*.ver
current_version.asset
```

This will avoid Git seeing modifications every time the scene gets loaded.

## Update Guide

**Before importing a new version of Digger, please delete the `Assets/Digger` folder.**
> **Important**: Do **not** remove the `Assets/DiggerData` folder. This is where your cave data is stored.

Also, before updating Digger, it is recommended to backup your project.

**After importing the new version:**
1. Open your scene.
2. Select **Digger Master** in the hierarchy.
3. Click on **Sync & Refresh**.

## Troubleshooting Installation

If you get errors after importing Digger:

1. Open the Package Manager (*Windows > Package Manager*).
2. Ensure the `Burst`, `Collection` and `AI Navigation` packages are installed and up to date.
3. If issues persist, delete `Assets/Digger` and re-import, ensuring you allow Unity to install dependencies.
4. If you still have issues, please join the [Discord server](https://discord.gg/C2X6C6s).

## Table of Contents

- [Getting Started](GettingStarted.md)
- [Editing Guide](Editing.md)
- [Settings](Settings.md)
- [Runtime (PRO only)](Runtime.md)
- [Integrations](Integrations.md)
- [Advanced Operations](AdvancedOperations.md)
- [Custom Voxel Generators](CustomVoxelGenerator.md)
- [FAQ](FAQ.md)

