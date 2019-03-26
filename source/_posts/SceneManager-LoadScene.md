---
layout: title
title: SceneManager.LoadScene
date: 2019-03-11 21:27:07
tags: Unity
---
public static void LoadScene(int sceneBuildIndex, SceneManagement.LoadSceneMode mode = LoadSceneMode.Single);

public static void LoadScene(string sceneName, SceneManagement.LoadSceneMode mode = LoadSceneMode.Single);

<!--more-->
Parameters：

sceneName	Name or path of the Scene to load.
sceneBuildIndex	Index of the Scene in the Build Settings to load.

mode	Allows you to specify whether or not to load the Scene additively. See LoadSceneMode for more information about the options.

Description：

Loads the Scene by its name or index in Build Settings.

Note: In most cases, to avoid pauses or performance hiccups while loading, you should use the asynchronous version of this command which is: LoadSceneAsync.

When using SceneManager.LoadScene, the loading does not happen immediately, it completes in the next frame. This semi-asynchronous behavior can cause frame stuttering and can be confusing because load does not complete immediately.

Since loading is set to complete in the next rendered frame, calling SceneManager.LoadScene forces all previous AsynOperations to complete, even if AsyncOperation.allowSceneActivation is set to false. This can be avoided by using LoadSceneAsync instead.

The given sceneName can either be the Scene name only, without the .unity extension, or the path as shown in the BuildSettings window still without the .unity extension. If only the Scene name is given this will load the first Scene in the list that matches. If you have multiple Scenes with same name but different paths, you should use the full path.

Note that sceneName is case insensitive, except when you load the Scene from an AssetBundle.

For opening Scenes in the Editor see EditorSceneManager.OpenScene.

SceneA can additively load SceneB multiple times. The regular name is used for each loaded scene. If SceneA loads SceneB ten times each SceneB with have the same name. Finding a particular added scene is not possible.
```cs
using UnityEngine;
using UnityEngine.SceneManagement;

public class ExampleClass : MonoBehaviour
{
    void Start()
    {
        // Only specifying the sceneName or sceneBuildIndex will load the Scene with the Single mode
        SceneManager.LoadScene("OtherSceneName", LoadSceneMode.Additive);
    }
}
// Load an assetbundle which contains Scenes.
// When the user clicks a button the first Scene in the assetbundle is
// loaded and replaces the current Scene.
```

```cs
using UnityEngine;
using UnityEngine.SceneManagement;

public class LoadScene : MonoBehaviour
{
    private AssetBundle myLoadedAssetBundle;
    private string[] scenePaths;

    // Use this for initialization
    void Start()
    {
        myLoadedAssetBundle = AssetBundle.LoadFromFile("Assets/AssetBundles/scenes");
        scenePaths = myLoadedAssetBundle.GetAllScenePaths();
    }

    void OnGUI()
    {
        if (GUI.Button(new Rect(10, 10, 100, 30), "Change Scene"))
        {
            Debug.Log("Scene2 loading: " + scenePaths[0]);
            SceneManager.LoadScene(scenePaths[0], LoadSceneMode.Single);
        }
    }
}
```
The following two script examples show how LoadScene can load Scenes from Build Settings. LoadSceneA uses the name of the Scene to load. LoadSceneB uses the number of the Scene to load. The scripts work together.

LoadSceneA file.
```cs
// SceneA.
// SceneA is given the sceneName which will
// load SceneB from the Build Settings

using UnityEngine;
using UnityEngine.SceneManagement;

public class LoadScenesA : MonoBehaviour
{
    void Start()
    {
        Debug.Log("LoadSceneA");
    }

    public void LoadA(string scenename)
    {
        Debug.Log("sceneName to load: " + scenename);
        SceneManager.LoadScene(scenename);
    }
}
```
LoadSceneB file.

```cs
// SceneB.
// SceneB is given the sceneBuildIndex of 0 which will
// load SceneA from the Build Settings

using UnityEngine;
using UnityEngine.SceneManagement;

public class LoadScenesB : MonoBehaviour
{
    void Start()
    {
        Debug.Log("LoadSceneB");
    }

    public void LoadB(int sceneANumber)
    {
        Debug.Log("sceneBuildIndex to load: " + sceneANumber);
        SceneManager.LoadScene(sceneANumber);
    }
}
```
public static SceneManagement.Scene LoadScene(int sceneBuildIndex, SceneManagement.LoadSceneParameters parameters);
public static SceneManagement.Scene LoadScene(string sceneName, SceneManagement.LoadSceneParameters parameters);
Parameters
sceneName	Name or path of the Scene to load.
sceneBuildIndex	Index of the Scene in the Build Settings to load.
parameters	Various parameters used to load the Scene.
Returns
Scene A handle to the Scene being loaded.

Description
Loads the Scene by its name or index in Build Settings.

An example using two scenes called Scene1 and Scene2. ExampleScript1.cs is for scene1 and ExampleScript2.cs is for scene2.
```cs
using UnityEngine;
using UnityEngine.SceneManagement;

// This is scene1.  It loads 3 copies of scene2.
// Each copy has the same name.

public class ExampleScript1 : MonoBehaviour
{
    private Scene scene;

    private void Start()
    {
        var parameters = new LoadSceneParameters(LoadSceneMode.Additive);

        scene = SceneManager.LoadScene("scene2", parameters);
        Debug.Log("Load 1 of scene2: " + scene.name);
        scene = SceneManager.LoadScene("scene2", parameters);
        Debug.Log("Load 2 of scene2: " + scene.name);
        scene = SceneManager.LoadScene("scene2", parameters);
        Debug.Log("Load 3 of scene2: " + scene.name);
    }
}
```
Scene2:
```cs
using UnityEngine;

// create a randomly placed cube

public class ExampleScript2 : MonoBehaviour
{
    private void Start()
    {
        GameObject cube = GameObject.CreatePrimitive(PrimitiveType.Cube);
        cube.transform.position = new Vector3(Random.Range(-5.0f, 5.0f), 0.0f, Random.Range(-5.0f, 5.0f));
    }
}
```