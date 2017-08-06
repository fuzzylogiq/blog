---
title: "Problems with disabling the sync of Desktop and Documents to iCloud"
date: 2017-08-03T16:17:13+01:00
---
In 10.12 Apple introduced the ability for users to sync their Documents and Desktop folder with iCloud so that they would be the same on any computer they were using. In 10.12.4 they gave us the ability to disable it (Rich Trouton has a very informative blog post on how to do so [here](https://derflounder.wordpress.com/2017/03/27/disabling-icloud-desktop-and-documents-syncing/).) You might think that setting the preference (`allowCloudDesktopAndDocuments=false` in `com.apple.applicationaccess`) means you'd be safe from users hiving off all their local data from those folders to the cloud.

You'd be wrong.

There's a nasty bug in Apple's implementation of the preference that concerns the Storage Management dialog which comes up when the computer is low on disk space, and can also be reached through `About this Mac > Storage > Manage` or `System Information > Window > Storage Management`.
![Storage Management Dialog](/img/storage_management.png)
Problematic is the Store in iCloud button in the Recommendations. If clicked it offers to store files from the Documents and Desktop folder in iCloud, regardless of the preference managed above.
![Store in iCloud Popover](/img/store_in_icloud.png)
Eek. What makes it worse is it's then very tricky to stop and reverse as the iCloud Drive preference pane has the option safely greyed out. I've filed a radar with Apple, and the Open Radar is [here](https://openradar.appspot.com/33652807) if you want to dupe it.
