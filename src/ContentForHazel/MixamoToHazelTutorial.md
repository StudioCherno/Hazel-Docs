<div class="title"> 
    <img src="/res/Hazel-IconLogo-2023.png" alt="Hazel Logo" />
    <h1> Mixamo to Hazel </h1>
    <img src="/res/MixamoLogo.png" alt="Mixamo Logo" />
</div>
<div class="page-metadata">
    <h4> Last edited: 10th May 2023 </h4>
</div>


Importing Mixamo Animation to Hazel is super simple! This guide will outline what is a very intuitive process anyway!

**1. Select the character or upload your own to Mixamo**

In this example we will be downloading one of the default characters the **Y bot**.

![Adding an Armature](/res/MixamoToBlender/Ybot.png)

**2. Downloading skinned mesh + animations**

- i. Navigate to the animations tab 
- ii. Choose the first animation that you would like
- iii. Download the mesh with the animation and the following settings:

![Mixamo Export Settings](/res/MixamoToBlender/DownloadSettings.png)

Frames per second can obviously be changed, but its important that under **Skin**, **With Skin** is selected.

Repeat steps i to iii for every animation that you would like to import. At the time of this guide being written, Hazel doesn't support *Animation only* import, with time that should be added but for the time being, **every animation needs to be downloaded from mixamo *with skin*.**

**3. Importing to Hazel**

Open up your project and import the Meshes, starting with your main one. In the future Hazel's workflow with importing assets is likely to change, but at the time of writing this this is the way the import looks like.

These are the settings for the main mesh:

![Hazel Import Settings](/res/MixamoToBlender/HazelImportMesh1.png)

These are the settings for the "animation meshes": 

![Hazel Import Settings #2](/res/MixamoToBlender/HazelImportMesh2.png)

**4. Delete unnecessary meshes and entities**

You can now delete all the `.hmesh` files that were created and all the *t-posing entities* in the scene. 

**5. Adding animations to the animation controller**

All that's left is to add the animation to the animation controller and you're golden!

- i. Open up the Animation controller, you can do this by navigating to `Assets/Animation` and finding the relevant `.hanimc` file or by selecting the top level entity and double clicking on the assigned field of the Animation Component:

![Animation Component](/res/MixamoToBlender/AnimationComponent.png)

- ii. Greeted with a window that looks somewhat like the window below, press the `Add +` button to create an additional state

![Animation Controller](/res/MixamoToBlender/AnimationController.png)

- iii. Assign the animation to the new state, by clicking on the `Animation` field.

- iv. Repeat steps ii. and iii. as many times as required to get all you animations into the Animation Controller.

With this, you now know how to import Animation and Rigged Meshes from Mixamo to Hazel, to change the current animation simply adjust the `Animation Index` in the Animation Component. 


