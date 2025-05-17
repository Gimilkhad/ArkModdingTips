Basic Explanation of Structure Snap Points
--------------------------------

My goal here is to provide a basic understanding of snap points and how the snap system works to new structure modders. Other modders in the Ark Modding Discord might be able to expand on this document, to explain some of the other snap settings, other tips, etc. This is meant as a starting point to get your head wrapped around the basics.

I recommend looking at a structure while reading this and finding each thing I'm talking about. Perhaps later I will add pictures to the guide.

--------------------------------

Tips:
--------------------------------
- Type DebugStructures in the console in PIE (Play in Editor) to see the blue spheres I mention in the guide. The spheres show you where a snap point is. In ASA, DebugStructures will not draw spheres for every snap point on a structure like it did in ASE. It only will only draw them within a short range and only if the snap points are usable at that moment.
- You can edit snap points while PIE is running (as well as lots of other stuff). When setting up or debugging snap points, you can place a structure, equip another structure near it, then play with snap points in the structure defaults to see the changes in real time.
- Do not use the Compile button when PIE is running. You can sometimes get away with it but a lot of the time it will crash your devkit. Make your snap point edits and when you're done, stop PIE, compile, and save.
- Snap Points can be changed at any time, even in a mod that is already live. It will not cause the loss of any existing structures, nor move them. The Snap Point system is only ever used when a player is currently placing a structure.

--------------------------------

Terminology:
--------------------------------
FROM snap - any snap point marked as "Attach from Point" in the snap point settings

TO snap - any snap point marked as "Attach to Point" in the snap point settings

Preview structure - the green ghost preview of the structure in your character's hands

Placed structure - any structure that has already been placed in the world, that you can attempt to snap a preview structure to

Bitmask - A type of number matching that compares the individual bits that make up an integer value. This is probably a bad explanation from a pure programming point of view (I have no formal programming education), but for understanding the snap system, this should be a fairly accurate description of what is going on when we get to the Flag and Match Group stuff further down.

--------------------------------

Concepts:
--------------------------------
1. TO snaps are only used by a structure if it's an already placed structure in the world, and FROM snaps are only used by a structure if it's a preview in your character's hands. If a snap point is marked as both Attach from Point and Attach to Point, then the structure will use that snap point in either scenario.

2. Marking a snap point as both TO and FROM is handy if you need a TO and FROM snap with the exact same location and rules. It can be confusing, because you essentially have 2 snap points in one. Some settings in a snap point entry only apply to placed structures, and some only apply to preview structures.

   For example, when dealing with inclusions and exclusions, pay attention to whether they are TO inclusions/exclusions or FROM inclusions/exclusions. The TO inclusions/exclusions will only apply to snaps marked as TO snaps, and FROM inclusions/exclusions will only apply to snaps marked as FROM snaps. To put it another way, TO inclusions/exclusions will only be checked by placed structures, and FROM inclusions/exclusions will only be checked by preview structures.

3. Snap Type Flags
- Each structure has a "Snap Type Flag" value, which is a special value used to represent what general type of structure part that structure is. Foundations are 2, ceilings are 4, walls are 8, and so on.
- Every snap point has a "To Point Snap Type Flags" value. During the snapping process this value gets compared to a corresponding Snap Type Flag value in snap points using "Bitmasking", which is explained further down.
- "To Point Snap Type Flags": This is an inclusion list for structures, using Snap Type Flags. If the snap is a TO snap, and you only want it to allow foundations to snap to it, set this to 2 (Foundations are Snap Type Flag 2). If you want to include foundations and ceilings, set it to 6 (2 for foundations, 4 for ceilings).
- "To Point Snap Type Exclude Flags": Similar to above, except it EXCLUDES Snap Type Flags instead of include. Putting a 2 here would exclude foundations from snapping to this snap point but allow almost everything else.
- If you are familiar with Including and Excluding structures using Structure Tags or classes, this is basically the same thing, you're just using bitmasks instead. It's confusing that they have the word "To" in them, as it implies it would only work on TO snaps as I believed for a very long time. They work on FROM snaps as well.

4. Extra Structure Snap Type Flags
- In addition to the main Snap Type Flag, ASA also added 3 "Extra" snap type flags, stored as a vector. X, Y, and Z can all have a number value which will further "describe" what a structure is to the game's structure system.
- For example the main Snap Type Flag of a square ceiling is 4, and if you look at the X value of the Extra Snap Type Flags, it says 2. The 4 tells the game it's a ceiling, and the 2 tells the game it's a square ceiling. If you look at a Triangle ceiling you'll see that the values are 4 and 4. Snap Type Flag 4 means it's a ceiling, and Extra Structure Snap Type Flag X=4 means it's a triangle ceiling.
- These numbers could theoretically be anything, but unless you want to change your entire snap setup and break compatibility with vanilla structures, you generally don't want to change any of these values.
- Y and Z are for getting even more specific. Wildcard doesn't use Z yet, and I can't think of any actual examples of them using Y. To avoid confusion I won't make up any examples. Just know that the main Snap Type Flag is the broadest description and X, Y, Z are used to get more and more specific.
- X is compared with X, Y is compared with Y, Z is compared with Z.
- "Extra Structure Snap Type Flags" on the structure correspond with "Extra Snap Type Flags" in snap points. They're slightly different names, but don't allow that to confuse you. They go together.

5. Snap Point Inclusions and Exclusions
- There are two kinds of Inclusions, and three kinds of Exclusions, found in snap points.
- You can Exclude structures from snapping using the To Point Snap Type Exclude Flags (or the Extra Snap Type Exclude Flags). These only work in TO snaps, meaning only placed structures will check this.
- You can also Exclude structures from snapping using class based and tag based Inclusions/Exclusions (the class based ones have names like "Snap to Structure Types to Exclude" and take an actual class reference, while the tag based ones rely on checking a structure's "Structure Tag").
- Inclusions can only be done via Class or Tag.
- Exclusions work about the way you'd expect. Exclude a class or tag, and its excluded. A class's children are also excluded.
- Inclusions however mean "Include ONLY this class/tag". They do not mean "Also Include this". If you add an Inclusion, you are automatically excluding everything else.
- You can mix and match, using both Class and Tag inclusions/exclusions, but Orionsun (Creator of the mod S+) has stated in the past that there are some issues you can run into if you do it wrong. Loosely quoting: "You can include a parent class & exclude a child class using the class arrays but you can't include a parent class & exclude a child tag. Once the structure's parent class has passed the class array include, it doesn't check the tags." He noted that it might be the other way around, since he hadn't looked at it in some time. Just be aware that mixing and matching classes/tags may require some extra testing.

6. Structure Inclusions and Exclusions

   There are 2 sets of Inclusion and Exclusion settings that are OUTSIDE of the individual snap point settings. They are defaults on the structure BP itself, and they can trip you up if they're in use and you aren't aware of them.

   They're called:
    - "Only Allow Structure Classes to/from Attach" (class based only) and 
    - "Snap to/from Structure Types/Tags to Exclude" (there are class and tag versions).

   These supersede all other snap settings. So if you set up a pillar to EXCLUDE the base foundation class using  "Snap from Structure Types to Exclude", then that pillar will NEVER snap to any child of the base foundation, ever. If you set up the pillar to INCLUDE the base foundation using "Snap from Structure Types to Include", that pillar will only ever be allowed to snap to children of the base foundation and nothing else (it will also still need to pass the usual snap system checks)

7. Snap Point Match Groups
- These values have nothing to do with Snap Type Flags. Snap Type Flags are for checking to see if two structures can snap to each other at all. Snap Point Match Groups are for comparing snap points to other snap points, to see which ones can snap together.
- A Snap Type Flag will never be compared to a Snap Point Match Group or vice-versa. Flags get compared to Flags, and Match Groups get compared to Match Groups. They both use Bitmasking, which means the number value you put there does not have to find an exact match, but rather a matching bit.

8. Extra Snap Point Match Group
- New to ASA, similar to the Extra Snap Type Flags. They are a way to further describe a snap point and what other snap points it can snap to. You can look at foundations or ceilings to see them in use. While the main Snap Point Match Group describes what side of the floor it's on, X is used to describe if the snap is for middle, left, or right.

9. Bitmasking

   - Snap Type Flags and Snap Point Match Groups can be any value between 2 and 536870912
   - Bitmasking looks at the values in binary, and compares the individual bits at each position

   For example, a value of 88 will match with 8, 16, and 64. If we convert these values to binary we can see why.

   |Decimal|Binary|
   |:-:|--------|
   |88 |01011000|
   |||
   |8  |00001000|
   |16 |00010000|
   |64 |01000000|

   As you can see, 88 has three bits flipped on (the three 1 values). From the right, it's the 4th, 5th, and 7th positions.
8 Has an ON bit at the 4th position too, so that's a match, even though 88 is not equal to 8. Only the value of the bit and its position matters. Only one bit has to match. 16 matches with 88 at the 5th position, and 64 matches with 88 at the 7th position.

   If you are trying to make your snaps work with a vanilla structure, then your Flag and Match Group values need to be compatible with the values on the vanilla parts. If you have structures that never need to snap to anything outside of your mod, and you've fully wrapped your head around bitmasking and know what you're doing, then in theory you can use any values you like to form your Flag matches and Match Group matches. Not that anyone in their right mind should do that.
   
   Remember, there is a limit to the bitmasks. They start at 000000000000000000000000000010 (2) and go up to 100000000000000000000000000000 (536870912). I'm not sure what actually happens if you type in something like 9999999999999999999999. I'm assuming it just won't work, or do something unintended.
  
10. Snap Points and Structure Linking are two different things. 
   - Snap Points determine where the preview structure is going to go on a placed structure. That's basically it. Once a structure is placed it doesn't care about snap points anymore unless a player is trying to snap something to it.
   - Structure Linking is what actually "glues" placed structures together and determines things like when a structure is supported or not, or when two pipes/wires are connected or not. I know that at least some Snap Point settings are checked during the Structure Linking process (like "Invalid for Structure Linking"), but other than that I don't know how the game decides what structures get linked to what. That code is unavailable to modders. My best guess is the linking process does little server-side overlap checks around the snap point that was used during placement, and links to whatever structures it found (with exceptions).

11. Allow Snap Rotation (and Point Rotation Offset)
   - Allow Snap Rotation: if checked, will make it so this structure is always allowed to be rotated by the rotation values of snap points. When snapping to any structure. All the time. This setting must be configured in the placing structure for it to do anything.
   - Allow Snap Rotation to Structures with Tag: allows you to enable rotation in more controlled, specific snap situations instead of "all the time". This setting must be configured in the placing structure for it to do anything.
   - Force Allow Snap Rotation to Structures with Tag: allows you to enable rotation in more controlled, specific snap situations instead of "all the time". This setting must be configured in the placed structure for it to do anything. This setting comes in handy if you had to change your Structure Tag for some reason, but still need snap rotation to work with existing parts from vanilla or other mods. You can add the tags for the vanilla parts to this "force array" and it will force the vanilla parts to rotate when snapping to your structure, even if they don't have the tag in their "Allow" array.
   - Point Rot Offset: a setting inside snap points. Affects both FROM and TO snaps, but only if the placing structure has Allow Snap Rotation set up correctly.
   - Example: On a square ceiling, "Allow Snap Rotation to Structures with Tag" is set include the "TriangleCeiling" structure tag. When a square ceiling is in placement mode, snapping to a triangle ceiling, the rotation value of the FROM snap on the square ceiling will rotate the square ceiling, pivoting the square ceiling *around the point where the FROM snap meets the TO snap of the Triangle ceiling*. At the same time, the rotation value of the TO snap on the triangle ceiling will ALSO rotate the square ceiling. This means you have to add the rotation values of the FROM and TO snaps to know how much the square ceiling will actually be rotated.

12. Structure Tag and Secondary Structure Tags
    - Structure Tag is a label you give to a set of structure parts of the same shape (for example Floor_Triangle, or Wall_Sloped_Left). The Structure Tag is checked by at least 3 systems in the game, maybe more: Snap point Inclusions/Exclusions, Allow Snap Rotation for Tags (and the "Force" version), and the new structure skin system (that's how Frontier knows what model to use when you place it on a part. It checks the Structure Tag).
    - Secondary Structure Tags, as far as I know, are only checked by the Snap Point Inclusion/Exclusion arrays. So if for some reason you need a structure to have multiple Structure Tags for the purposes of snap point inclusion/exclusion, you can put the extra tags in this array.

--------------------------------

How the snap point matching system works (as far as I can tell):
--------------------------------

When you equip a structure to place, the snap system immediately starts running its logic on tick, client-side. Then, when the preview structure gets within range of other structures, various checks start to occur. I won't pretend to know the exact order of every check, but I imagine it's like this (in descending order of priority):

- Tribe/Ownership

- Structure inclusions/exclusions (Only Allow Structure Classes to/from Attach, and Snap to/from Structure Types/Tags to Exclude)

- Snap Type Flags - The Snap Type Flag of the preview structure will be compared to the "To Point Snap Type Flags" value in the TO snap points of every nearby placed structure, to see if if there is a match (remember it's comparing bits). If no matches are found within a placed structure, you won't see any blue DebugStructure spheres on that placed structure, and your preview structure will be unable to snap to it.

- Extra Snap Type Flags - New to ASA. The X value of the FROM snaps in the preview structure will be compared to the X value found in the TO snaps of every nearby placed structure. The Y values will be compared, and the Z values will be compared. From what I can tell, ALL snap type flags have to find a match. If the value is Zero, it means it will always match. So even if one structure has an Extra Snap Type Flag X value of 4, but the other structure has an X value of zero, it will still match.

- Snap Point Match Group - if the above checks were passed, then the TO snap points on the placed structure get compared to the FROM snap points on the preview structure. If a TO snap and a FROM snap have a matching Snap Point Match Group value (remember it's comparing bits), then it proceeds to the final step. If no matches are found, I'm pretty sure the blue DebugStructure spheres don't show up (would need to test that to remind myself).

- Extra Snap Point Match Group - same as above but runs through the X, Y, and Z values, comparing X to X, Y to Y, Z to Z. Again, each one has to match, and Zero means it will always match.

- Snap Point Inclusions and Exclusions

   Inclusions
   - If a TO snap includes a class or structure tag, then the *preview structure* must be a child of that class or have that tag. 
   - If a FROM snap includes a class or structure tag, then the *placed structure* has to be a child of that class or have that tag. This is true even though the Match Group is the same.

   Exclusions
   - If a TO snap excludes a Snap Type Flag, then the *preview structure* can't be using that Snap Type Flag. Pretty sure if even one bit is a match during the bitmask process, it will be excluded.
   - If a TO snap excludes a class or structure tag, then the *preview structure* can't be a child of that class or have that tag.
   - If a FROM snap excludes a class or structure tag, then the *placed structure* can't be a child of that class or have that tag. Again, this is true even though the Match Group is the same.

   If you're testing in the devkit and using DebugStructures, I'm pretty sure the blue DebugStructure spheres will show up during this stage, even if the inclusions/exclusions are preventing a snap from happening.

 If all the above criteria are met, then in theory the preview structure can now snap to the placed structure using any of the valid snap points that were found (based on stuff like player camera angle and cycling with Q). 
 
 At this time the Point Location Offset, Rotation, etc values will dictate the position and orientation of the preview structure.
 
 During the placement process, the "Allowed To Build" function runs on tick on client, which is part of the logic turns the preview structure Red or Green and displays messages to the player to indicate if the structure can actually be placed there. That function will also run once on server when the structure attempts to place. As far as I know it does not by default influence snap rules. It's more related to placement rules, like preventing placement inside terrain or other structures (Obstructed), too close to an enemy base, too high above ground, etc.

--------------------------------
Troubleshooting:
--------------------------------
First of all, the DebugStructures console command is a must when troubleshooting snap points. You're blind without it. Turn it on any time you're troubleshooting snap points.

Second, when DebugStructures is enabled, you'll notice that a working snap point is indicated by the preview structure moving to the correct position, and a lighter blue ball appears where the snap was made to indicate success (it also displays the names and indexes of the two snap points that made a successful connection).

If you see the lighter blue ball appear, but your structure doesn't move to a new position:
- It's possible you need to enable "Snap Force No Ground Requirement" in the TO snap point on the placed structure. 
- I encountered this in the old ASE devkit with my square roof caps and pillars. I wanted to make pillars snap downward from each corner of the roof cap, and I was seeing the light blue ball, but the pillar was not actually moving to position. The roof cap was too high above the ground, and the pillar apparently required touching the ground before it would snap. I went into the roof cap, to the four TO snaps I set up for pillars, and enabled that option in each to force no ground requirement. This allowed the pillars to snap even though they were too high to touch the ground.

If your structure isn't snapping to another structure:
- Make sure the placed structure has collision. No collision means the snap system won't "see" the structure, and you probably won't see the blue snap point spheres if you have DebugStructures turned on.
- Make sure you don't have some inclusions or exclusions hiding in the structure defaults. These would be the "Only Allow Structure Classes to/from Attach" and "Snap to/from Structure Types/Tags to Exclude" variables I mentioned previously.

If you aren't seeing any blue snap point spheres or some are missing when DebugStructures is enabled, it could also mean:
- Your Snap Type Flags are misconfigured
- Your Snap Point Match Groups are misconfigured
- Your actor origins are trying to place too close together
- It might also possibly be the STRUCTURE inclusions/exclusions, but that's something I'd have to re-test to confirm

--------------------------------

The Actors-too-Close Problem

If your snap points are going to cause your preview structure's actor origin to be within 10 units of the placed structure's actor origin, the blue snap point sphere of that particular snap on the placed structure will disappear and the snap will not work at all, causing much confusion and gnashing of teeth. In other cases, the preview structure might snap to that position but refuse to actually place. I haven't quite figured out the rules.

Another way you can cause this problem is if the rotation offsets on the TO and FROM snaps are set wrong and are rotating the preview into the placed structure, potentially placing the actors too close together. In that case you can just fix the rotation values and be good, since the snap is totally wrong and isn't supposed to be putting the preview there to begin with.

The purple sphere in Debug mode represents the structure actor's origin. It doesn't always show up, and I'm not sure why (I'm not sure I've seen it at all in ASA now that I think of it). It's probably a setting in the Placement section of structures. You can figure out where your actor origins are either by drawing a debugsphere yourself via graphing, or just looking in the component setup to see how you've adjusted your mesh's positioning. If you have the mesh raised up 20 units in the components tab, then you have to imagine the actor origin of the structure is 20 units below the mesh.

An easy way to test if you have the problem is to just go into your FROM snap and lower it more than 10 units (20 just to make sure you get a good test result). Which will raise the position of the structure when snapping). So if it's point location Z offset is 0, set it to -20, or if it's 40, set it to 20, and so on. If the snap works after that it means the actor origins are too close together when the snaps are at their normal values.

--------------------------------

How to fix Actors-too-Close:
- The only way I know of to fix the actors-too-close issue is to change the setup of one of the structures (but this can be tricky to do if your mod is already live and people have built these structures).

Let's use a crop plot as an example. You made a crop plot and you want it to snap straight to the top of a square ceiling you made. Currently they don't snap because their actor origins would be within 10 units of each other.

A. Adjust the crop plot
- Lower the mesh of the crop plot in the components tab at least 10 units, then go to every snap point and lower its Point Location offset by 10 as well.
- The crop plot will now snap because the FROM snaps are 10 units lower, which raises the crop plot 10 units above the ceiling, so now the actor origins are 10 units apart. The crop plot will look correct because you also lowered the mesh in the components tab to compensate for the change you made in the snap point.
- Probably the better option of the two.
 
  Downsides: 
  - This will fix the issue only with the crop plot. Other structures you make later that need to snap the same way will potentially require similar adjustments.
  - If this structure has already been built on server, the placed ones will appear to be 10 units lower than before, and people will have to pick them up and re-place them to fix it.

B. Adjust the ceiling
- Raise the mesh of the ceiling 10 units, then go through and adjust ALL the snap points. Raise the TO snaps AND the FROM snaps by 10. This effectively makes the ceiling have an actor origin 10 units lower than before.

  Downsides:
  - Ceilings from other mods might not replace yours, and overlap instead. This is because the replacement logic that blows up the previous structure actually relies on checking if the actor origins are within 10 units. So by fixing the snap issue this way, you partially break that feature. Your ceilings will replace your ceilings, because they have the same modification.
  - If people have built these ceilings, they're all going to appear to shift 10 units up from where they were before. People might not like having to re-place all their ceilings.

I chose to do Option B back in my ASE Arkitect Structure mods, but I also did a decent amount of graphing to automatically (and very carefully) fix the locations of previously placed ceilings (remember ceilings can be on saddle platforms too, adding extra complexity). I chose this route because I was tired of encountering the actor origin issue, which I kept running into in classic Ark with some of the things I was trying to do. It has been less of a problem in ASA, but I did encounter it again recently while working on my roof cap parts, specifically getting flat pillars to snap to their top edge. I don't want to go through the headache of "Option B" again, or deal with its downsides, so I'll probably end up making special pillars just for that situation or something. If someone has found a way to just "turn it off" so a structure can always place no matter where the actor origins are, I'd sure love to know how.

--------------------------------

Other notes:
--------------------------------
Snap Overlap Check Radius and Max Snap Loc Range
- If you create a structure that is larger than normal, you may find yourself needing to adjust these settings for snapping to work.
- Snap Overlap Check Radius controls the range a placing/preview structure will check for a snap target, aka a nearby placed structure to snap to.
- Once a snap target is found, Max Snap Loc Range controls the range that the preview structure will check for snap points. Exactly how it does the check is a mystery to me still. In testing and playing with these two range values I've never quite made sense of this one.
- In general, you'll want both of these range settings to match. I haven't come across a scenario where I've needed to use different values.

IsValidSnapTo and IsValidSnapFrom structure functions
- These are functions you can implement in your structures, that allow you to graph your own extra rules for whether one snap point can snap to another snap point, or perform other tricks.
- IsValidSnapTo runs on client, on tick, in every nearby placed structure.
- IsValidSnapFrom works just like IsValidSnapTo except it runs on the preview structure. In ASE it was tricky to use depending on what you were doing. If you were setting any variables on the preview structure during placement, those values were lost when placement occured because the preview structure was destroyed and a new copy was created on the server, with none of your edits. However in ASA, Wildcard has added the ability to store data in a special variable which will replicate to the server structure. So IsValidSnapFrom might be more useful now.
- Both functions will run at least once on server when the structure is placed. In ASE sometimes they ran multiple times. Not sure about ASA. I haven't checked.

IsAllowedToBuild and IsAllowedToBuildEx
- These are functions you can implement in your structures, that allow you to graph your own extra rules for whether a structure is allowed to place at the current location. It isn't *directly* related to snapping, but it can be useful to override the built-in placement rules if need to.
- These functions also fire on tick on the client side, and at least once during placement on server (potentially multiple times on server)
- If you use the "Ex" version, you have to enable it in the structure defaults
- It seems you can use both at the same time, but I don't see why anyone would want to. It would likely cause problems to use both, especially if you're doing different things in each.

There are plenty of other snap settings I didn't cover but these are the basics. I still have no clue how some of the other settings inside snap points work, and sometimes they seem to make no difference how I set them when playing with them. For example "Porthole" and "Snap to Only Allow Single Attachment". I've never figured those two out. I've no idea what "Porthole" is supposed to mean and the other one has never appeared to work at all for me, or I misunderstand what it does.

There are also a lot of Placement settings on the structure not covered yet in this guide. I'm referring to a lot of the settings under the "Placement" section in the structure defaults. Stuff related to placing on the ground, deciding if a structure is a foundation, etc etc. With settings like those it comes down to just playing with them until I get what I want. I don't mess with those settings nearly as much as the snap points, since most of the structures I've worked on are typical sized foundations, walls, etc.

--------------------------------

Resources:
--------------------------------
Something that helped me translate flags and match groups between decimal and binary values:

https://www.rapidtables.com/convert/number/decimal-to-binary.html?x=64

Documentation of most of the snap flags and match groups in Ark Survival Ascended, courtesy of Orionsun:

https://docs.google.com/document/d/e/2PACX-1vTa2FzbYhjtDwJoDbIjTd1bMNUOvhj9zn38GyGTV7pVVMq7sWHRJAkP84oGSk8-Q18iRTXz8KwoStEt/pub

https://docs.google.com/document/d/e/2PACX-1vRmUGRfb3knh8s0OB6tRElZO_xvGu8ykqF4hPZTecULJWdnhhSwWSQqDKFtLJsVcYTf_LqooX3HDRCT/pub
