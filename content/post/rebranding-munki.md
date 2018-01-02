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
 * Use command line `git` or some other GUI method (shiver) to clone the repo e.g. `git clone https://github.com/ox-it/munki-rebrand.git`
* If you want to change Managed Software Center's icon, you'll need an icns file or a 1024x1024 transparent png with your graphics. Creating the icon is somewhat out of scope of this document, but there are some useful tips [!!FIXME!!](here)

#### Method
##### Change app name
* First change directory to wherever the `munki_rebrand.py` script is on your computer e.g. `cd ~/Developer/munki-rebrand/`
* The simplest type of rebranding is just changing the name of the app, which you can do with the `--appname` or `-a` (for short) options. For instance if you wanted to change the name of Managed Software Center in the Finder to _Awesome Software Hub_ you'd use: <br>`sudo ./munki_rebrand.py --appname "Awesome Software Hub"`<br>(Note that it's important to enclose your app name in quotes if it has any spaces in it.)
* `munki_rebrand.py` should then download the latest release of `munkitools-X.X.XXXX.mpkg` from Github





