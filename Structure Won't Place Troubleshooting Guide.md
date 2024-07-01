While this troubleshooting list is primarily aimed at ASA, some of it will still apply to ASE.

Symptoms: 
- You try to equip your structure but using the item does nothing. No structure appears.
- You have multiple structures on an item, but only some show up.
- Your custom structure is in preview/placement mode, it’s green, but it won’t place when you click to place it.

1. Check additional structures to place in ModData asset

As of 4-23-24, the ModDataAsset bug reported by Quellcrest is still there. Meaning if you just added new structures to the Additional Structures to Place array you’ll need to start PIE once without your mod’s PGD loaded (or restart your devkit) to refresh the ModData. Otherwise when you start PIE it will use an old cached copy of your ModData.

2. Check item to consume on the structure

3. Check structure to build on the item

4. If you were editing snaps with PIE running, restart PIE. Especially if you were compiling structures while it was running but even if you weren’t. Sometimes that’s all it is.

5. If it’s a structure that has alternative “override” structures, double check that your FROM snaps are good not only on the main structure but the alternative parts as well. Examples would be vanilla rails, sloped walls, triangle roofs.

Note: From some of the testing I’ve done recently I now believe the snap array on the alternate structure has to match up with the primary structure. Even inserting one snap point at the beginning on the alternate structure seems to be enough to break its ability to place.

Note2: Additional testing and recent issues led me to discover that not only do the snap arrays have to be in sync, their point location offsets have to be the same. It's entirely possible other settings have to match, such as Point Rot Offset, Point Scale Offset. I haven't tested those. I do know that the Attach To and Attach From bools can be different values.

6. Check Placement Encroachment settings. Sometimes even if the encroachment boxes are green in debugstructures mode, the structure won’t place. I had this issue with my “short” doors, and I had to shrink the vertical extent of the encroachment boxes before the doors would actually place, even though the boxes were all green.

7. Snap Range settings (MaxSnapLocRange and SnapOverlapCheckRadius). Sometimes even if the structure is snapping where you want and the preview mesh is green, you have to fiddle with one of these settings (increase range). Try one then the other. I forget which one does what.

8. Sometimes you need to check both Placement Encroachment and the Snap Range settings simultaneously. My steep roof caps did this to me recently. I had to increase one of the snap range settings AND reduce the placement encroachment size before they’d place. Changing only one or the other did not help.

9. Regarding the problem with a structure item only showing one or two structures you've added to the item: open the primal item and strict compile it with PIE closed. You may also need to “dirty” the file by toggling a random default. This issue seems to happen reliably if you try to edit the Structures to Build array from property matrix.

10. Check the collision mesh component on the structure you're trying to snap the placing structure to. If it's messed up or positioned wrong it can cause issues.
