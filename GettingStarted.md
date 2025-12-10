# Getting Started with Digger

## Setup

1. Open a scene with a Unity Terrain (or create a new one).
2. Configure your terrain layers as usual.
3. Go to menu **Tools > Digger > Setup terrains**.
   - This adds the `Digger System` component to your terrains.
   - It creates a `Digger Master` object in the scene.

## First Steps

1. Click on **Digger Master** in the scene hierarchy to inspect it.
2. In the **Edit** tab (default), ensure **Action** is set to `Dig`.
3. In the Scene View, you will see a brush reticle (sphere or cube).
4. Click on your terrain to start digging!

> **Note**: The first time you dig, Unity might freeze for a few seconds. This is the Burst compiler compiling Digger's background jobs. Subsequent operations will be much faster.

## Sync & Refresh

If you change your terrain textures or modify terrain heights using Unity's built-in tools *after* you have already dug caves:

1. Select **Digger Master**.
2. Go to the **Settings** tab (or look at the bottom of the Edit tab).
3. Click **Sync with terrain(s) & Refresh**.

This ensures Digger's meshes match your new terrain configuration.

