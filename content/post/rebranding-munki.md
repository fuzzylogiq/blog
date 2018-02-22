---
title: "Rebranding Munki"
date: 2017-12-21T13:56:10Z
draft: true
---
#### Background

In this post I'm going to detail how you can rebrand the [Managed Software Center](https://github.com/munki/munki/wiki/Managed-Software-Center-Introduction) application that is the main end-user interface for [munki](https://github.com/munki/munki), Walt Disney Animation Studios' frankly amazing tool for software package management on macOS.

![Orchard's rebranded Managed Software Center](/img/Orchard_Software_Centre_app.png)

So why would you want to rebrand [Managed Software Center](https://github.com/munki/munki/wiki/Managed-Software-Center-Introduction)? Well, for us, we already had a strong branding for our Apple endpoint management service, and we wanted to be able to use that consistently in our management tools so that users saw a familiar logo and name, aiding adoption and trust. Note we're not talking about any custom banners here (those are all handled through [munki's Client Customization](https://github.com/munki/munki/wiki/Client-Customization)) we're talking about the .app's icon and it's name in the Finder.

![Orchard Software Center's icon in the Dock](/img/Orchard_Software_Centre_dock.png)

When we first looked into this for University of Oxford's _[Orchard](https://docs.orchard.ox.ac.uk)_ Apple endpoint management service we found [this](https://groups.google.com/forum/#!msg/munki-dev/WQScNPxpEhA/2b8GCuLQQrMJ) discussion on munki-dev from 2014, leading to this [gist](https://gist.github.com/bochoven/c1c656e0c2e1b1078dfd) by Arjen van Bochoven which appeared to do what we wanted - it git cloned munki, changed the icon and the Finder name for the .app, and could add a postinstall script. The only thing it didn't do that we needed was change the Finder name in our language of choice (UK English). So we wrote some code to further rebrand the localisations and put it up on [our organization on Github](https://github.com/ox-it). 

After some user feedback it was rewritten to take command-line options to make it easier to use. Then a chance remark by Greg Neagle himself led me to wonder why we were always building Managed Software Center with Xcode when in fact we could just use localizations to make the changes we wanted, if we could unpack the package and repack it successfully.  So now that's exactly what the latest version of the code does - it uses macOS's built-in packaging tools to modify a pre-built munkitools mpkg.

What follows is a beginner's guide to using munki-rebrand to bring your own name and icon to Managed Software Center.

#### Prerequisites

* You'll need a Mac running [mac]OS[ X] 10.11 or higher
* You'll need to be somewhat familiar with the CLI (Terminal)
* You'll need to grab the latest version of the munki-rebrand code. You can either:
 * Download a zip/tgz of the [latest release from Github](https://github.com/ox-it/munki-rebrand/releases/latest) and uncompress it somewhere suitable, or
 * Use command line `git` or some other GUI method (_shiver_) to clone the repo e.g. `git clone https://github.com/ox-it/munki-rebrand.git`
* If you want to change Managed Software Center's icon, you'll need an icns file or a 1024x1024 transparent png with your graphics. Creating the icon is somewhat out of scope of this document, but there are some useful tips [!!FIXME!!](here)

#### Method

##### Change app name
* First change directory to wherever the `munki_rebrand.py` script is on your computer e.g. `cd ~/Developer/munki-rebrand/`
* The simplest type of rebranding is just changing the name of the app, which you can do with the `--appname` or `-a` (for short) options. For instance if you wanted to change the name of Managed Software Center in the Finder to _Awesome Software Hub_ you'd use: <br> `sudo ./munki_rebrand.py --appname "Awesome Software Hub"`<br>(Note that it's important to enclose your app name in quotes if it has any spaces in it.)
* You'll need to provide your password to run the script as root
* `munki_rebrand.py` should then download the latest release of `munkitools-X.X.XXXX.mpkg` from Github, unpack it, make your changes to the app name in all localizations, then repack it and you'll be left with the rebranded munkitools mpkg in the current working directory.

##### Change icon
* Do this with the `--icon-file/-i` option
* Assuming you have an icns file called `Awesome_Software_Hub.icns` in the same directory as the script, this time run <br> `sudo ./munki_rebrand.py --a "Awesome Software Hub" --icon-file Awesome_Software_Hub.icns`
* If you specify a 1024x1024 png file, `munki_rebrand.py` will automatically convert it on the fly to an .icns file
* Now `munki_rebrand.py` should download the latest release of munkitools, and rebrand it with your chosen name and icon.

##### Add postinstall script
* Sometimes you might want to add a postinstall script to the rebranded Managed Software Centre installer pkg, that runs after the files are moved into place. For instance, you might want to set some of its preferences with `defaults write` commands. For more information on setting Munki's preferences, please read the [Munki wiki](https://github.com/munki/munki/wiki/Preferences).
* Your postinstall script should be written in one of the supported languages of the macOS installer environment. There's no exhaustive list, but you're probably safe using bash, python, perl or ruby (note your scripts should not have any further library dependencies beyond what is installed by default on a Mac).
* Make sure your script starts with a shebang e.g. `#!/bin/bash` so the system knows what interpreter you want to use. Don't worry about making it executable, `munki_rebrand.py` takes care of that for you.
* Now run something like <br>`sudo ./munki_rebrand.py -a "Awesome Software Hub" -i Awesome_Software_Hub.icns --postinstall awesome_postinstall_script.sh`
* `munki_rebrand.py` now downloads the latest munkitools, rebrands it with your name and icon, and adds your script to the app's installer pkg as its postinstall script

##### Change name of output file
* Perhaps you don't want the output file to still be called `munkitools-X.XXXX.mpkg` so that you can differentiate it from an un-rebranded one
* In this case run<br>`sudo ./munki_rebrand.py -a "Awesome Software Hub" -i Awesome_Software_Hub.icns -p awesome_postinstall_script.sh --output-file Awesome_Software_Hub`
* `munki-rebrand.py` downloads the latest munkitools, rebrands it, adds a postinstall, and renames the output pkg to `Awesome_Software_Hub-X.XXXX.mpkg` where `X.XXXX` is the same as the munkitools mpkg version

##### Signing the output pkg
* Signing the output pkg is optional, but maybe your company requires it or people will be manually installing it and need to pass Gatekeeper
* `munki_rebrand.py` can call `productsign` for you to sign your output pkg if you have an Apple Developer Installer Identity (certificate and private key) somewhere in your keychain. Please note that even if you built a custom mpkg and signed it, the process to rebrand the mpkg will have rendered that signature invalid so you would need to sign again
* Let's say your signing identity is called "Developer ID Installer: Awesome Software (X9TT35F4L8)" do:
`sudo ./munki_rebrand.py -a "Awesome Software Hub" -i Awesome_Software_Hub.icns -p awesome_postinstall_script.sh -o Awesome_Software_Hub --sign-package "Developer ID Installer: Awesome Software (X9TT35F4L8)"`
* You may be prompted for your keychain password if the keychain containing the identity is locked

##### Rebranding a custom Munki build
* You may want to rebrand a custom Munki build, for instance if you need to build Munki for DEP (see Eric Gomez's many useful posts on this)[!!FIXME!!]. Alternatively, you may want to download a specific build from the automatic builds that are produced every time a change is committed to Munki at <https://munkibuilds.org>
* To use a custom Munki build that is local to your computer, just use `--pkg` or `-k` followed by the full path to the mpkg on disk
* To use a Munki build from the web, use `--pkg`/`-k` followed by the full URL to the mpkg e.g. `sudo ./munki_rebrand.py -a "Awesome Software Hub" --pkg https://munkibuilds.org/munkitools3-latest.pkg`

##### Checking it all worked



