# OrcaSlicer-Profiles-Disappearing
Here is WHY some of your OrcaSlicer profiles can "suddenly" disappear

Many people report of how some, or even all, of their OrcaSlicer Profiles are suddenly "missing"... gone.
This may be just one, or some, or a Printer Priofile, or a Filament Profile, or a Process Profile.
Whilst it happens to enough people to annoy THEM, and have "quite a lot of people" report it and want
to find out whay happened or why - and get them BACK! - it is a very small percentage of the total user
base. Well, according to the OrcaSlicer team....  and I also believe that is true.
I beleive I have found out WHY they "disappear".....

I don't know a lot about JSON, or OrcaSlicer specifics, so I had to Reverse Engineer what is going on.
So some of my explanation/s might be amiss and not be fundamentally correct, but it does explain what
is going on and it is how you can FIX it.

The file structure/system that OrcaSlicer uses is a Hierarchical one. A file (profile) can be an
individual file that RELIES on a link to a "higher level" other file. And these links can be what
seems toi be "unlimited" and then you can have a chain of files that the final file RELIES on every
file ion that chain - every link in that chain.

When you edit a Profile, very often you are just editing that FINAL file in the chain.
eg If you save some changes that you made to a buily in "system" profile, it will not let you save
that profile - you have to choose between saving it as a User Preset or an
When you do that, it will save a new file with JUST your changed parameters, but link to the Profile
that you had begun with. And that profile might have been part of a longer chain. or not.
Your new Profile will work perfectly fine......

The ways it gets "killed" and then "disappears" - it seems to me - is if one day you decide to delete
a Profile, and that happens to be one that is used by OTHER Profiles in their chain. With that file
you deleted gone now, the higher level file is 'broken' and will not show up in the profile lists
anymore. But the Profile file (JSON file) itself is actually still within the OrcaSlicer file system.

When you first install and set up OrcaSlicer it will have asked you to ADD a printer (or as many as you
want to). This is where it adds the System Profiles.  This also where you COULD REMOVE any given System
Profile quite easily... and if you had since created your own Profile(s) that link from any of these
System Profiles (and almost certainly they would have!), then your added Profile will "disappear"
when its dependent linked profile is no longer there!

I think that SOME people have decided to "clean up" their Profile list, one day, and thus deleted some
that they CAN actually delete from within the OrcaSlicer main screen profile lists (you can't delete
a "System" Profile from there), or removed a System Profile via that main "Printer List". eg Decided
you don't need the Creality K1 printer anymore, as you have too many Profiles cluttering up the lists,
so you remove that.  BUT your own Profiles NEEDED those lower level, Base, files in their chain!

The System profiles are the ones line when you add a brand of Printer (Creality) and a model, and then
OrcaSlicer adds a bunch of various Profiles for that Printer. Typically a printer Profile, but also a
bunch of Filament Profiles, and a few "quality" process profiles. Even these System Profiles will
most often have their own links to other lower level "Base" Profiles.

First, let's list WHERE these files all reside in your system.
C:\Users\<your account>\AppData\Roaming\Orcaslicer\
and then in either "user\default" or "system"
Then within those folders you have "Machine", "Filament" and "Process" - which are the three Profile
types you have per printer.
The files withn SYSTEM are those "System files" that you cannot delete - except via removing a printer
brand and model.

Within those Profile folders there are all the JSON files, which are the files holding all the
Profile Parameters, but also otehr header/linking etc information also.

Within any Profile File their can be an entry named:  "inherits"   which will have a data value of
another Profile name that is it linked from.
It will also have an entry named:   "from" and the data value lists where that Inherit file comes from.

Here is an example, taken from the "0.15mm Optimal @ Creality Ender3V2" Profile JSON file:

{
    "type": "process",
    "setting_id": "GP004",
    "name": "0.15mm Optimal @Creality Ender3V2",
    "from": "system",
    "inherits": "fdm_process_creality_common",
    "instantiation": "true",
    
I am not 100% sure what this "from" : "system" parameter specifically points to, but you will find there
is a System\Creality.json that was added, when you added that printer model, and inside that file is
a complete list of all the Creality models and what path they are in within the OrcaSlicer file structure.
That file has a list of Machines, Filaments and Processes, so it is a TABLE of all the files to do with
Creality. And within these it lists the "sub-PATH" to where all those files are.
So if you look up "0.15mm Optimal @Creality Ender3V2" you can find out what path/file it came from.

{ The "from": "system" entry seems to mean it was from the main system folder, or it can be "user"
  which then would mean it came from the user folder. And that is possibly a Flag to tell if a file
  is allowed to be editable or not.}

If you ALTER any parameter(s) within that Profile - let's say within the Process Profile - when you go to
SAVE that, it won't let you alter that base System Profile, it will force you to choose User Preset, or
Project Inside Preset - and it will default to User Preset, so most people would have used that.

This will create a new JSON file, with whatever name you gave that, and that will be inside the path:
user\default\process

If you look into that file, you will find:

{
    "from": "User",
    "inherits": "0.20mm Standard @Creality K1Max (0.4 nozzle)",
    "initial_layer_print_height": "0.24",
    "is_custom_defined": "0",
    "name": "0.20mm Standard @Creality K1Max (0.4 nozzle) - Copy",
    "print_settings_id": "0.20mm Standard @Creality K1Max (0.4 nozzle) - Copy",
    "version": "1.8.0.0",
    "wall_loops": "2"
}

And that is ALL it has. So you can see it is not a full Profile. In this example I only changed TWO
parameters of the Base Profile
"initial_layer_print_height": "0.24"
"wall_loops": "2"

And you will see the "inherits" value is that original Base Profile name that I had used to begin from.
Thus you now have AT LEAST the two files, linked, and both required.
If you uninstall the Creality files, yours will remain BUT with the link file gone yours will not be
listed within OrcaSlicer anymore!

Let's look within the first linked file:
"inherits": "0.20mm Standard @Creality K1Max (0.4 nozzle)",

In that file we have:

{
	"type": "process",
	"setting_id": "GP004",
	"name": "0.20mm Standard @Creality K1Max (0.4 nozzle)",
	"from": "system",
	"inherits": "fdm_process_creality_common",
	"instantiation": "true",

So that file ALSO Inherits another lower level file!!
"inherits": "fdm_process_creality_common"

Looking into that file, it ALSO inherits a lower level file!!

{
    "type": "process",
    "name": "fdm_process_creality_common",
    "from": "system",
    "instantiation": "false",
    "inherits": "fdm_process_common",

And looking into that next inheritted file, it ALSO inherits yet another file!
"inherits": "fdm_process_common"

{
    "type": "process",
    "name": "fdm_process_common",
    "from": "system",
    "instantiation": "false",

And that file does NOT inherit any further files!  Phew, finally!


If ANY ONE of those above files are removed, or deleted, then any higher level file that "inherits" that file
will not work anymore!

=====================================================================================================================

So now, how to FIX your "disappeared" profiles!!!

Well, you need to be sure any inheritted files LISTED do exist. Or edit it to point to another similar file.

Maybe you removed the Creality K1 max printer?  If you put it back then the inherit chain will be there again,
thus your own Profiles will work again and "re-apprear".

If you don't want, or like, the big clutter of files you get from adding a printer, you could delete all the ones
you do not want - but you will need to keep any files that Inherits in your profile chain use!

You could possibly add your own "complete" Profile that does not require a chain of inherits. But I think you
at least always need to inherit from a base OrcaSlicer file.
I have not worked out how to do this yet.... but it should be possible to make you own file that ONLY relies on
the basic, built in, OrcaSlicer "name": "fdm_process_common" etc.  Thus no need to add any other printers.
Or manually add ONLY the chain that you want, from some printer (whether that is your brand or not) and edit
the names etc to be what you want. So it looks 'good' in the OrcaSlicer profiles list (its name etc).



But anyway, THAT is why YOU BROKE your OrcaSlicer profiles and they "disappeared" !!!!


