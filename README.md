# UndoPro
UndoPro is a command-based undo system integrated into Unity's default system. This allows devs to use actions for their undo/redo operations without forcing the user into a new undo-workflow!

[Forum Thread](https://community.unity.com/t5/Extensions-OnGUI/WIP-Open-Source-UndoPro-Command-pattern-Undo-integration/m-p/2640947)

##Features
- Extended Callbacks for Undo: Seperate Undo/Redo with record names, OnAddUndoRecord, ...
- API for creating command-based undo records
- Handles most anonymous actions without problems, even using the context!

##How it works
Unity provides only very limited information about the undo system:
- The ID of the current record, but it's not unique, rather than steadily increasing :/
- The current record/group name (not unique)
- With reflection, the complete Undo/Redo stack only by name (not unique)
- UndoRedoPerformed callback

Internally, UndoPro creates a dummy record in the default system when the API for adding a command-based record is called. This is then tracked using the available callback and the commands are executed when they switch from redo- undo stack or the other way around respectively.
A big hurden was the behaviour of the default Undo system: it is nearly unpredictable! Records may duplicate in certain conditions when undone/redone, or vanish. It is very hard, but a requirement to make a solid tracking algorithm. Obviously the addition of new records has also to be detected.

When tracking has been done, a shift value for both undo and redo stack seperately is calculated (remember, records might vanish or duplicate during undo/redo!). The internal records are then updated accordingly and the records to undo/redo are outlined.

On the way of all this, the additional callbacks OnUndoPerformed/OnRedoPerformed/OnAddUndoRecord are called.

##Serialization of Command-based records
Serialization is also a big problem, as actions, and even worse anonymous actions, are hard serialize. UndoPro maintains a hidden GameObject in the current scene which is deleted upon scene change which holds all custom records that needs to be serialized. The serialization is achieved by a few wrapper classes that intelligently handle every case of combination.

This system can even be used generally!
-> Supports all serializable objects (of both UnityEngine.Object and System.Object) and unserializable objects partially (one layer serializable member serialization), all other objects get defaulted
-> Supports even most anonymous actions :eek: (no unserializable found yet)! You can fully use the context and reference nearly all local variables (conditions outlined above apply)!

##Problems
This system does seem reliable but I do not claim it is completely bullet-proof!
In case it messes up the tracking, your records are offset by a small amount (actions are executed one or two records after/before). There is currently no such situation known fortunately but I take no liability for any failures!
If you manage to break it please notify me about it and provide me with information of what you've done (through issues) and I try to fix this:)

##API
The API for developers is very simple.
Add the UndoPro dependency and then use UndoProManager to interact with the system. Important elements:
- RecordOpterationAndPerform - Accepts the actions for undo/redo along with a label and adds them as a record. Additionally it will perform the redo action.
- RecordOperation (2 overloads) - Accepts the actions for undo/redo along with a label and adds them as a record. Use if you already manually performed the action
- OnUndoPerformed Callback - called when undo was performed. Passes all undone records, which can be multiple if they were grouped.
- OnRedoPerformed Callback - called when redo was performed. Passes all redone records, which can be multiple if they were grouped.
- OnAddUndoRecord Callback - called when a record was added, both from UndoPro and the default system. Passes it's name and whether it is considered 'important' (Some records clear the redo stack when added, some, like the Selection Change, do not). This could also relate to whether the record added is included in the last group of record or if it opened a new group (uncertain).
- EnableUndoPro/DisableUndoPro - toggle the state of the system. Not recommended to disable as it will simply create a hole in the record and mess up previous records!

Recommendation: If you calculated something or performed any operation with an intermediate result, it is very easy to set this up provided the result is serializable. Instead of recalculating in the undo/redo actions just set the previous serializable result in the undo action and the new one in the redo action. Done!

##Author and License
This extension was created by [Seneral](https://community.unity.com/t5/user/viewprofilepage/user-id/615187) and is published under the GNU license (further specified in LICENSE.md)
