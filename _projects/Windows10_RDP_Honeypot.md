---
layout: page
title: Windows 10 Azure RDP Honeypot
description: Creating a Windows 10 machine on the Microsoft Azure Cloud and open the RDP Port to the internet to allow attackers to attempt to brute force the login.
img: assets/img/RDP-Warning.png
importance: 1
category: work
related_publications: false
---

# Summary

I've experimented with the cloud before (mainly Amazon AWS) and I wanted to branch out into Microsofts cloud platform 
Microsoft Azure, but to see real world attacks on infrastructre that I control.

In this project I was able to create a log and analytics platform within azure that enabled the collection of
various data types but for this projet it was Windows Security logs on the Windows 10 platform.

I was able to view brute force attacks in real time and watch how people from various places in the world
tried to guess my password for an account that doesnt even have RDP rights within the computer.

In total, there where `158,966` attemps made to get into the Windows 10 Machine over a `10 day` period `29/01/2025 - 07/02/2025`
from about `30 different contries`.

# Definitions
- Brute Force
    - The act of attempting multiple times to guess a correct cridentials to gain `unauthorised or unintended access` into a system.
- IDS
    - Stands for `Intrution Detection System` and is a technology used to typicaly scan network traffic and compair it to a set of rules to 
    determine if there is any suspicous or harmful network activity.
- IP
    - Stands for `Internet Protocol`. This is the backbone of the mondern internet and is the thing that allows for communication between networks.
    A IP Address will have 4 Octects and can range from 0-255. Example: 192.168.0.150, 5.254.33.111, 245.77.83.2
- RDP
    - Stands for `Remote Desktop Protocol` and is the protocol that allows a user who is not physicaly infront of the machine to sign into that 
    windows computer. Its like having the computer monitor anywhere, you just have to sign in.

# Expected results & method

# Results

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Top10.png" title="Top 10 IP's" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/top20.png" title="Top 20 Users" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    On the left is the `10 IP addresses` that sent out the most login attempts and the right image is the `20 most common users` that where guessed.
</div>

Here on the left is just a nice visuilasation on what IP addresses are attacking the machine. We can see that `185.7.214.81` was very dedicated to 
trying to get into the machine. But in the later graphs we will see that this IP address in particular was going sort of "All or nothing" and sent 
in thousands of attempts in a short time frame then was never to be seen again.

Although I didnt have any account restrictions or timeouts activated on the machine, but it seems like in a order to avoid or mask the fact that they 
are trying to brute force their way in, they throuttle their pace.

Im thinking there are 2 possible reasons
    - They are attacking multiple machines and have `limmited bandwidth`
    - `Avoiding detection` either from any `Windows firewall, account, network policies` etc. or from a `IDS`

Next image is the Top 20 users that where attacked.

This is in-line with the predicted guess as when you first create a Windows Server machine the default configuration will have the Administrator account
active. Although it is good practice to not only `disable the Administrator account` but to also `restrict it` from even having the `ability to remotely connect`
but im guessing this isnt as common as i'd like it to be. . .

What im most intrested about is the `domain\` usage and the mispelt ADMINISTRATOR spelt `"ADMINISTRATEUR"`
    - I want to test if using the `domain\<USER>` will allow you to connect to a domain you dont know. For example if I had a domain called `Cherry` and I
    wanted to remotely login to a account of the `Cherry` domain, could I just type in `DOMAIN\ADMINISTRATOR` and get access to the 
    domain administrator for Cherry?

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/wholeiprange.png" title="Vast IP Range" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Its kind of hard to guadge, but this graph tries to visualise how many IP addresses attempted to sign in. There where a total of `118 different IP addresses`.
</div>

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/inoneday.png" title="Total attempts in one day" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    In the Azure console for Azure Sentinal, if I set the range to the period length we get to see the events generated and that the `30th` was the `most active day`.
</div>

If you wanted to download the raw CSV data 
    - (results-sorted.csv)[assets/csv/results-sorted.csv]
    - (duplicate-usernames.csv)[assets/csv/duplicate-usernames.csv]

The `results-sorted.csv` contains the extracted data from Azure Logs and Workspace analytics and has the `time generated, account, account type, computer, activity, ip address and logon type`.

The `duplicate-usernames.csv` has a `count` column that displays the ammount of times that particular username was attacked and the `account` column next to it.


### IP Address reputation & location

Next I wanted to get the geographical data and other info from maybe their ISP. As I had over 100 IP addresses I decided to just write a script that uses
the VirusTotal API to gather that info for me instead of manualy going onto their website and having to search them up by hand.

The code is pretty simple:

{% raw %}

```python
import requests
import time

with open("iteration-ip.txt", "r") as ip_file:  # Grab the IP addresses in the file
    IPs = []
    IPs = ip_file.read().split("\n")            # Split each IP address by a newline character

for i in range(len(IPs)):     # loop

    url = "https://www.virustotal.com/api/v3/ip_addresses/" + IPs[i]    # URL for the needed API + the current IP

    headers = {
        "accept": "application/json",
        "x-apikey": "<API-KEY>"     # Add in your API key
    }

    response = requests.get(url, headers=headers)       # Send the response

    print(IPs[i], " Had a response code of ", response)     # feedback in terminal

    time.sleep(0.5)       # Make sure the API doesnt block us for too many requests

    filename = IPs[i] + ".json"         # If you want the json to be human readable, in vscode or codium
                                        # use the hotkey CTRL + SHIFT + I to format it
    with open(filename, "w") as f:
        f.write(str(response.text))             # save the output in a json file named with the IP
```
{% endraw %}

Now if I ran that script I would start getting an output that looks like this:


<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/running-script.jpg" title="Output of running script" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/post-script.jpg" title="Results of Script" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    On the left is the script outputting feedback in the terminal telling us what VirusTotals API response was (200 means good) and the left is the output of the api response stored in the json file with the ip address attached to it.
</div>

With this new data in json format I was able to generate a cool looking `interactive heatmap` of what contries all of these IPs belonged to. 

This heatmap proved to confirm another of my guesses at the start of this project, that it would mostly be `developed contries` that are spraying passwords
everywhere trying to get lucky.

<iframe title="Map of IP locations" aria-label="Map" id="datawrapper-chart-dFe5i" src="https://datawrapper.dwcdn.net/dFe5i/1/" scrolling="no" frameborder="0" style="width: 0; min-width: 100% !important; border: none;" height="416" data-external="1">
</iframe>
<script type="text/javascript">!function(){"use strict";window.addEventListener("message",(function(a){if(void 0!==a.data["datawrapper-height"]){var e=document.querySelectorAll("iframe");for(var t in a.data["datawrapper-height"])for(var r=0;r<e.length;r++)if(e[r].contentWindow===a.source){var i=a.data["datawrapper-height"][t]+"px";e[r].style.height=i}}}))}();
</script>

# Conclution & Preventions

## References