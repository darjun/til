# AppData
## What is AppData?
When you install a program on Windows, it will typically be installed in either **C:\Program Files** or, in the case of a 32-bit program, to **C:\Program Files (x86)**. The application will be installed for all users on the machine and require administrator access to write to. Any application settings stored in this folder will also propagate for all users.

That's where AppData comes in. It's a hidden folder that resides under each user folder. It's located in **C:\Users\<username>\AppData** and contains program-specific information that may not relate to the program's ability to run, such as user configurations. In your AppData folder, you will find files like:

* User-specific installations
* App configuration files
* Cached files

If you've ever installed a program that asked you whether you wanted to install it for all users or not, it was basically asking you if you wanted to install it into Program Files or AppData. Python is one such program that does this, and Discord will also install to a user's AppData. Additionally, there are three subfolders in AppData, and their differences are important to note.

## What is Local?
The Local folder is for storing files that can't move from your user profile and also often contains files that may be too large to synchronize with a server. For example, it might house some files that are needed for a video game to run or your web browser cache, which are files that may be too large or wouldn't make sense to transfer anywhere else. A developer might also use Local to store information that pertains to file paths on this particular machine. Moving these configuration files to another machine might cause programs to stop working, as the file paths would not match up.

Other files stored here tend to be log files, temporary files, or non-essential data.

## What is LocalLow?
LocalLow is very similar to Local, but the "low" in the name refers to a lower level of access granted to the application. For example, a browser in incognito mode may be limited to only accessing the LocalLow folder to prevent it from being able to access the normal user data stored in Local. Essentially, this is aimed at applications that are running with with more restricted security permissions.

## What is Roaming?
If you use a Windows machine on a domain (that is, a network of computers with a central domain controller that handles your login), then you might be familiar with the Roaming folder. Files in this folder are synced to other devices if you log in on the same domain since they're considered important for using your device. This could be your web browser favorites and bookmarks, important application settings, and more.

It's recommended to use this folder when the data being stored can be moved from device to device without any problems. For example, Minecraft stores its world files, screenshots, and more in the Roaming folder because these files can all be taken and migrated to a new device, where it's expected to work.

Roaming is ideal for corporate environments, including settings such as Outlook profiles and networked printer configurations. It helps bring consistency to a user's environment across different machines within a network by storing user-specific settings and files.