---
layout: post
title: "How To Force Microsoft Flight Simulator X: Steam Edition To Run As A Standard User"
date: 2015-06-26
disqus_identifier: 1
---
I recently purchased [Microsoft Flight Simulator X: Steam Edition](http://store.steampowered.com/app/314160/) for my son to play. To my surprise, every time I tried running the game it insisted on running as an administrator. And of course, since the game requires Steam, if you're not already running as an administrator, the game can't find Steam and refuses to run. Seriously, Microsoft? I get the game has been around since Windows XP, but did it really need that even then? I have a computer with a shared user account for the family that my son uses for gaming, and I have it locked down pretty tight to reduce the chance of him causing too much damage. Yes, that account is a standard account. Yes, the administrator account on that computer has a password. And, no, I'm not sharing that password with my son. And I'm definitely not going to try to teach my wife how to run Steam as an administrator. I make things too difficult for her already.

So what to do? I found this [thread](http://steamcommunity.com/app/314160/discussions/0/626329820755274025/) suggesting using Visual Studio to edit the manifest in the exe. While I get the concept and have Visual Studio to do it with, it also seemed like potential for problems. Not only do I risk corrupting the exe, but an update could undo my change at any time. This [thread](http://steamcommunity.com/app/314160/discussions/0/626329820782333837/) mentioned using the Microsoft Application Compatibility Toolkit to force `runAsInvoker` which made me curious. So I did some googling, figured out what that meant, and was eventually able to get that to work. I can now run Flight Simulator as a standard user and not have it ask to run as an administrator. 

How did I do it? Short version first:

1. Download and install the [Microsoft Application Compatibility Toolkit](http://www.microsoft.com/en-us/download/details.aspx?id=7352).
2. Run the Compatibility Administrator (32-bit). You need to the 32-bit version even if your OS is 64-bit because you're matching the game, not the OS.
3. Start the Application Fix wizard.
4. Give the application fix a name of your choice.
5. Setting the vendor name is optional.
6. Set the program file location to where `fsx.exe` is located on your computer.
7. Click Next.
8. From the list of compatibility modes, check RunAsInvoker.
9. Click Next twice then click Finish.
10. Install the database containing the application fix.
11. That's it!

So if you're still reading, I guess you need more detail. And of course, it isn't quite as simple as the above. The first challenge I ran into is you need to run the Compatibility Administrator as an administrator. And because of the way the shortcut is created in the Start Menu you can't right-click on it and choose `Run as administrator`. (I'm beginning to sense a conspiracy here.) The default location for the Compatibility Administrator is `C:\Program Files (x86)\Microsoft Application Compatibility Toolkit\Compatibility Administrator (32-bit)`. Find `Compatadmin.exe`, right-click on it, and choose `Run as administrator`.

{% include figure.html caption="Compatibility Administrator" url="/images/posts/How To Force Microsoft Flight Simulator X (Steam Edition) To Run As A Standard User/Compatibility Administrator.png" %}

Next, click the gear icon in the toolbar to start the `Create new Application Fix` wizard. Enter a name for the fix. The vendor is optional. Set the program file location to where `fsx.exe` is located. The default location for your Steam Library is most likely `C:\Program Files (x86)\Steam`, so you will probably find `fsx.exe` in  `C:\Program Files (x86)\Steam\SteamApps\common\FSX`. I've moved my libary to `E:\Games`, so `fsx.exe` is in `E:\Games\SteamApps\common\FSX` for me.

{% include figure.html caption="Create new Application Fix - Step 1" url="/images/posts/How To Force Microsoft Flight Simulator X (Steam Edition) To Run As A Standard User/Create new Application Fix - Step 1.png" %}

Click Next. Find `RunAsInvoker` in the list of additional compatibility modes and check it. Nothing else needs checked.

{% include figure.html caption="Create new Application Fix - Step 2" url="/images/posts/How To Force Microsoft Flight Simulator X (Steam Edition) To Run As A Standard User/Create new Application Fix - Step 2.png" %}

Click Next then click Next again. No additional compatibility fixes need checked from step 3. In step 4, uncheck everything under `Main Executable`. I haven't verified, but I'm assuming the different options in this list act as restrictions for what this fix will match to. For example, if `BIN_FILE_VERSION` is checked, this fix will only apply to `fsx.exe` when the value of `BIN_FILE_VERSION` is what you see in the list. Since some of these properties could change the next time Flight Simulator is upgraded, it's safest to uncheck everything. After you uncheck everything, click Finish.

{% include figure.html caption="Create new Application Fix - Step 4" url="/images/posts/How To Force Microsoft Flight Simulator X (Steam Edition) To Run As A Standard User/Create new Application Fix - Step 4.png" %}

The first time I did this, I thought I was done at this point. Until I tried the game again and was still prompted to elevate. Then I figured out I had to install the fix. And then I figured out I had to save the database before I could install it. *Sigh.* Ok, fine. So click the big fat Save button on the toolbar, give your database a name and click Ok. For the filename, use the database name and let it save in the default location since it won't be staying there long. Next, right-click on the database - not the fix - and choose `Install`. 

{% include figure.html caption="Save Database" url="/images/posts/How To Force Microsoft Flight Simulator X (Steam Edition) To Run As A Standard User/Database - Save.png" %}

{% include figure.html caption="Install Database" url="/images/posts/How To Force Microsoft Flight Simulator X (Steam Edition) To Run As A Standard User/Database - Install.png" %}

Click Ok for the confirmation you get next. You will now see a section for Installed Databases with your database in the list. When you installed the database, the Compatibility Administrator copied it to a different location. You can see where it went by right-clicking on the database and choosing Properties. But that doesn't matter. What does matter is you should now be able to run Flight Simulator without being prompted to elevate. Try it out and see if it works. If it doesn't and you can't figure out what you missed from my instructions - or if I missed a detail - holler at me in the comments below.

{% include figure.html caption="Database Installed" url="/images/posts/How To Force Microsoft Flight Simulator X (Steam Edition) To Run As A Standard User/Database - Installed.png" %}

Oh, and feel free to take all the credit and tell your friends or family how you outsmarted Microsoft.