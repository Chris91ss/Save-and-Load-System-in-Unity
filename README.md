# Save-and-Load-System-in-Unity
## Save &amp; Load System in Unity. A flexible system that you can expand upon.


## How to use it in your own project:

- Create an empty oject in your project and add the SaveLoadSystem.cs script to it, by right clicking on the script in the context menu on the "Save" or "Load" buttons you can save or load your data. 
- Remember to inherit from the "ISaveable" script (look at the example in the .txt file named "Add This To The Script You Want To Save Data From".
- Add the .txt file named "Add This To The Script You Want To Save Data From", from there you can chose what data you want to save by adding it in the "struct SaveData", saving in the "SaveState()" and then loading the data in the "LoadState()"	
- Add the script "SaveableEntity" to the object you want to save data from, an example is the Player, after that generate an ID by right clicking the script in the context menu and then click "Generate ID". We are generating an ID so that our system knows from witch entity to save data from. For example, if we have multiple players, all with different stats, we can save and load the data for each one by having an ID.
