---
title: "Rogue Amoeba Echelon InstantOn Installs"
date: 2017-12-13T14:23:44Z
draft: false
---
We recently had a request from a user to package the [Piezo](https://www.rogueamoeba.com/piezo/) app so they could easily record Skype conversations and other computer audio. The user had been running the app from their home folder but then needed admin creds in order to install _InstantOn_ - which is Rogue Amoeba's audio driver that can instantly start recording audio from apps without restarting them. The app shows the dialog to install the driver under _Install Extras..._ in the Piezo menu, and also if the user tries to record from an app whose audio stream it can't immediately connect to.

Poking around in the app itself, I found a number of frameworks, including the ExtrasInstaller.framework which contained the driver itself and a binary called `EchelonInstaller` which looked promising:

	$ ls -l /Applications/Piezo.app/Contents/Frameworks/ExtrasInstaller.framework/Versions/A/Resources/
	total 232
	-rwxr-xr-x@ 1 ouit0354  staff  54352 28 Sep 18:45 EchelonInstaller
	drwxr-xr-x@ 6 ouit0354  staff    204 28 Sep 18:45 English.lproj
	-rw-r--r--@ 1 ouit0354  staff   1134 28 Sep 18:45 Info.plist
	drwxr-xr-x@ 4 ouit0354  staff    136 28 Nov 17:03 InstantOn
	drwxr-xr-x@ 4 ouit0354  staff    136 28 Sep 18:45 SHVersion
	-rw-r--r--@ 1 ouit0354  staff   9702 28 Sep 18:45 alert_bubble.tiff
	-rw-r--r--@ 1 ouit0354  staff  13776 28 Sep 18:45 icon_instanton.tiff
	-rw-r--r--@ 1 ouit0354  staff  12626 28 Sep 18:45 icon_schedulehelper.tiff
	-rw-r--r--@ 1 ouit0354  staff   9734 28 Sep 18:45 install_bubble.tiff
    $ ls -l /Applications/Piezo.app/Contents/Frameworks/ExtrasInstaller.framework/Versions/A/Resources/InstantOn
	total 1800
	-rw-r--r--@ 1 ouit0354  staff    2117 28 Sep 18:45 InstantOn.driver-Info.plist
	-rw-r--r--@ 1 ouit0354  staff  915360 28 Sep 18:45 InstantOn.driver.tgz

At first, I tried to poke the `EchelonInstaller` binary with `strings` and it certainly looked like it could take arguments such as `install`, `uninstall` and `remove` but every time I tried to run it I got a segmentation fault.

	$ strings EchelonInstaller
	extras
	Echelon installer tool running as %s:%s
	Arguments: %@
	No option was specified in the arguments %@
	%@ %s%s
	exists
	does not exist
	 but isn't a directory?!?
	install
	No archive path was specified for install in the arguments %@
	Missing archive file: %@
	Archive file should not be a directory: %@
	uninstall
	Unrecognized arguments for uninstall: %@
	Trying to uninstall when not installed.
	Unrecognized option in arguments %@
	remove
	Removing %@
	Error removing '%@': %@
	/usr/bin/tar
	-xvzf
	/usr/sbin/chown
	root:wheel
	/usr/bin/killall
	coreaudiod
	_coreaudiod
	Echelon installer reporting no error
	message
	Note: %@
	com.rogueamoeba.EchelonInstaller
	/Library/Audio/Plug-Ins/HAL/
	InstantOn.driver
	Contents/MacOS/instanton-agent
	The tool '%@' could not be launched
	The tool '%@' crashed with arguments %@
	The tool '%@' returned %d with arguments %@
	Running %@ %@
	    %@
	    <task launch failed>
	diagnostic
	...
	
But the commands it runs seemed to be there - `tar -xvzf`, `/usr/bin/chown root:wheel`, `killall coreaudiod` etc., so I then tried to write a script that untarred the driver to the correct place, chown'd it and then killed coreaudiod. Hilarity ensued as I removed the audio outputs from my laptop. Also, _InstantOn_ didn't look that happy in the GUI and was complaining that it wasn't compatible with my OS version.

A quick word with [Tim Sutton](https://macops.ca) and I discovered he'd also been writing scripts to automate Rogue Amoeba installs to VMs and he was poking `EchelonInstaller` to do so, having gone far further down the rabbithole and run `newproc.d` to see exactly what processes it was spawning. (Before you run off and look into `newproc.d` you should know that it's part of the `dtrace` suite and therefore can't be run with SIP enabled. You'll need to disable SIP in order to get anything useful out of it.) Sure enough it was untarring, chowning and killing coreaudio but also setting up some LaunchDaemons that I'd missed.

Finally I realised I was getting the segfault because I was trying to run `EchelonInstaller` unprivileged, I ran it with `sudo` and finally, _InstantOn_ reported back that it was successfully installed (and recording from all currently running apps worked.) So in the end all you need to do is use your favourite management tool to run something like this as a postinstall script:

	#/bin/bash
	APP_PATH="/Applications/Piezo.app"
	EXTRAS="$APP_PATH/Contents/Frameworks/ExtrasInstaller.framework/Version/Current/Resources"
	"$EXTRAS/EchelonInstaller" install "$EXTRAS/InstantOn/InstantOn.driver.tgz"

Presumably this works with other Rogue Amoeba apps and their extras, so test at your convenience! Many thanks to Tim Sutton for his help with this post.