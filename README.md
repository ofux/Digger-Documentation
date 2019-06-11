## Introduction

Digger is a simple yet powerful tool to create natural caves and overhangs on your Unity terrains directly from the Unity editor.

More and more AAA games add gentle overhangs and caves to their environment to make it more realistic, more interesting and more diversified.
A common way to do this is to create 3D models with an external modeling tool, and then place them manually over the terrain and blend them with it. This is tedious, unpractical and inefficient unless you have an army of 3D artists and level designers... and even in that case, you'd probably want them to focus on more valuable work.
This is where Digger comes to action. No more external tool required, no more loss of time, no more headache. It lets you create caves and overhangs on your terrain directly within the scene view, in a few clicks.

With this tool, you will be able to:
- Dig in your Unity terrain just like if it was a smooth voxel terrain.
- Create overhangs (the opposite of digging).
- Apply different textures on the overhangs, in the caves, etc.
- ***NEW:*** Dig in real-time at runtime.

However, you won't be able to:
- Generate caves procedurally. If you need this feature, you should get a full voxel-based terrain solution, like [Ultimate Terrains](https://assetstore.unity.com/packages/tools/terrain/ultimate-terrains-voxel-terrain-engine-31100).

Digger can be downloaded from the [Asset Store](https://assetstore.unity.com/packages/tools/terrain/digger-caves-overhangs-135178).


## Getting Started

Digger is very easy to setup, but it requires you to install 3 packages.

**First, make sure your project uses *.NET 4.x* as [shown here](https://docs.unity3d.com/Manual/ScriptingRuntimeUpgrade.html).**

Then, open the Package Manager (menu *Windows > Package Manager*).

<img src="assets/img/package-manager-menu.png" alt="Package Manager" width="180"/>

In the Package Manager window, click on "Advanced" and enable "Show preview packages".

<img src="assets/img/show-preview-packages.png" alt="Package Manager" width="180"/>

Install the latest version of the packages `Mathematics`, `Collections` and `Burst`.

Then, import Digger into your project (from the Asset Store).

From now on, Digger should be imported and **you should not have any error in the console**. There should be a new menu: *Tools > Digger*.

**You are ready to use Digger.**

Open a scene with a terrain (you can open "simple-scene" in *Assets/Digger/Demo* for example) or create a terrain in a new scene. Configure your terrain layers as usual and modify your terrain as usual (raise or lower height, etc.). Make sure all your terrain layers have both a texture and a normal map.

Once this is done, click on *Tools > Digger > Setup terrains*. This will prepare texture arrays for Digger material, add Digger System to all terrains in the scene and add Digger Master object.

`ADD IMG HERE`

Click on Digger Master in the hierarchy of the scene to display the Digger Master inspector.

`ADD IMG HERE`

The Digger Master inspector looks like this:

`ADD IMG HERE`

To start digging, just click somewhere on your terrain!

Note: the first time you dig, Unity will freeze during about 1s. This is because the Burst compiler needs to compile internal Digger jobs.

### Details of each field

- Scene data folder: Digger will automatically persist data in Assets/DiggerData/<scene-data-folder>. By default, this is the name of the scene. You can change it if you want, but don’t forget to rename the directory as well.
- Resolution: by default, Digger generates meshes that fit to terrain’s mesh, which is directly related to heightmap resolution (and terrain size), but you can tell Digger to use a finer resolution (respectively, 2 times, 4 times or 8 times the terrain’s mesh resolution) thanks to this parameter.
- Screen Relative Transition Height of LODs: adjust these sliders to tell at which distance from the camera the cave/overhangs meshes should switch between LODs.
The bigger it is, the closer you will have to be from the object to get the highly detailed mesh.
- Collider LOD: lets you change the Level Of Details of the collider mesh. If you want accurate collisions that fit exactly to the ground, set it to 0. If you want better performance and don’t mind to have a lower accuracy, increase it to 1 or 2.
- Brush: lets you choose the action to perform between digging terrain, raising overhangs, reseting (reset to terrain height but do not restore terrain details objects), or painting.
- Opacity: the speed at which you will dig/add mater to the terrain. This has no effect on reset and paint brushes.
- Size: the size of the brush.
- List of textures: lets you choose which texture to use.
- Clear: this will clear all modifications you’ve made to the terrains with Digger, but it won’t restore terrain details objects. This cannot be undone.
- Sync & Refresh: forces Digger to synchronize with terrains and recompute everything. This is useful if you changed terrain textures or heights.


## Integration with CTS

CTS (Complete Terrain Shaders) is supported by Digger, but as things stand, you won’t be able to change textures in caves or on overhangs. It will pick-up the terrain texture. Future versions of CTS might allow to fix this.


## Upgrade guide

When a new version of Digger is released, you will probably want to install it. Just keep in mind that some updates might contain breaking changes that won't work with previous Digger saved data. In such case, it is clearly mentioned in the release note of the new version.

Follow these steps to upgrade your version of Digger:
- Completely backup your project (including **full copy-paste** of *DiggerData* folder)
- Remove *Digger* folder in Assets (but do **not** remove *DiggerData* folder)
- Import the new version
- Open you scene(s) and click on “Sync & Refresh” button


`ADD IMG HERE`

`ADD IMG HERE`
