---
layout: page
title: Windows 10 Azure RDP Honeypot
description: Creating a Windows 10 machine on the Microsoft Azure Cloud and open the RDP Port to the internet to allow attackers to attempt to brute force the login.
img: assets/img/RDP-Warning.png
importance: 1
category: work
related_publications: true
---

[_source for background picture_](https://community.hostek.com/uploads/default/original/1X/26d2a6b4777c861e314ff5358e94452a6dad169e.png)

# Summary

I've experimented with the cloud before (mainly Amazon AWS) and I wanted to branch out into not only
Microsofts cloud platform Microsoft Azure, but to see real world attacks on infrastructre that I control.

In this project I was able to create a log and analytics platform within azure that enabled the collection of
various data types but for this projet it was Windows Security logs on the Windows 10 platform.

I was able to view brute force attacks in real time and watch how people from various places in the world
tried to guess my password for an account that doesnt even have RDP rights within the computer.

In total, there where `158,966` attemps made to get into the Windows 10 Machine over a `10 day` period `29/01/2025 - 07/02/2025`

# Configuration



# Expected results

# Results

div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>

# Conclution & Preventions