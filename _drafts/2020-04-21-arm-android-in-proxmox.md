---
layout: post
title:  "Installing ARM64 on Proxmox"
date:   2020-04-21 12:00:00 -0500
categories: proxmox android arm virtualization
---
After crawling through many dead forums and outdated websites, I thought I would document my journey of running ARM64 Android on Proxmox to make it a little easier on the next person to travel down this road.

# Prerequisites
- A Proxmox server running v5.3 or newer (I used the latest build v6.1-8)
- A copy of [the arm64 GSI image](https://developer.android.com/topic/generic-system-image/releases) from the official Android website.

# VM Setup
Most of these steps come from a [Reddit post](https://www.reddit.com/r/Proxmox/comments/ed2ldo/installing_and_launching_an_arm_vm_from_proxmox/) combined with an [outdated page about using GSI images in QEMU](https://www.cnx-software.com/2014/08/23/how-to-build-and-run-android-l-64-bit-arm-in-qemu/).


[^1]: [https://www.ted.com/talks/jane_mcgonigal_gaming_can_make_a_better_world](https://www.ted.com/talks/jane_mcgonigal_gaming_can_make_a_better_world)
[^2]: [http://www.npr.org/templates/story/story.php?storyId=128081896](http://www.npr.org/templates/story/story.php?storyId=128081896)
