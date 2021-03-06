---
layout: default
title: "Hard Drive Speeds"
shortDescription: "The bottleneck is the rotating disks!"
date: 2010-01-19 12:10:39
comments: true
---
So my project to get a hosted subtext blog up and running hit a snag over the weekend because once again I had Hard Drive problems. Once again it was all my own fault of course&hellip;

Last week I rebuilt my main workhorse PC (called Chani, my prettier living room PC being called Irulan for any [Dune (Frank Herbert)](http://www.amazon.co.uk/Dune-Frank-Herbert/dp/0450011844/ref=sr_1_1?ie=UTF8&amp;s=books&amp;qid=1263901782&amp;sr=8-1) fans out there.) I rebuilt it ostensibly to start using Windows 7 on it. I wanted to run it on some faster Hard Drives than the standard SATA 7200RPM rubbish. I can't afford any more SSDs for the moment though, but I remembered somewhere I left at work 2 old WD 150GB 10K Raptor SATA drives. Perfect in RAID 0 as a fast OS drive I thought!

So a whole night spent building a sweet Win 7 setup, SQL Server 2008, Visual Studio 2008, Office 2007, blah blah blah&hellip; It all seemed to be running nice and fast. 2 Days later: BSOD. Ugh. NTFS.Sys was the culprit too. Double Ugh. Dying drives. I guess that was why I left these "sweet" drives at work. OK so a 2nd rebuild on my bog standard WD Caviar 500GB 7200rpm drive. Same setup &ndash; another evening wasted.

The surprising outcome (to me anyway), the WD Caviar 7200rpm was virtually as fast as the 2 10K Raptors in Raid 0 in most tests. I don't really pretend to understand what goes on in the innards of these HDs, but perhaps the fact the raptors were dying made them signifcantly slower. I'm still a proponent of fast small drives in Raid 0 as OS drives, unless you are rich enough to run SSDs everywhere of course! I'm sure when I first used those raptors years ago they were faster&hellip;

Of course at work on our more current systems we are running small 73GB 15K SAS Drives. The difference in speeds in these and even the fastest SATA drives is huge. Of course SAS is pretty pricey too.

I am running a [60GB OCZ Technology, Vertex SSD, MLC-Flash](http://www.scan.co.uk/Products/60GB-OCZ-Technology-OCZSSD2-1VTX60G-MLC-25-SATA-3Gb-s-200MB-s-Read-and-160MB-s-Write-speed) as my OS drive on my living room "media" pc though, and *droool* it is soooo fast.
