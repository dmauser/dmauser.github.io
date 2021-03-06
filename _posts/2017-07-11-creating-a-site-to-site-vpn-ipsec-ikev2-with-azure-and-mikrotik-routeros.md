---
layout: post
title: Creating a Site-to-Site VPN (IPSec IKEv2) with Azure and MikroTik (RouterOS)
date: 2017-07-11 22:06
author: dmauser@hotmail.com
comments: true
categories: [Azure, Azure, IPSec, IPSec, VPN, VPN]
---
<span style="font-size: 12pt"><strong>Authors</strong>: <a href="https://twitter.com/DapiElMago">Daniel Pires</a> and <a href="https://twitter.com/DanMauser">Daniel Mauser</a>
</span>
<h3>Introduction</h3>
In this article, we are going to show you how to setup a IPSec Site-to-Site VPN between Azure and On-premises location by using MikroTik Router. Another blog post has been published few years ago about the same subject <a href="https://blogs.technet.microsoft.com/rharper/2012/11/14/creating-a-site-to-site-vpn-with-windows-azure-and-mikrotik-routeros/">Creating a site-to-site VPN with Windows Azure and MikroTik ( RouterOS )</a>. However, we have some major updates in this article. First, we are going to setup Site-to-Site VPN using Azure Resource Manager Portal (http://portal.azure.com), while original article uses Classic Azure Portal. Second, VPN Gateway in this blog post is Route-Based which will leverage IKE version 2 (IKEv2) compared with the Policy-Based Gateway on first article leveraging IKE version 1(IKEv1). If you are not familiar with the terminology of IPSec Parameters, in particular IKEv2, please take a look on the following documentation <a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-devices">About VPN devices and IPsec/IKE parameters for Site-to-Site VPN Gateway connections</a>.
<h3>Scenario</h3>
Below we have a diagram of the scenario covered in this step-by-step.

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK1.png" alt="" />

Relevant information on the diagram above necessary to configure the Site-to-Site VPN.

<strong>Azure Side:
</strong>
<ul>
 	<li>VNET Subnet: 10.4.0.0/16</li>
 	<li>Public IP of the Azure VPN Gateway: 13.85.83.XX</li>
</ul>
<strong>On-Premises Side:
</strong>
<ul>
 	<li>Subnet: 192.168.88.0/24</li>
 	<li>Public IP of On-Prem Gateway: 47.187.118.YY</li>
</ul>
<h3>Azure: Configuring Route-Based IPSec Site-to-Site VPN</h3>
This section we will go over step-by-step on configuring Site-to-Site VPN on the Azure side. The steps demonstrated here are the same in the official documentation: <a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal">Create a Site-to-Site connection in the Azure portal</a>. So, we are not going to cover specific step by step on how to get to the screens, you can use the official documentation as reference. Also, If you are already familiar with those steps feel free to jump right the way on the session below: <strong>MikroTik (On-Premises) Configuring IPSec (IKEv2) Site-to-Site VPN</strong>.

1. Create a virtual network
<p style="margin-left: 18pt"><img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK2.png" alt="" /></p>
2. Specify a DNS server (Optional for this and not necessary for this demonstration to work)

3. Create the gateway subnet:

a. Select Gateway Subnet
<p style="margin-left: 36pt"><img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK3.png" alt="" /></p>
b. Add Gateway subnet. In this case I will use the final 255 network inside 10.4.0.0/16 to create 32 addresses allocated to VPN Gateways and subnet is: 10.4.255.0.27
<p style="margin-left: 36pt"><img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK4.png" alt="" /></p>
4. Create the Virtual Network Gateway. It is important here to highlight we are going to use VPN Type: Route-Based Also for your lab purposes you can use SKU Basic, for production workloads it is recommend at least Standard SKU. More information about VPN Gateway sizes consult: <a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings">Gateway SKUs</a>.
<p style="margin-left: 18pt">a) Creating the Virtual Network Gateway named VNET1GW</p>
<p style="margin-left: 36pt"><img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK5.png" alt="" /></p>
<p style="margin-left: 18pt">b) After you create Virtual Network Gateway you can see the status as well as the Public IP that is going to be used:</p>
<p style="margin-left: 18pt"><img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK6.png" alt="" /></p>
5. Create the local network gateway which requires you specify Public IP of your VPN Device (47.187.117.YY) as well as the On-premises Subnet(s) (192.168.88.0/24).

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK7.png" alt="" />

6. Configure your VPN device – See section: <strong>MikroTik (On-Premises) Configuring IPSec (IKEv2) Site-to-Site VPN</strong>.

7. Create the VPN connection
<p style="margin-left: 18pt"><img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK8.png" alt="" /></p>
8. Verify the VPN connection

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK9.png" alt="" />
<h3>MikroTik (On-Premises) Configuring IPSec (IKEv2) Site-to-Site VPN</h3>
MikroTik RouterOS has several models and there are very affordable devices models that you can use also to play and learn how to configure Site-to-Site VPN with Azure.

<em>DISCLAIMER: Although we demonstrate Mikrotik in this article, it is important to mention Microsoft does not support the device configuration directly. In case you have issue, please contact device manufacturer for additional support and configuration instructions.
</em>

One important point to highlight is IKEv2 has been introduced on release 6.38. Therefore, make sure you have a compatible version to be able to proceed with the configuration demonstrated in this article which we used: RouterBOARD 750 and software version: RouterOS 6.39.

In this tutorial <strong>Winbox</strong> management utility has been used to perform MikroTik configuration and here are the necessary steps to configure MikroTik correctly:
<ol>
 	<li>
<div>Add IPSec Policy by Selecting on Menu <strong>IP</strong> and <strong>IPSec </strong>– On Policies tab click <strong>+ (plus) sign</strong> to add a New Policy. On General tab add both subnets (Source: On-Prem and Destination: Azure) as shown:</div>
<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK10.png" alt="" /></li>
 	<li>
<div>On the same screen but <strong>Action</strong> Tab – Select Tunnel and specify On-Prem Source Public IP and Destination Azure Gateway Public IP which can be obtained after you Create Virtual Network Gateway (See: <strong>Azure S2S VPN section – Step 4b)
</strong></div>
<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK11.png" alt="" /></li>
 	<li>
<div>On <strong>Peers</strong> Tab – Click + (Plus) and add a new IPSec Peer. In IPSec terminology we are working on IKE Phase 1 (Main Mode) on this configuration tab. Here we need Azure Gateway Public IP, specify Pre-Shared Key which can be specified on <strong>Part I – Step 7 (Create the VPN connection).</strong></div>
<strong>Note:</strong> If you MikroTik does not show IKEv2 make sure you have at least RouteOS release: 6.38. Before that release only IKEv1 is available.

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK12.png" alt="" /></li>
 	<li>
<div>On the same screen, go to <strong>Advance</strong> Tab and make adjustment on Lifetime to 8h = 28,800 seconds based on Azure official documentation <a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-devices">IPSec/IKE parameters</a> - SA (security association) for IKE Phase 1 (Main Mode).</div>
<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK13.png" alt="" /></li>
 	<li>
<div>On <strong>Encryption</strong> Tab, you can use the default which are supported by Azure or make adjustment for a stronger Hash and Encryption (See details here: <a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-devices">IPSec/IKE parameters</a>) . For this article the following have been selected:</div>
<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK14.png" alt="" /></li>
 	<li>
<div>Now, let's move on to IKE Phase 2 (Quick Mode) which is represented in MikroTik by <strong>Proposals. </strong>For this one you can either create a new one (+ sign) or change the default one. In case you create a new, make sure to change the Step 2 (IP Sec Policy) and Action Tab and select the appropriate Proposal. For this article, we will change the default IPSec Proposal which the following have been selected based on official Azure information for IKE Phase 2 in <a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-devices">IPSec/IKE parameters</a>:</div>
<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK15.png" alt="" /></li>
 	<li>
<div>The last step to make sure VPN will route correctly between On-Prem and Azure is to configure a NAT Rule. This is done by going <strong>IP</strong> and select <strong>Firewall</strong> – Select <strong>NAT</strong> tab</div>
<ol>
 	<li>Add Chain as <strong>srcnat </strong>and both subnets (On-Prem and Azure Subnet).</li>
</ol>
<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK16.png" alt="" />
<ol>
 	<li>On Action tab select <strong>accept</strong></li>
</ol>
<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK17.png" alt="" /></li>
</ol>
<h3>Validating the IPSec Tunnel</h3>
<strong>Ping between two computers in each side. </strong>In the right side On-Prem computer (192.168.88.17) correctly pinging Azure VM (10.4.0.4) and the other way around works fine too.

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK18.png" alt="" />

In both sides we see TTL of 126 which corresponds to two hops (both Gateways) getting decremented. Default TTL of Windows machines is 128.

<strong>Note:</strong> By default ICMP is disabled. Make sure you allow ICMP by running the following PowerShell command:
<span style="font-family: Lucida Console;font-size: 9pt"><strong><span style="color: blue">Set-NetfirewallRule </span><span style="color: navy">-Name </span><span style="color: blueviolet">FPS-ICMP4-ERQ-In </span><span style="color: navy">-Enable </span></strong><span style="color: blueviolet"><strong>True</strong>
</span></span>

<strong>On Azure Side
</strong>

On Azure Portal you can validate brand new tunnel created as showed on item 8. Verify the VPN connection above. That can be also validated by PowerShell by using command: <strong>Get-AzureRmVirtualNetworkGatewayConnection -Name From-Azure-to-Mikrotik -ResourceGroupName S2SVPNDemo</strong>

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK19.png" alt="" /><strong>
</strong>

<strong>On MikroTik Side
</strong>There are multiple ways to validate the IPSec VPN connection to Azure on MikroTik. Here are some ways:
1. IPSec – <strong>Policies</strong> tab. It shows if the IKE Phase 2 is working correctly.

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK20.png" alt="" />
2.<strong> Remote Peers</strong> tab. This shows if IKE Phase 1 (Main mode) is working correctly.

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK21.png" alt="" />
3.<strong> Installed SAs</strong> tab shows current Security Associations:

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK22.png" alt="" />
<h3>IPSec Troubleshooting</h3>
If something does not work for some reason during your configuration, you can do a troubleshooting to determine what is going on. MikroTik provides a good interface for logging and troubleshooting IPSec in case you want to get more detailed information on what is going on. Events can be visualized in Log Menu but to ensure you can get IPSec events exposed you need to make a simple change in Logging configuration (System – Logging) and add the IPSec as a Topic:

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK23.png" alt="" />

After you add this new Logging rule you have will see the following detailed IPSec events:

<img src="https://msdnshared.blob.core.windows.net/media/2017/07/071217_0229_CreatingaIK24.png" alt="" />

<h3><strong>Extra Tip: A Note about TCP MSS Clamp</strong></h3>
Based on a reader feedback. It is necessary to setup TCP Clamp because Azure has a reduced MTU. See notes here: <https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec>

Thefore, ensure you have your Mikrotik also to change the MSS to 1350 as suggested by official documentation from Microsoft.

ip firewall mangle add place-before=0 action=change-mss new-mss=1350 dst-address=10.4.0.0/16 chain=forward protocol=tcp tcp-flags=syn

<h3><strong>Conclusion
</strong></h3>
In this article we demonstrated how to setup a IPSec Site-to-Site VPN using IKEv2 (Route-Based) between Azure and MikroTik RouterBoard. These instructions also may help you to setup any IPSec device which is compatible with Azure VPN Gateway settings. I hope you liked the information shared here and please let us know below in the comments if you have additional questions. I would like to make a special thanks to Azure Support Escalation Engineer Daniel Pires who co-author this article with me. Thanks!
