<div class="title"> 
    <img src="/res/HazelGradientLogo-Square.png" alt="Hazel Logo" />
    <h1> Fixing Broken Prefab References </h1>
</div>

You will know when a prefab is broken when you see it's name and it's type rendered in red in the Scene Hierarchy Panel. It should look like this:

![Broken Prefab in Scene Hierarchy Panel](/res/BrokenPrefabs/BrokenPrefab.png)

If thats the case then by following the next couple of steps you will be able to fix it no problem!

## 1. Enable "Advanced Mode"

1. Navigate to `Edit -> Application Settings -> Hazelnut`
2. Enable "Advance Mode" if it wasn't on already.

    ![Advanced Mode](/res/BrokenPrefabs/AdvancedMode.png)

## 2. Find the Prefab in the Asset Manager

1. Navigate to `View -> Asset Manager`
2. Enable the "Allow Editing" option.
3. Search for your prefab by name. Careful name here means name on disk, i.e. Content Browser, the name in the Scene Hierarchy Panel may not necessarily be the same.
4. Copy the Prefab's Handle.

    ![Asset Manager](/res/BrokenPrefabs/AssetManager.png)

## 3. Modifying the root Prefab's ID.

1. Select the entity in the Scene Hierarchy Panel.
2. The topmost Component in the Entity Properties Panel, should be the prefab component.

    ![Prefab Component](/res/BrokenPrefabs/PrefabComponent.png)

3. Replace the Prefab ID with the Prefab's Handle that we copied in the last step.
4. Make sure to save the scene so you don't loose your changes.
5. That should fix your broken prefab reference!

    ![Fixed Prefab](/res/BrokenPrefabs/FixedPrefab.png)