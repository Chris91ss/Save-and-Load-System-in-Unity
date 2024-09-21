# Save and Load System in Unity

## Overview
A flexible save-and-load system for Unity that allows you to easily save and load game data. The system can be expanded and adapted for various use cases. The core functionality revolves around identifying objects and saving/loading their states with unique IDs.

## Features
- Save and load system for multiple entities.
- Easily expandable to save data from various objects.
- Uses unique IDs for each object to ensure the correct data is loaded.
- Simple to integrate with any game object.
- Saves and loads data using **binary serialization**.

## Table of Contents
1. [Getting Started](#getting-started)
2. [How to Use It](#how-to-use-it)
3. [Code Overview](#code-overview)
    - [ISaveable Interface](#isaveable-interface)
    - [SaveableEntity](#saveableentity)
    - [SaveLoadSystem](#saveloadsystem)
4. [Example](#example)
5. [License](#license)

---

## Getting Started

### Prerequisites
- Unity 2019.4 or later.
- Basic understanding of Unity's component system.

---

## How to Use It

### Step-by-Step Instructions:

1. **Set up the SaveLoadSystem**:
    - Create an empty GameObject in your scene and add the `SaveLoadSystem.cs` script to it.
    - The system will use Unity's **Context Menu** for saving and loading.

2. **Add the SaveableEntity to Objects**:
    - Add the `SaveableEntity.cs` script to any GameObject that you want to save or load data from (e.g., the Player).
    - Right-click on the `SaveableEntity` script in the Inspector and select **Generate ID**. This will create a unique ID for the object.

3. **Mark Objects as Saveable**:
    - Ensure that the script where you want to save the data from implements the `ISaveable` interface. This will allow the system to capture and store the data.
    - Use the `.txt` file example ("Add This To The Script You Want To Save Data From") to see how to save and load custom data for each entity.

4. **Customize Saving and Loading**:
    - In the `ISaveable` interface, specify what data you want to save in the `SaveState()` function and how it should be restored using the `LoadState()` function.
    - Customize the time at which the system saves and loads, by calling the methods from UI elements or game events.

---

## Code Overview

### ISaveable Interface
The `ISaveable` interface must be implemented by any script that wants to save data. This interface ensures each object has methods to **save** and **load** its state.

```csharp
public interface ISaveable 
{
    object SaveState();
    void LoadState(object state);
}
```

### SaveableEntity
`SaveableEntity.cs` is responsible for managing the unique ID of each object and providing methods to save and load the data associated with it.

```csharp
public class SaveableEntity : MonoBehaviour
{
    [SerializeField] private string id;
    public string ID => id;

    [ContextMenu("Generate ID")]
    private void GenerateID()
    {
        id = Guid.NewGuid().ToString();
    }

    public object SaveState()
    {
        var state = new Dictionary<string, object>();
        foreach (var saveable in GetComponents<ISaveable>())
        {
            state[saveable.GetType().ToString()] = saveable.SaveState();
        }
        return state;
    }

    public void LoadState(object state)
    {
        var stateDictionary = (Dictionary<string, object>)state;
        foreach (var saveable in GetComponents<ISaveable>())
        {
            string typeName = saveable.GetType().ToString();
            if (stateDictionary.TryGetValue(typeName, out object savedState))
            {
                saveable.LoadState(savedState);
            }
        }
    }
}
```

### SaveLoadSystem
`SaveLoadSystem.cs` handles the actual saving and loading of game data, by serializing the data to a file and deserializing it when needed.

```csharp
public class SaveLoadSystem : MonoBehaviour
{
    public string SavePath => $"{Application.persistentDataPath}/save.txt";

    [ContextMenu("Save")]
    void Save()
    {
        var state = LoadFile();
        SaveState(state);
        SaveFile(state);
    }

    [ContextMenu("Load")]
    void Load()
    {
        var state = LoadFile();
        LoadState(state);
    }

    public void SaveFile(object state)
    {
        using (var stream = File.Open(SavePath, FileMode.Create))
        {
            var formatter = new BinaryFormatter();
            formatter.Serialize(stream, state);
        }
    }

    Dictionary<string, object> LoadFile()
    {
        if (!File.Exists(SavePath))
        {
            Debug.Log("No save file found");
            return new Dictionary<string, object>();
        }

        using (FileStream stream = File.Open(SavePath, FileMode.Open))
        {
            var formatter = new BinaryFormatter();
            return (Dictionary<string, object>)formatter.Deserialize(stream);
        }
    }

    void SaveState(Dictionary<string, object> state)
    {
        foreach (var saveable in FindObjectsOfType<SaveableEntity>())
        {
            state[saveable.ID] = saveable.SaveState();
        }
    }

    void LoadState(Dictionary<string, object> state)
    {
        foreach (var saveable in FindObjectsOfType<SaveableEntity>())
        {
            if (state.TryGetValue(saveable.ID, out object savedState))
            {
                saveable.LoadState(savedState);
            }
        }
    }
}
```

---

## Example

### Example Script Using the Save System:

```csharp
public class Player : MonoBehaviour, ISaveable     
{
    public float playerHealth, playerMaxHealth;

    public object SaveState()  // Save the data
    {
        return new SaveData()
        {
            Health = playerHealth,
            maxHealth = playerMaxHealth
        };
    }

    public void LoadState(object state)  // Load the data
    {
        var saveData = (SaveData)state;
        playerHealth = saveData.Health;
        playerMaxHealth = saveData.maxHealth;
    }

    private struct SaveData  // Data to save
    {
        public float Health;
        public float maxHealth;
    }
}
```

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
