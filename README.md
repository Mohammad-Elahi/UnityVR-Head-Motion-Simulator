
# [VR-Head-Motion-Simulator](https://github.com/Mohammad-Elahi/VR-Head-Motion-Simulator)
![VR-Head-Motion-Simulator](https://github.com/Mohammad-Elahi/VR-Head-Motion-Simulator/assets/93424032/a3558de8-a5e8-41ae-b578-8bd5a4722771)

# Description
This repository contains a C# script that simulates head motion in a Unity VR environment using head data which is timeframe, position:x,y,z, and Orientation:qx,qy,qz,qw that exist in the CSV file.

## Requirements
Unity Editor version 2022.3.19f1. If you want to download the repository and run it in your Unity, you should adjust your Unity editor version to 2022.3.19f1. Alternatively, you can use the instructions and C# code with your own Unity editor version.

## Setup

1. Clone the repository.
2. Open the project in Unity Editor.
3. Replace the `csvFilePath` in the `HeadMotionSimulator.cs` script in the Assets with the path of your `head_motion.csv` file.

## Usage

The script should be attached to a GameObject named `VRHeadset` in the Unity hierarchy. If you want to use the code with your own Unity editor version, follow these steps:

1. Create a GameObject and name it `VRHeadset`.
2. Attach the `HeadMotionSimulator.cs` script to the `VRHeadset` GameObject.
3. Right-click on the `VRHeadset` GameObject in the hierarchy and create a Camera object.
4. In the Inspector of the camera, add `Tracked Pose Drive` and adjust the following options:
   - Device: Generic XR Device
   - Pose Source: Center Eye
   - Tracking Type: Rotation and Position
   - Update Type: Update and Before Render

## C# Code

The `HeadMotionSimulator.cs` script reads the `head_motion.csv` file, which includes time and x, y, z positions and qx, qy, qz, qw orientations. The script then simulates the head motion in the VR environment based on this data.

```csharp
using UnityEngine;
using System.Collections;
using System.IO;
using System.Collections.Generic;

public class HeadMotionSimulator : MonoBehaviour
{
    public string csvFilePath = "your_head_motion.csv_path";
    private List<HeadMotionData> motionData;
    private int currentIndex = 0;
    private float timeBetweenRows = 0.001f; // 1 millisecond

    void Start()
    {
        // Read and parse the .csv file
        motionData = ReadCSVFile(csvFilePath);
        Time.timeScale = 1; // Set the time scale to 1 for one millisecond runtime between rows
    }

    void Update()
    {
        if (motionData != null && motionData.Count > 0)
        {
            // Update cube position and rotation based on the motion data
            transform.position = new Vector3(motionData[currentIndex].x, motionData[currentIndex].y, motionData[currentIndex].z);
            transform.rotation = new Quaternion(motionData[currentIndex].qx, motionData[currentIndex].qy, motionData[currentIndex].qz, motionData[currentIndex].qw);

            // Update currentIndex for the next frame
            currentIndex++;

            // Check if this is the last row
            if (currentIndex >= motionData.Count)
            {
                // Stop the program
                Application.Quit();
            }

            // Wait for 1 milisecond before processing the next row
            StartCoroutine(WaitForNextRow());
        }
    }

    private IEnumerator WaitForNextRow()
    {
        yield return new WaitForSeconds(timeBetweenRows);
    }

    private List<HeadMotionData> ReadCSVFile(string filePath)
    {
        List<HeadMotionData> data = new List<HeadMotionData>();
        if (File.Exists(filePath))
        {
            // Read the lines of the .csv file
            string[] lines = File.ReadAllLines(filePath);
            for (int i = 1; i < lines.Length; i++) // Start from 1 to skip the header
            {
                string[] values = lines[i].Split(',');
                if (values.Length >= 8) // Ensure all columns are present
                {
                    HeadMotionData motion = new HeadMotionData();
                    // Parse and assign the values to the HeadMotionData object
                    motion.time = float.Parse(values[0]);
                    motion.x = float.Parse(values[1]);
                    motion.y = float.Parse(values[2]);
                    motion.z = float.Parse(values[3]);
                    motion.qx = float.Parse(values[4]);
                    motion.qy = float.Parse(values[5]);
                    motion.qz = float.Parse(values[6]);
                    motion.qw = float.Parse(values[7]);
                    data.Add(motion);
                }
                else
                {
                    Debug.LogError("Invalid data in the .csv file at line " + (i + 1));
                }
            }
        }
        else
        {
            Debug.LogError("File not found: " + filePath);
        }
        return data;
    }
}

public class HeadMotionData
{
    public float time;
    public float x;
    public float y;
    public float z;
    public float qx;
    public float qy;
    public float qz;
    public float qw;
}
```
Note: Replace the `your_head_motion.csv_path` with the path of your `head_motion.csv` file.

# Author
Mohammad Elahi, TU Dresden, Vodafone Chair, mohammad.elahi@mailbox.tu-dresden.de
