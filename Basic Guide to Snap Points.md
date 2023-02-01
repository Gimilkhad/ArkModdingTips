Basic Explanation of Structure Snap Points
--------------------------------

My goal here is to provide a basic understanding of snap points and how the snap system works to new Structure Modders. Other modders in the Ark Modding Discord might be able to expand on this document, to explain some of the other snap settings, other tips, etc. This is meant as a starting point to get your head wrapped around the basics.

I learned by trial and error, and piecing together info from various locations, plumbing the depths of old Discord comments and forum posts. I wanted to gather it all together into one place.

I recommend looking at a structure while reading this and finding each thing I'm talking about. Perhaps later I will add pictures to the guide.

--------------------------------

Terminology:
--------------------------------
FROM snap - any snap point marked as "Attach from Point" in the snap point settings

TO snap - any snap point marked as "Attach to Point" in the snap point settings

Preview structure - the green ghost preview of the structure in your character's hands

Placed structure - any existing structure you can attempt to snap a preview structure to

Bitmask - A type of number matching that compares the individual bits that make up an integer value. This is probably a bad explanation from a pure programming point of view (I have no formal programming education), but for understanding the snap system, this should be a fairly accurate description of what is going on when we get to the Flag and Match Group stuff further down.

--------------------------------

Some things to understand (that personally took me a while to get):
--------------------------------
1. TO snaps are only used by your structure if it's an already placed structure in the world, and FROM snaps are only used by your structure if it's a preview in your character's hands. If a snap point is marked as both Attach from Point and Attach to Point, then that structure will use that snap point in either scenario.

2. Marking a snap point as both TO and FROM is handy if you need a TO and FROM snap with the exact same location and rules. It can be confusing, because you essentially have 2 snap points in one. Some settings in a snap point entry only apply to placed structures, and some only apply to preview structures.

   For example, when dealing with inclusions and exclusions, pay attention to whether they are TO inclusions/exclusions or FROM inclusions/exclusions. The TO inclusions/exclusions will only apply to snaps marked as TO snaps, and FROM inclusions/exclusions will only apply to snaps marked as FROM snaps. To put it another way, TO inclusions/exclusions will only be checked by placed structures, and FROM inclusions/exclusions will only be checked by preview structures.

3. Snap Type Flags
- Each structure has a "Snap Type Flag" value, and every snap point has a "To Point Snap Type Flags" value. These two values get compared to each other during the snapping process, which is explained further down.

4. Snap Point Inclusions and Exclusions
- There are class based and tag based Inclusions/Exclusions (the class based ones have names like "Snap to Structure Types to Exclude" and take an actual class reference, while the tag based ones rely on checking a structure's "Structure Tag").
- Exclusions work about the way you'd expect. Exclude a class or tag, and its excluded. A class's children are also excluded.
- Inclusions however mean "Include ONLY this class/tag". They do not mean "Also Include this". If you add an Inclusion, you are automatically excluding everything else.
- You can mix and match, using both Class and Tag inclusions/exclusions, but Orionsun (Creator of the mod S+) has stated in the past that there are some issues you can run into if you do it wrong. Loosely quoting: "You can include a parent class & exclude a child class using the class arrays but you can't include a parent class & exclude a child tag. Once the structure's parent class has passed the class array include, it doesn't check the tags." He noted that it might be the other way around, since he hadn't looked at it in some time. Just be aware that mixing and matching classes/tags may require some extra testing.

5. Structure Inclusions and Exclusions

   There are 2 sets of Inclusion and Exclusion settings that are OUTSIDE of the individual snap point settings. They are defaults on the structure BP itself, and they can trip you up if they're in use and you aren't aware of them. When looking at S+ source for example, internal pipes and wires used some of these settings, and they drove me mad when I was trying to learn how internal parts snapped to things because I wasn't aware of their existence.

   They're called:
    - "Only Allow Structure Classes to/from Attach" (class based only) and 
    - "Snap to/from Structure Types/Tags to Exclude" (there are class and tag versions).

   These supersede all other snap settings. So if you set up a pillar to EXCLUDE the base foundation class using  "Snap from Structure Types to Exclude", then that pillar will NEVER snap to any child of the base foundation, ever. If you set up the pillar to INCLUDE the base foundation using "Snap from Structure Types to Include", that pillar will only ever be allowed to snap to children of the base foundation and nothing else (it will also still need to pass the usual snap system checks)

6. The Snap Type Flag and Snap Point Match Group are completely separate mechanics that have no bearing on each other whatsoever. A Snap Type Flag will never be compared to a Snap Point Match Group or vice-versa. Flags get compared to Flags, and Match Groups get compared to Match Groups. They both use Bitmasking, which means the number value you put there does not have to find an exact match, but rather a matching bit.

    Snap Type Flags are for checking to see if two structures can snap to each other at all. Snap Point Match Groups are for comparing snap points to other snap points, to see which ones can snap to each other.

7. Bitmasking

   - Snap Type Flags and Snap Point Match Groups can be any value between 2 and 536870912
   - The individual bits that make up each value in binary are compared at each position

   For example, a value of 88 will match with 8, 16, and 64. If we convert these values to binary we can see why.

   |Decimal|Binary|
   |---|--------|
   |88 |01011000|
   |8  |00001000|
   |16 |00010000|
   |64 |01000000|

   As you can see, 88 has three bits flipped on (the three 1 values). From the right, it's the 4th, 5th, and 7th positions.
8 Has an ON bit at the 4th position too, so that's a match, even though 88 is not equal to 8. Only the value of the bit and its position matters. Only one bit has to match. Same for 16 and 64.

   If you are trying to make your snaps work with a vanilla structure, then your Flag and Match Group values need to be compatible with the values on the vanilla parts. If you have structures that never need to snap to anything outside of your mod, and you've fully wrapped your head around bitmasking and know what you're doing, then in theory you can use any values you like to form your Flag matches and Match Group matches. 
   
   Remember, there is a limit to the bitmasks. They start at 000000000000000000000000000010 (2) and go up to 100000000000000000000000000000 (536870912). I'm not sure what actually happens if you type in something like 9999999999999999999999. I'm assuming it just won't work, or do something unintended.

--------------------------------

How the snap point matching system works (as far as I've observed/discerned from testing and researching):
--------------------------------

When you equip a structure to place, the snap system immediately starts running its logic on tick, client-side. Then, when the preview structure gets within range of other structures, various checks start to occur. I won't pretend to know the exact order of every check, but I imagine it's like this (in descending order of priority):

- Tribe/Ownership

- Structure inclusions/exclusions (Only Allow Structure Classes to/from Attach, and Snap to/from Structure Types/Tags to Exclude)

- Snap Type Flags - the preview structure will check each of its snap points, looking at the "To Point Snap Type Flags" values in those snap points, to see if any of them match the "Snap Type Flag" value of any nearby structures (remember it's comparing bits). If no matches are found with a placed structure, you won't see any blue DebugStructure spheres on that placed structure, and your preview structure will be unable to snap to it.

- Snap Point Match Group - if the above checks were passed, then the TO snap points on the placed structure get compared to the FROM snap points on the preview structure. If a TO snap and a FROM snap have a matching Snap Point Match Group value (remember it's comparing bits), then it proceeds to the final step. If no matches are found, I'm pretty sure the blue DebugStructure spheres don't show up (would need to test that to remind myself).

- Snap Point Inclusions and Exclusions

   Inclusions
   - If a TO snap includes a class or tag, then the *preview structure* must be a child of that class or have that tag. 
   - If a FROM snap includes a class or tag, then the *placed structure* has to be a child of that class or have that tag. This is true even though the Match Group is the same.

   Exclusions
   - If a TO snap excludes a class or tag, then the *preview structure* can't be a child of that class or have that tag.
   - If a FROM snap excludes a class or tag, then the *placed structure* can't be a child of that class or have that tag. Again, this is true even though the Match Group is the same.

   I'm pretty sure the blue DebugStructure spheres will show up during this stage, even if the inclusions/exclusions are preventing a snap from happening.

 If all the above criteria are met, then in theory the preview structure can now snap to the placed structure using any of the valid snap points that were found (based on stuff like player camera angle and cycling with Q). 
 
 At this time the Point Location Offset, Rotation, etc values will dictate the position and orientation of the preview structure.
 
 This is also where the "Allowed To Build" rules kick in, turning the preview structure Red or Green and displaying messages to the player to indicate if the structure can actually be placed there.

--------------------------------
Troubleshooting:
--------------------------------

If your structure isn't snapping to another structure:
- Make sure the placed structure has collision. No collision means the snap system won't "see" the structure, and you probably won't see the blue snap point spheres if you have DebugStructures turned on.
- Make sure you don't have some inclusions or exclusions hiding in the structure defaults. These would be the "Only Allow Structure Classes to/from Attach" and "Snap to/from Structure Types/Tags to Exclude" variables I mentioned previously.

If you aren't seeing any blue snap point spheres or some are missing when DebugStructures is enabled, it could also mean:
- Your Snap Type Flags are misconfigured
- Your Snap Point Match Groups are misconfigured
- Your actor origins are trying to place too close together
- It might also possibly be the STRUCTURE inclusions/exclusions, but that's something I'd have to re-test

--------------------------------

The Actors-too-Close Problem

If your snap points are going to cause your preview structure's actor origin to be within 10 units of the placed structure's actor origin, the blue snap point sphere of that particular snap on the placed structure will disappear and the snap will not work at all, causing much confusion and gnashing of teeth.

Another way you can cause this problem is if the rotation offsets on the TO and FROM snaps are set wrong and are rotating the preview into the placed structure, potentially placing the actors too close together. In that case you can just fix the rotation values and be good, since the snap is totally wrong and isn't supposed to be putting the preview there to begin with.

The purple sphere in Debug mode represents the structure actor's origin. It doesn't always show up, and I'm not sure why (or I've forgotten). It's probably a setting in the Placement section of structures. You can figure out where your actor origins are either by drawing the spheres yourself, or just looking in the component tab to see how you've adjusted your mesh's positioning.

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

I chose to do Option B with my Arkitect Structure mods, but I also did a decent amount of graphing to automatically (and very carefully) fix the locations of previously placed ceilings (remember ceilings can be on saddle platforms too, adding extra complexity). I chose this route because I was tired of encountering the actor origin issue, which I feel is a common problem and design flaw of the base ceiling setup, and decided I wanted to change mine.

--------------------------------

Other notes:
--------------------------------
IsValidSnapTo and IsValidSnapFrom structure functions
- These are functions you can implement in your structures, that allow you to graph your own extra rules for whether one snap point can snap to another snap point, or perform other tricks.
- IsValidSnapTo runs on client, on tick, in every nearby placed structure.
- IsValidSnapFrom works just like IsValidSnapTo except it runs on the preview structure. It can be tricky to use depending on what you're doing. If you are setting any variables on the preview structure during placement, those values are lost when placement occurs because the preview structure is destroyed and a new copy is created on the server.
- Both functions can run multiple times in the server-side of the structures during a structure placement event.

IsAllowedToBuild and IsAllowedToBuildEx
- These are functions you can implement in your structures, that allow you to graph your own extra rules for whether a structure is allowed to place at the current location. It isn't *directly* related to snapping, but it can be useful to override the built-in placement rules if need to.
- These functions also fire on tick on the client side, and at least once during placement on server (potentially multiple times on server)
- If you use the "Ex" version, you have to enable it in the structure defaults
- It seems you can use both at the same time, but I don't see why anyone would want to. It would likely cause problems to use both, especially if you're doing different things in each.

There are plenty of other snap settings I didn't cover but these are the basics. I still have no clue how some of the other settings inside snap points work, and sometimes they seem to make no difference how I set them when playing with them.

There are also a lot of Placement settings on the structure not covered yet in this guide. I'm referring to all the settings under the "Placement" section in the structure defaults. Stuff like snap range (how far a preview structure can be from other structures before the snap logic starts running), stuff related to placing on the ground, deciding if a structure is a foundation, etc etc. I won't cover all that here, now, because frankly some of it is still a mess in my head and it comes down to just playing with settings until I get what I want. I don't mess with those settings nearly as much as the snap points, since most of the structures I've worked on are typical sized foundations, walls, etc.

--------------------------------

Resources:
--------------------------------
Something that helped me translate flags and match groups between decimal and binary values:

https://www.rapidtables.com/convert/number/decimal-to-binary.html?x=64

Documentation of most of the snap flags and match groups in the Ark building system, and further explanation on how the bitmasking works:

https://forums.unrealengine.com/t/reference-snap-point-flags/48937
