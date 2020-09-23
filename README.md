# MSFS Toolbar Handle Removal Mod

This repository is meant to serve two purposes:  as the source for a mod to remove the annoying "handle" for the toolbar that appears in Microsoft Flight Simulator when the mouse moves while you're flying; and as a bit of a tutorial for the MSFS modding community on how to create a mod that stands the best chance of not conflicting with others or breaking on future updates.

If you are just looking for the mod, go to the [releases page]() to download it and be happy.

But if you want to know a bit more about why I created this, and maybe learn a bit about MSFS or software engineering in general, read on for a discussion of the original mod and what I've done to it.

## The Problem

I discovered the existing [NoToolbarHandle mod](https://www.reddit.com/r/MicrosoftFlightSim/comments/iqniyx/mod_to_remove_the_toolbar_handle_whenever_you/) on reddit thanks to a comment in a [thread at Avsim](https://www.avsim.com/forums/topic/585331-how-do-you-guys-find-add-ons-i-m-loosing-track-haha/?do=findComment&comment=4354285).  The commenter there, "bobcat999", noted that the mod was made before the 1.8.3.0 update, which added an active pause button to the toolbar, and that it hadn't been updated for the new patch; further, while it still worked, using it meant that you did not get the new item on your menu bar.

This seemed like something that would be simple enough to fix, so I grabbed the original mod -- because that handle annoys me too -- and had a look at what it was doing.  It took me a couple minutes to figure out the change the mod had made because it included a number of files, and was written for an earlier version of the game.  But by digging the previous version of the `fs-base-ui-pages` package out of a backup I was able to compare the changes directly.

## A Disclaimer

Before I continue, there are two things to note.   First, I have included a copy of the original mod in this repo under the "[original](original/)" directory.  As I note in the [README there](original/README.md), this is purely for illustrative purposes, so that it is easy for the reader to follow along with this text if they wish.  I claim no ownership or rights of any sort, and if the original author ever objects I will remove it.

Second, the fact that I am writing this is in **no way** meant to criticize or disparage the original author.  It is merely to help educate the community as a whole on ways in which they can produce mods with the maximum chance of sustained compatibility -- or, at least, graceful failure modes.

## A Quick Overview of the MSFS Filesystem

I think most folks in the modding community have a basic understanding of the MSFS filesystem, but it pays to have a little more than a basic understanding, so here's a crash course.

The basic unit of content is a "package".   There are the directories that live on your `Community` folder, or the `Official` one.  Each package has at its top level two files, a `layout.json`, and a `manifest.json`, and a collection of directories under it.  If you're familiar with the FSX or P3D directory structure you may recognize some of these, especially `SimObjects`.

When the game starts it builds a virtual filesystem (VFS) by reading everything in the `Official` packages.  It then reads the packages in `Community` in alphabetical order by directory name. For every package it reads the `layout.json` to learn about the contents of the package and adds every file named in there to its VFS.   It essentially "layers" the packages on top of one another in the order it reads them.   A file in a given package later in the order will overwrite a file in the same location in an earlier package.

The main thing to note here is that is unnecessary to include "extra" files that you don't use in a mod.  You only need to package the ones you modify and add, and leave the rest alone.

(There's also some confusion in places about what the metadata files in a package should look like.  For a side note on that, have a look at [this additional document](package_metadata.md))

## The Original Mod

If you examine the original mod, its directory structure looks like this:

    ├── html_ui
    │   └── Pages
    │       └── Toolbar
    │           ├── ToolBar.css
    │           ├── ToolBar.html
    │           └── Toolbar.js
    ├── layout.json
    └── manifest.json

This is a mirror of the same directory structure as in the official `fs-base-ui-pages` package, and thus overwrites those particular files in it.

If you then move on to examine the differences between the files in the mod and the original version 1.7 files in turn, you'll notice that there is no difference in `ToolBar.html` or `Toolbar.js` between the original and the mod.   There appear to be a fair number of differences in `ToolBar.css`, but if you look more closely you'll note that most of them are format changes.  There is only one substantive difference between the two.

Where the stock file has:

    tool-bar .toolbar-handle.visible, tool-bar .toolbar-handle.active {
        transition: opacity .7s 0s;
        opacity: 1; }

The mod (removing the formatting change) has:

    tool-bar .toolbar-handle.visible, tool-bar .toolbar-handle.active {
        transition: opacity .7s 0s;
        opacity: 0; }

It just changes the opacity of those two elements.

## Shortcomings

What are the flaws with doing it in this way?  The main one is that this, effectively, completely obliterates the toolbar as it was shipped with the game and replaces it with an almost identical copy.  While this was fine at first, when the update to game version 1.8 came and brought changes to all three files in that directory in order to add support for the active pause menu item... they were completely lost for anyone who had the mod installed.

Beyond that, shipping files that you do not need to include means that you are almost guaranteed to conflict with any other mod that's changing anything in that area.  In this case, if someone had created a small mod that altered, say, the javascript used for the toolbar, and this mod came last in the load order, this mod would totally mask the other one.  An additional shortcoming is that this makes porting in upstream changes more challenging:  I had to carefully examine a three-way diff between this version, the 1.7 version, and the 1.8 version to figure out what would need to change to "port" the mod to the new game version, and this is for a simple three-file mod.

In this case, since the surface area of the change is limited, the trouble is not that great.  But, for something that touches more files, or in more commonly used areas, this could cause much harder to solve problems.

## A Better Way

So, how can we do this better?  Clearly one improvement would have been to only ship the changed CSS file.  This would reduce the area the mod covers and keep you from "masking" other changes made either by the game itself or by other modders.  That has its own drawbacks, though.   You still have the problem of having made potentially comprehensive changes to a stock file which could make updating it for underlying game changes more difficult.

(Interestingly, in this case, the way the mod was built had one advantage:  since the entire toolbar was included it managed to mask out the *actual* changes made to the official package by Asosbo and continue to provide a functional, if limited, toolbar.  This would not necessarily happen in all cases.)

I would suggest an alternate approach:  make the absolute minimum number of changes to stock files necessary to provide a "foothold" for your code, and add all remaining content in additional files unique to your mod.  In a way, this is similar the OOO concept of [encapsulation](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)), in which you compartmentalize the logic and data of a component and provide access to it only through a public interface.  Here, you're separating the logic and data of your mod from that of the stock game.

In this particular case, due to the nature of CSS, this is very easy.   I created an extremely small CSS file, [ToolBarOverride.css](mainmod/html_ui/Pages/Toolbar/ToolBarOverride.css), which contains only the content needed to effect this change:

    tool-bar .toolbar-handle.visible, 
    tool-bar .toolbar-handle.active {
        opacity: 0; 
    }

Note that I don't even rewrite the full content of that block from the original CSS, I just change this one parameter.  Since CSS cascades this is very doable.  For other kind of code it's likely to not be quite so simple...  but that's even more reason to do it this way here, where it's easy.

Then, I made one modification to `ToolBar.html`, adding a reference to the new CSS file after the original:

    <link rel="stylesheet" href="./ToolBar.css" />
    <link rel="stylesheet" href="./ToolBarOverride.css" />

This is all it takes.

## The Effect

So, what does making the mod in this fashion do?   It means that other people can make changes to the javascript or HTML of the toolbar without us masking it.  It also means that updating this mod for future game versions that change this should be very easy -- we know exactly what we changed, and that it only happened at one point.  We can simply update our CSS to work with whatever new HTML we have and we should be good to go. 

It *does* mean that any upstream changes to `ToolBar.html` itself will be masked by our mod, but that's just a fact of life of modding:  you must hook your code in at some point so there's always going to be an interface that can break.  It's just a matter of making fixing it as easy as possible.  In this case, as long as our CSS is updated, we can very simply bring the mod up to date with future game versions by pulling in the updated `ToolBar.html` and adding just that one line to include our CSS.  Tada!

## In Summary

There is almost never One Right Way to do anything in software engineering.   That's simultaneously one of its great joys and its great challenges.  Assessing any problem and trying to figure out which of the tools in your toolkit is the best way to solve it is one of the key skills of a software engineer, and the better you do it the better an engineer you are.  The precise method here will certainly not work for some changes.   The [Working Title mod group](https://github.com/Working-Title-MSFS-Mods/fspackages) that I'm part of are making some very complicated changes that would be practically impossible to do with a near-zero point of contact like this toolbar mod can have.

But that doesn't mean we shouldn't try to get as close as we can.  And we have, indeed, been having discussions recently about how we can restructure some of our mods, and perform future development, to better promote the sort of encapsulation I'm talking about here.  And, hopefully, reading this will have given you something to think about too, and maybe put another tool in your engineering toolkit.

Thanks for your time.  I hope this helps.