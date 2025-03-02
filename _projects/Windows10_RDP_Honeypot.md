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

I've experimented with the cloud before (mainly Amazon AWS) and I wanted to branch out into Microsoft's cloud platform. 
Microsoft Azure, but to see real-world attacks on infrastructure that I control.

In this project, I was able to create a log and analytics platform within Azure that enabled the collection of
various data types but for this project it was Windows Security logs on the Windows 10 platform.

I was able to view brute force attacks in real-time and watch how people from various places in the world
tried to guess my password for an account that doesn't even have RDP rights within the computer.

In total, there were `158,966` attempts made to get into the Windows 10 Machine over a 10-day period `29/01/2025 - 07/02/2025`
from about `30 different countries`.

# Definitions
- Brute Force
    - The act of attempting multiple times to guess the correct credentials to gain `unauthorized or unintended access` into a system.
- IP
    - Stands for `Internet Protocol`. This is the backbone of the modern internet and is the thing that allows for communication between networks.
 An IP Address will have 4 Octects and can range from 0-255. Example: `192.168.0.150 | 5.254.33.111 | 245.77.83.2`
- RDP
    - Stands for `Remote Desktop Protocol` and is the protocol that allows a user who is not physically in front of the machine to sign into that 
 Windows computer and instead allowing sign-in through the network or internet. It's like having the computer monitor anywhere,
 you just have to sign in.
- Azure Sentinal
    - Azure Sentinal is `Microsofts integrated cloud SEIM solution` and can intake logs from various workspaces, read them, and compare them to 
 a set of rules to determine if there is any `suspicious activity`. People like `SOC Analysts are then able to log into the portal to look at `cases, `
    `incidents, create new rules, playbooks, send in KQL queries to gather log information, threat management and configuration for the intake of `
    `logs, data connections and create automation rules.`

# Method

- In my results, I want to find some of the top offenders and gather information on them and their habits
    - Attributes like Geo-location
    - IP Reputation on public Blacklists
    - What user was attacked the most


<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Windows-machine.png" title="Windows 10 Config" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

First we need to create the Windows machine and configure it to have a uncommon and complex username and password and a public IP address.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/azureSentinal.png" title="Azure Sentinal" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Once the VM is up and running we can create the Sentinal solution, logs, and analytics workspace and link it up to the Windows VM using the Data Connecters function

The data connectors function allows the Windows VM to send all of its logs to the Logs and Analytics workspace so Sentinal can read them.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/azuresentinal.png" title="Azure Sentinal" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The Medium alert has the following KQL Query

{% raw %}
```kql
//Checks in the security event logs and discards any logs that have the user "SYSTEM" 
//and looks for event code 4625 which is a failed login attempt
SecurityEvent
| where Account !has "SYSTEM" and EventID == "4625"
```
{% endraw %}

The Successful login attempt rule has the same query but the EventID is "4624".

The query will run every 5 minutes and if there are matches then it will generate an incident. These incidents can have automated responses, playbooks, and other
automated tasks assigned to it.

# Results

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/Top10.png" title="Top 10 IP's" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image is the `10 IP addresses` that sent out the most login attempts.
</div>

Here on the left is just a nice visualization of what IP addresses are attacking the machine. We can see that `185.7.214.81` was very dedicated to 
trying to get into the machine. But in the later graphs, we will see that this IP address in particular was going sort of "All or nothing" and sent 
in thousands of attempts in a short time frame then was never to be seen again.

Although I didn't have any account restrictions or timeouts activated on the machine, it seems like to avoid or mask the fact that they 
are trying to brute force their way in, they throttle their pace.

- I think there are 2 possible reasons
    - They are attacking multiple machines and have `limited bandwidth`
    - `Avoiding detection` either from any `Windows firewall, account, network policies`, etc., or from an `IDS`

<div class="row justify-content-sm-center">
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/top20.png" title="Top 20 Users" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image is the `20 most common users` that where guessed.
</div>

These are the most attacked accounts and they make sense. The default configuration on a Windows Server will have you create a password for the
Administrator account and it's unlikely that server administrators will want to change or disable the pre-made account as it's the easier option. 
Although it is good practice to not only `disable the Administrator account` but to also `restrict it` from even having the `ability to remotely connect`
but I'm guessing this isn't as common as I'd like it to be. . .

- What I'm most interested in is the `domain\` keyword
    - I want to test if using the `domain\<USER>` will allow you to connect to a domain you don't know. For example, if I had a domain called `Cherry` and I
    wanted to remotely log in to an account of the `Cherry` domain, could I just type in `DOMAIN\ADMINISTRATOR` and get access to the 
    domain administrator for Cherry? Or will I have to know the domain to sign into that domain administrator (`CHERRY\ADMINISTRATOR`)?

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/wholeiprange.png" title="Vast IP Range" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Its kind of hard to gauge, but this graph tries to visualize how many IP addresses attempted to sign in. There were a total of `118 different IP addresses`.
</div>

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/inoneday.png" title="Total attempts in one day" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    In the Azure console for Azure Sentinal, if I set the range to the period length we get to see the events generated and that the `30th` was the `most active day`.
</div>

-   If you wanted to download the raw CSV data 
    - [results-sorted.csv](/assets/csv/results-sorted.csv)
    - [duplicate-usernames.csv](/assets/csv/duplicate-usernames.csv)

The `results-sorted.csv` contains the extracted data from Azure Logs and Workspace analytics and has the `time generated, account, account type, computer, activity, ip address, and logon type`.

The `duplicate-usernames.csv` has a `count` column that displays the number of times that particular username was attacked and the `account` column next to it.


### IP Address reputation & location

Next, I wanted to get the geographical data and other info from maybe their ISP. As I had over 100 IP addresses I decided to just write a script that uses
the VirusTotal API to gather that info for me instead of manually going onto their website and having to search them by hand.

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

    response = requests.get(url, headers=headers)       # Send the request

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
</div>
<div class="caption">
    This is the script outputting feedback in the terminal telling us what VirusTotals API response was (200 means good).
</div>

<div class="row justify-content-sm-center">
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/post-script.jpg" title="Results of Script" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This is the output of the api response stored in the json file with the ip address attached to it.
</div>

This is a sample of the JSON data I got from VirusTotal.

{% raw %}

```json
{
    "data": {
        "id": "185.7.214.81",
        "type": "ip_address",
        "links": {
            "self": "https://www.virustotal.com/api/v3/ip_addresses/185.7.214.81"
        },
        "attributes": {
            "last_https_certificate_date": 1637657634,
            "asn": 207566,
            "last_analysis_results": {
                "Acronis": {
                    "method": "blacklist",
                    "engine_name": "Acronis",
                    "category": "harmless",
                    "result": "clean"
                },
//===================== CUT =====================\\
                },
                "ZeroFox": {
                    "method": "blacklist",
                    "engine_name": "ZeroFox",
                    "category": "undetected",
                    "result": "unrated"
                }
            },
            "last_analysis_stats": {
                "malicious": 9,
                "suspicious": 2,
                "undetected": 28,
                "harmless": 55,
                "timeout": 0
            },
            "as_owner": "Chang Way Technologies Co. Limited",
            "last_modification_date": 1739509942,
            "country": "RU",
            "reputation": -1,
            "regional_internet_registry": "RIPE NCC",
            "jarm": "29d29d20d29d29d00029d29d29d29decf7fca7fb071cf5a94949fbcc16ef69",
            "continent": "EU",
            "network": "185.7.214.0/24",
            "last_analysis_date": 1739134408,
            "total_votes": {
                "harmless": 0,
                "malicious": 1
            },
            "whois_date": 1736598507
        
    }
}
```

{% endraw %}

This is a cut of the JSON data that I was able to receive but I cut out a lot of the garbage and useless data from it like http certificates and RSA public keys that take up 
too much space on this page.

-If you want the data from `all` of the IP addresses I put it in a zip file along with the IPs in a list and the file version of the script
    - [IP-Reputation-script.zip](/assets/zip/IP-Reputation-script.zip)

This data contained things like information on the ISP, like contry, company, encryption keys etc.

These where the top 10 offenders:

1. 185.7.214.81
    - Attempts          28,013
    - Contry            Russian Federation (RU)
    - State/City        Moskva/Moscow
    - ISP               Chang Way Technologies Co. Limited
    - Blacklist/Rep     9/79
    - Tor/Proxy/VPN     False
    - Services          VPN Server
    
2. 31.43.185.40
    - Attempts          16,330
    - Contry            Netherlands
    - State/City        Noord-Holland/Amsterdam
    - ISP               FOP Dmytro Nedilskyi
    - Blacklist/Rep     2/79
    - Tor/Proxy/VPN     False
    - Services          Datacenter
    
3. 87.251.75.145
    - Attempts          15,641
    - Contry            Netherlands
    - State/City        Noord-Holland/Amsterdam
    - ISP               Aleksandr Valerevich Mokhonko
    - Blacklist/Rep     9/79
    - Tor/Proxy/VPN     False
    - Services          Datacenter
    
4. 31.43.185.39
    - Attempts          15,595
    - Contry            Netherlands
    - State/City        Noord-Holland/Amsterdam
    - ISP               FOP Dmytro Nedilskyi
    - Blacklist/Rep     2/79
    - Tor/Proxy/VPN     False
    - Services          Datacenter
    
5. 31.43.185.42            
    - Attempts          11,838
    - Contry            Netherlands
    - State/City        Noord-Holland/Amsterdam
    - ISP               FOP Dmytro Nedilskyi
    - Blacklist/Rep     1/79
    - Tor/Proxy/VPN     False
    - Services          Datacenter
    
6. 185.243.96.107
    - Attempts          11,691
    - Contry            United States (USA)
    - State/City        New York/New York City
    - ISP               Demenin B.V.
    - Blacklist/Rep     1/79
    - Tor/Proxy/VPN     False
    - Services          Datacenter
    
7. 31.43.185.41            
    - Attempts          10,707
    - Contry            Netherlands
    - State/City        Noord-Holland/Amsterdam
    - ISP               FOP Dmytro Nedilskyi
    - Blacklist/Rep     2/79
    - Tor/Proxy/VPN     False
    - Services          Datacenter
    
8. 45.141.84.154
    - Attempts          10,672
    - Contry            Russian Federation (RU)
    - State/City        Sankt-Peterburg/Saint Petersburg
    - ISP               Media Land LLC
    - Blacklist/Rep     1/79
    - Tor/Proxy/VPN     False
    - Services          Datacenter
    
9. 194.165.17.25
    - Attempts          9,386
    - Contry            Belgium
    - State/City        Brussels Hoofdstedelijk Gewest/Brussels
    - ISP               FlyServers S.A.
    - Blacklist/Rep     7/79
    - Tor/Proxy/VPN     True (Hosting Provider)
    - Services          VPN Server
    
10. 185.7.214.177
    - Attempts          7,149
    - Contry            Russian Federation (RU)
    - State/City        Moskva/Moscow
    - ISP               Chang Way Technologies Co. Limited
    - Blacklist/Rep     7/79
    - Tor/Proxy/VPN     False
    - Services          VPN Server

- From the top 10 attackers:
    - 3 where from the Russian Federation
    - 5 where from the Netherlands
    - 1 was from the United States and
    - 1 was from Belgium

With this new data in json format, I was able to generate a cool-looking `interactive heatmap` of what countries all of these IPs belonged to. 

This heatmap proved to confirm another of my guesses at the start of this project, that it would mostly be `developed countries` that are spraying passwords
everywhere trying to get lucky.

## To see the descriptive text in the embeded link, you have to highlight the text or turn on light mode as the color is of the text is black

<iframe title="Map of IP locations" aria-label="Map" id="datawrapper-chart-dFe5i" src="https://datawrapper.dwcdn.net/dFe5i/1/" scrolling="no" frameborder="0" style="width: 0; min-width: 100% !important; border: none;" height="416" data-external="1">
</iframe>
<script type="text/javascript">!function(){"use strict";window.addEventListener("message",(function(a){if(void 0!==a.data["datawrapper-height"]){var e=document.querySelectorAll("iframe");for(var t in a.data["datawrapper-height"])for(var r=0;r<e.length;r++)if(e[r].contentWindow===a.source){var i=a.data["datawrapper-height"][t]+"px";e[r].style.height=i}}}))}();
</script>

# Conclution & Preventions

So, we learned that if you have a server exposed to the internet, no matter if you think that you have hidden it, never share its IP address, and don't tell anyone that it exists, 
it will still get targeted. So how can we prevent this from happening?

## Cyber Security Frameworks

- Mitre ATT&CK Brute force
    - On [attack.mitre.org](https://attack.mitre.org) there is a page in the `Credential Access` section called `Brute Force` and explains some useful definitions like
    `Password Guessing, Password Cracking, Password Spraying, and Credential Stuffing`.
    - It explains mitigation techniques like `Multi-Factor Authentication, Password Policies, and User Account Management` and how to utilise them to defend your
      system effectively.
    - It has Detection mechanisms and explains how to collect the necessary data and process it to get valuable information and determine whether or not there are possible
      brute-forcing or credential exfiltration happening.
    - It contains lots of `references` on how it has gathered its information and you can access them to gather more information and strategies on how to mitigate this attack

- If you are trying to prevent attacks on a `Web App`, then `OWASP` could help [owasp.org](https://owasp.org/www-community/attacks/Brute_force_attack)
    - OWASP gives an in-depth guide to regular users on the description and risk of how brute forcing can affect a company and a guide to penetration testers on how to find and 
      exploit this.

- The `Essential Eight` by the `Australian Cyber Security Centre` (ACSC) [essential eight](https://www.cyber.gov.au/resources-business-and-government/essential-cyber-security/essential-eight/essential-eight-explained)
    - Multifactor Authentication
    - Restrict administrative privilege

With multifactor Authentication even if someone can get a user password they are not able to get into the account because they have to complete another challenge
like entering a 6-digit OTP code or E-Mail verification.

Restricting Administrative privileges can also help a company, if a user that gets breached doesn't have MFA then it will be the account policies assigned to that user that will
stump the attacker, with minimal privileges then an attacker isn't able to access critical applications or data reducing the risk.

If every company in Australia has not already, they should start adopting this framework as it is becoming the new Australian standard.

## Preventions

If we take a look at the Mitre ATT&CK and the Essential Eight documents on brute force attacks, then can determine that 2 things can stop attackers extremely 
effectively.

1. MFA
2. Access Controls

MFA or Multi-factor Authentication will stop almost all attacks regarding straight brute forcing and guessing credentials. Even if someone gets a user username and password they
have to go through another layer of security that is often extremely hard to get through.

Having good Access Controls not only for the user but for the devices is also good practice, if you have a machine with an RDP port open to the internet, then maybe restrict what IP
addresses can access that port by having `whitelists`. Having `blocklists for IP addresses that are known to be malicious` can also improve security and access controls 
can also help a company if a user does manage to get breached by limiting what they can even access in the first place.