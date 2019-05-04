---
layout: title
title: Object.DontDestroyOnLoad
date: 2019-03-11 20:43:27
categories: Unity
tags: 官方文档
---
思考并回答以下问题：
这个函数是什么意思？为什么要使用这个函数？如何使用？


<!--more-->

public static void DontDestroyOnLoad(Object target);

** Parameters： **  

target：	An Object not destroyed on Scene change.

** Description： **

Do not destroy the target Object when loading a new Scene.

The load of a new Scene destroys all current Scene objects. Call Object.DontDestroyOnLoad to preserve an Object during level loading. If the target Object is a component or GameObject, Unity will also preserve all of the Transform’s children.

Object.DontDestroyOnLoad does not return a value. Change the argument type using the typeof operator.

The following example script uses Object.DontDestroyOnLoad. The example has scene1 which starts playing background music from an AudioSource. The music continues when scene2 loads. Switch between scenes using a button.

To implement this example, create two new Scenes, named scene1 and scene2. Open scene1 and add the SceneSwap.cs script to an empty GameObject and name it Menu. Next, add DontDestroy.cs to a new GameObject and name it BackgroundMusic. Add an AudioSource to BackgroundMusic - Add Component->Audio->Audio Source - and import an AudioClip into your Project. Assign the AudioClip to the AudioSource's AudioClip field. Create a tag, call it music, and add it to BackgroundMusic. Switch to scene2. Again add SceneSwap.cs to a new GameObject and name it Menu. Save the Project. Return to scene1 and run the Project from the Editor.

> SceneSwap.cs script:

```cs
using UnityEngine;
using UnityEngine.SceneManagement;

// Object.DontDestroyOnLoad example.
//
// Two scenes call each other. This happens when OnGUI button is clicked.
// scene1 will load scene2; scene2 will load scene1. Both scenes have
// the Menu GameObject with the SceneSwap.cs script attached.
//
// AudioSource plays an AudioClip as the game runs. This is on the
// BackgroundMusic GameObject which has a music tag.  The audio
// starts in AudioSource.playOnAwake. The DontDestroy.cs script
// is attached to BackgroundMusic.

public class SceneSwap : MonoBehaviour
{
    private void OnGUI()
    {
        int xCenter = (Screen.width / 2);
        int yCenter = (Screen.height / 2);
        int width = 400;
        int height = 120;

        GUIStyle fontSize = new GUIStyle(GUI.skin.GetStyle("button"));
        fontSize.fontSize = 32;

        Scene scene = SceneManager.GetActiveScene();

        if (scene.name == "scene1")
        {
            // Show a button to allow scene2 to be switched to.
            if (GUI.Button(new Rect(xCenter - width / 2, yCenter - height / 2, width, height), "Load second scene", fontSize))
            {
                SceneManager.LoadScene("scene2");
            }
        }
        else
        {
            // Show a button to allow scene1 to be returned to.
            if (GUI.Button(new Rect(xCenter - width / 2, yCenter - height / 2, width, height), "Return to first scene", fontSize))
            {
                SceneManager.LoadScene("scene1");
            }
        }
    }
}
```
> DontDestroy.cs script:

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// Object.DontDestroyOnLoad example.
//
// This script example manages the playing audio. The GameObject with the
// "music" tag is the BackgroundMusic GameObject. The AudioSource has the
// audio attached to the AudioClip.

public class DontDestroy : MonoBehaviour
{
    void Awake()
    {
        GameObject[] objs = GameObject.FindGameObjectsWithTag("music");

        if (objs.Length > 1)
        {
            Destroy(this.gameObject);
        }

        DontDestroyOnLoad(this.gameObject);
    }
}
```

{% asset_img 3.png %}

{% asset_img 1.png %}

{% asset_img 2.png %}