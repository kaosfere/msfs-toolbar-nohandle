# About MSFS package metadata

A package for MSFS contains two metadata files:  `manifest.json` and `layout.json`.  Since they're small, let's look at the ones from this mod as an example.  Here's its `manifest.json`:

    {
        "creator": "Rob Jones",
        "package_version": "0.1.0",
        "release_notes": {
            "neutral": {
                "OlderHistory": "",
                "LastUpdate": ""
            }
        },
        "content_type": "CUSTOM",
        "minimum_game_version": "1.7.14",
        "title": "Titlebar Handle Zapper",
        "dependencies": [
            "fs-base-ui-pages"
        ],
        "manufacturer": "",
        "total_package_size": "00000000000000001613"
    }

Most of this should be fairly self-explanatory.  Not all of these fields are required but this would provide a good basic template for making your own if you wished to do so.  In particular the dependencies section can be helpful.  If you know that your mod will not work without other ones installed it would be good to add them as dependencies here.   I like to add, as well, the official packages that my mod modifies, if only to provide a little bit of clarity.

You can also specify versions as part of a dependency.  I've found that most people don't do this and for the most part that will be fine.  It could come in useful, though, if you know that you're making a change that will absolutely break with an earlier version of something else.  It can also be used for protecting the future:  if you're changing something you suspect will break badly if the underlying dependencies change you can provide a version spec here and your mod will be automatically disabled if a incompatible version is found.

(One note:  I do not know, and the SDK documentation does not yet explain, what can be used as valid version dependency specifiers.  Hopefully it works something like the [pip version specifiers](https://pip.pypa.io/en/stable/reference/pip_install/#requirement-specifiers), which would allow you to say something like "this mod works with every 1.8 version of the game, but won't work on anything later", but that needs more experimentation or documentation.  This would be useful, if it's possible, when it becomes more clear what Asobo intend to do with their version numbers and if they're going to follow some form of [semantic versioning](https://semver.org/).)

As for `layout.json`, that can be the subject of a little more confusion and improperly formed ones are widespread in the mod community.  But the don't have to be, with a little automation making a proper `layout.json` is dead simple.  Here's the one for this mod:

    {
        "content": [
            {
                "path": "html_ui/Pages/Toolbar/ToolBar.html",
                "size": 1509,
                "date": 132452768417158840
            },
            {
                "path": "html_ui/Pages/Toolbar/ToolBarOverride.css",
                "size": 104,
                "date": 132452768221961470
            }
        ]
    }

Two of the components, the path and the size, seem obvious.   That date format, though, is a somewhat unusual one -- it is based on the [Win32 filetime](https://docs.microsoft.com/en-us/windows/win32/api/minwinbase/ns-minwinbase-filetime), which is similar to the [Unix epoch](https://www.epochconverter.com/), but is even more bizarre:  it measures the number of 100-nanosecond intervals since 1 January 1601.  Yep.

It currently does not seem to matter what the size and date in the layout are, as long as the paths are correct.   But it is simple enough to make a correctly-specified layout file, and it is possible that at some point in the future MSFS will enforce correct files, at which point you'll be glad you did.

There are a number of utilities you can use to automate the creation of a correctly formed `layout.json`.  The folks at the freeware [A320](https://github.com/flybywiresim/a32nx) project have a [simple but solid python script](https://github.com/flybywiresim/a32nx/blob/master/A32NX/build.py) that would be a great start for folks to use.  It's what I use when I'm trying to roll something quick outside of the Working Title build system.

So now you know.