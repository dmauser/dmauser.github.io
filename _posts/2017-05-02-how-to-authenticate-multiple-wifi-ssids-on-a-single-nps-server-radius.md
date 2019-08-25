---
layout: post
title: How to authenticate multiple WIFI SSIDs on a single NPS server (RADIUS)
date: 2017-05-02 09:52
author: dmauser@hotmail.com
comments: true
categories: [HowTo, NPS, OnPrem, RADIUS, Wireless]
---
<span style="font-size: 12pt;text-decoration: underline"><strong>Introduction
</strong></span>

Recently I worked with customer on interesting scenarios where they needed they were unable to make necessary restriction when using multiple WIFI Networks. They created WIFI Network devices such as Computer, Tablets and Mobile Phones. That was necessary because which network provided a different level of access. The goal was to ensure all WIFI networks (SSIDs) can be handled by a single NPS Server and users cannot use their credentials to access other WIFI SSID if they are not authorized. However, after creating a few Network Policy Rules, the first side effect was when a user accessed SSID, for example for SSID dedicated to mobiles, user was getting authenticated in another rule dedicated SSID for computers.

<span style="font-size: 12pt;text-decoration: underline"><strong>Scenario
</strong></span>

For this scenario, the following Network Policy Rules and respective specific Groups show below:
<ul>
 	<li><strong>Rule 1: Wireless-Computers [SSID: CTCORP]
</strong>- NAS Port Type = Wireless - Other OR Wireless - IEE 802.11
- Windows Groups = CONTOSO\WIFI-Corp-Users</li>
 	<li><strong>Rule 2: Wireless-Mobiles [SSID: CTMOBILE] </strong>
- NAS Port Type = Wireless - Other OR Wireless - IEE 802.11
- Windows Groups = CONTOSO\WIFI-Mobiles</li>
 	<li><strong>Rule 3: Wireless-Tablets [SSID: CTTABLETS]</strong>
- NAS Port Type = Wireless - Other OR Wireless - IEE 802.11
- Windows Groups = CONTOSO\WIFI-Tablets</li>
</ul>
Here is a view of the same rules above inside the NPS interface:

<img src="https://msdnshared.blob.core.windows.net/media/2017/05/050217_1451_Howtoauthen1.png" alt="" />

The diagram below shows how the policies should work. One of the requirements is to ensure when I user does not belong to a group, she or he should not be authorized to use the respective SSID. In this case, we can do more granular control of which type of devices can access the network with different restrictions. For example, for tables accessing CTTABLETS will have only access to Internet but no access internal resources, while for computers accessing WIFI SSID CTCORP they should have full access to the network and Internet.

<img src="https://msdnshared.blob.core.windows.net/media/2017/05/050217_1451_Howtoauthen2.png" alt="" />

The real problem starts when user belongs to two or more groups. For example, user John Smith is authorized to access WIFI from all types of devices, which means he belongs to all groups listed above. However, when using his computer, we need to make sure he authenticates on WIFI SSID CTCORP, when using his Smartphone, he needs to access WIFI SSID CTMOBILE and when he uses his Tablet he needs to use WIFI SSD CTTABLETS.

In case we John Smith is removed from group CONTOSO\WIFI-Mobiles he will still have access to the SSID CTMOBILE. Why? If you see the list of rules Rule: Wireless-Computers [SSID: CTCORP] has process order 1. Which means he will be authenticated by that Rule.

<span style="font-size: 12pt;text-decoration: underline"><strong>Resolution
</strong></span>The Solution for this scenario is to add a <span style="text-decoration: underline">Condition</span> inside the Network Policy and specify the <strong>Called Station ID </strong>which presents the WIFI Access Point MAC Address plus SSID.<strong>
</strong>This information can be easily extracted from NPS Event logs (Event Viewer – Custom Views – Server Roles – Network Policy and Access Services). When user is using a specify SSID that information is specified on <strong>Called Station ID</strong> populated as highlighted below.<strong>
</strong>

<span style="font-size: 9pt"><em>Source: Microsoft-Windows-Security-Auditing
Event ID: 6278
Task Category: Network Policy Server
Level: Information
Keywords: Audit Success
Description: Network Policy Server granted full access to a user because the host met the defined health policy.
Security ID: CONTOSO\JohnSmith
Account Name: CONTOSO\JohnSmith
Account Domain: Domain
Fully Qualified Account Name: CONTOSO\JohnSmith   </em></span>

<span style="background-color: yellow"><strong>Called Station Identifier: </strong>00-1C-C5-01-52-00: <strong>CTMOBILE </strong></span><strong>
</strong>

<span style="font-size: 9pt"><em>Calling Station Identifier: 25-E6-8C-24-E3-11
NAS: NAS IPv4 Address: 10.1.1.210
NAS IPv6 Address: -
NAS Identifier: 3Com NAS
Port-Type: Wireless - IEEE 802.11
RADIUS Client: Client Friendly Name: WIFIAccessPoint
Client IP Address: 10.1.1.210
Authentication Details: Connection Request Policy
Name: Secure Wireless Connections Network Policy
Name: 802.1X-Wireless-MOBILES [CORPv3]
Authentication Provider: Windows
</em></span>

Note: While creating the condition in Network Policy do not make confusion between <span style="text-decoration: underline">Called Station Identifier</span> with <span style="text-decoration: underline">Calling Station Identifier</span> which presents real computer's MAC address.

In summary, here are the action to do in each one of the Network Policy Rile, where you will specify the respective SSID as shown:<strong>
</strong>
<ul>
 	<li><strong>Rule 1 Wireless-Computers [SSID: CTCORP]
</strong>Called Station ID=<strong> CTCORP</strong></li>
 	<li><strong>Rule 2 Wireless-Mobiles [SSID: CTMOBILE] </strong>
Called Station ID=<strong> CTMOBILE</strong></li>
 	<li><strong>Rule 3 Wireless-Tablets [SSID: CTTABLETS]</strong>
Called Station ID=<strong> CTTABLETS</strong></li>
</ul>
Here is an example on how is done via GUI in five and self-explanatory simple steps :

<img src="https://msdnshared.blob.core.windows.net/media/2017/05/050217_1451_Howtoauthen3.png" alt="" />

It is important to note that you just need to add the SSID name as is and it will be searched in the field as string in any position. You can play with regular expression also well to adequate with your needs. You can leverage this documentation in TechNet: <a href="https://technet.microsoft.com/en-us/library/cc755272(v=ws.10).aspx">Using Regular Expressions in NPS</a>

<span style="font-size: 12pt;text-decoration: underline"><strong>Conclusion
</strong></span>

In this article, we demonstrated how to allow a single user who belongs which needs access multiple WIFI Networks (SSID's) while using a single Network Policy Server (NPS) to perform the authentication correctly on its respective rule matching the SSID by using Called Station ID. I hope this help you to implement this kind of scenario on your network and let us know your thoughts or questions in the comments below.
