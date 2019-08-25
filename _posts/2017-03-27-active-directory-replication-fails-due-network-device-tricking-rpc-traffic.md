---
layout: post
title: Active Directory Replication fails due network device tricking RPC traffic
date: 2017-03-27 17:40
author: dmauser@hotmail.com
comments: true
categories: [3rd party device, Active Directory Replication, Network Analysis, Network Analysis, OnPrem, RPC Error]
---
Hello, here is <a href="https://twitter.com/danmauser">Daniel Mauser</a> from Windows Networking Support team. Recently I worked in a support case where customer complained about Active Directory replication issues, and the underline issue was caused by a device in the middle, which was doing some kind weird behavior with TCP/IP.

In this scenario, basic test connectivity by using tools such as <a href="https://technet.microsoft.com/en-us/sysinternals/psping.aspx">psping</a> or telnet over primary AD TCP port 135 (RPC) showed port was correctly open. However, things start to get uncommon when TCP/IP communication is initiated. I will walk you on the troubleshooting process taken and the outcome to discover the root cause of the issue.
<h4><span style="text-decoration: underline"><strong>Scenario:</strong></span></h4>
This is very simple scenario where we have the following relevant information.
<ul>
 	<li>Involved domain controllers:
<ul>
 	<li>DC1 has NIC configured with IP 10.1.1.10/24;</li>
 	<li>DC4 has NIC configured with IP 10.4.1.40/24;
As you can note both DCs are in different subnets.</li>
</ul>
</li>
 	<li>Network Captures taken on both sides at the same time (that is core to understand what is going on).</li>
 	<li>DC04 initiates the communication to DC01.</li>
</ul>
<h4><span style="text-decoration: underline"><strong>Data Collection:</strong></span></h4>
You can leverage<strong> netsh trace</strong> to start the network captures. This command is built-in starting on Windows 7 and Server 2008 R2. Also, for this troubleshooting ,we do not need to capture the whole TCP payload but only first 512 bytes (<em>packettruncatebytes=512</em>) which brings some advantages on reducing the size of the capture file, as well as reducing the chances of packet drops when servers, in production, are very busy talking to multiple clients and other services. This article gives you a good insight on that piece: <a href="https://social.technet.microsoft.com/wiki/contents/articles/6192.how-to-enable-a-circular-network-capture-with-nmcap-or-netsh.aspx">How to Enable a Circular Network Capture with Nmcap or Netsh</a>

<em>netsh trace start capture=yes packettruncatebytes=512 tracefile=c:\%computername%_nettrace.etl maxsize=200 filemode=circular overwrite=yes report=no</em>

After we reproduce the issue we stop the trace above via <strong>netsh trace stop</strong> on both DC01 and DC04.
<h4><span style="text-decoration: underline"><strong>Analysis</strong></span></h4>
ETL file produced by the command above can be opened either in Network Monitor 3.4 or Message Analyzer (Recommended). You can also convert this ETL file to CAP and review it on Wireshark. Check this article from my PFE friend Yong Rhee on how to do that: <a href="https://blogs.technet.microsoft.com/yongrhee/2013/08/16/so-you-want-to-use-wireshark-to-read-the-netsh-trace-output-etl/">So you want to use Wireshark to read the netsh trace output .etl?</a>

Here is the analysis of the network capture taken for both sides, see comments column for more details.

<b><span style="font-family: 'Calibri',sans-serif">Review of Capture Taken on </span></b><b><span style="font-size: 11.0pt;font-family: 'Calibri',sans-serif">DC04</span></b>
<div class="WordSection1">
<table width="98%" class="MsoNormalTable" style="width: 98.06%;border-collapse: collapse;border: none" border="1" cellspacing="0" cellpadding="0">
<thead>
<tr>
<td width="4%" valign="top" style="width: 4.6%;border: solid windowtext 1.0pt;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Frame</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Src</span></b></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Dst</span></b></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="4%" valign="top" style="width: 4.14%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Proto</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="46%" valign="top" style="width: 46.18%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Description</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.2%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 0in 0in 0in 0in">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Comments</span></b></p>
</td>
</tr>
</thead>
<tbody>
<tr>
<td width="4%" valign="top" style="width: 4.6%;border: solid windowtext 1.0pt;border-top: none;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="right" style="margin: 0in;margin-bottom: .0001pt;text-align: right;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">133</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="4%" valign="top" style="width: 4.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.18%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP:Flags</span></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">=......S., <span class="SpellE">SrcPort</span>=57865, <span class="SpellE">DstPort</span>=DCE endpoint resolution(135), <span class="SpellE">PayloadLen</span>=0, <span class="SpellE">Seq</span>=2468100402,Ack=0, Win=8192 ( Negotiating scale factor 0x8 ) = 8192</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.4.1.40, <span class="SpellE">Dest</span> = 10.1.1.10, Next Protocol = TCP,
Packet ID = 16375,     <b><span style="background: lime">Identification: 16375 (0x3FF7)</span></b></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TimeToLive</span></b></span><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">: 128 (0x80)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 0in 0in 0in 0in">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"><span> </span>DC04 initiates TCP 3-Way Handshake with a SYN. On the same conversation captured on DC01 we see this packet arrived there on Frame </span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">1998 (table below)</span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">. We see <b><span style="background: lime">IP identification
16375</span> </b>on both sides which means packet left DC04 and the same
arrived on DC01.<span>      </span></span></p>
</td>
</tr>
<tr>
<td width="4%" valign="top" style="width: 4.6%;border: solid windowtext 1.0pt;border-top: none;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="right" style="margin: 0in;margin-bottom: .0001pt;text-align: right;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">134</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="4%" valign="top" style="width: 4.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.18%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP:Flags</span></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">=...A..S., <span class="SpellE">SrcPort</span>=DCE endpoint resolution(135), <span class="SpellE">DstPort</span>=57865, <span class="SpellE">PayloadLen</span>=0, <span class="SpellE"><b><span style="background: red">Seq</span></b></span><b><span style="background: red">=1441552686,</span> Ack=2468100403, Win=0</b> ( Negotiated scale factor 0x8 ) = 0</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.1.1.10, <span class="SpellE">Dest</span> = 10.4.1.40, Next Protocol = TCP,
Packet ID = 65534, Total IP Length = 52</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: yellow">Identification:
65534 (0xFFFE)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: yellow">TimeToLive</span></b></span><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: yellow">: 54 (0x36) </span></b><b><span style="font-size: 10.0pt;line-height: 105%;font-family: Wingdings;color: black">è </span></b><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: yellow">ROOT CAUSE OF THE ISSUE</span></b><span style="font-size: 10.0pt;line-height: 105%;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 0in 0in 0in 0in">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Here is when issue starts.</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"> Something, which is not DC01, replies that TCP SYN flag with <b>TCP Window Size=0,</b> which means no buffer available to process TCP Request. How do we know this is not really DC01 but a device talking on behalf DC01?</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">The evidences are <b><span style="background: yellow">IP Identification 65534</span></b><span style="background: yellow"> and <b>TTL which is 54</b></span><b>.</b>
We will see the right packet from DC01 is on frame 136 (TCP Flags ACK and SYN) and has <b><span style="background: lime">TTL 121</span></b>. </span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"> </span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><u><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Note</span></u></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">  Windows default TTL is 128 and decrements on each hop (router)<u>. In this frame, we have <b><span style="background: yellow">TTL=54</span></b> usually is attributed for a
non-Windows device.</u><b></b></span></p>
</td>
</tr>
<tr>
<td width="4%" valign="top" style="width: 4.6%;border: solid windowtext 1.0pt;border-top: none;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="right" style="margin: 0in;margin-bottom: .0001pt;text-align: right;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">135</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="4%" valign="top" style="width: 4.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.18%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP:Flags</span></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">=...A...., <span class="SpellE">SrcPort</span>=57865, <span class="SpellE">DstPort</span>=DCE endpoint resolution(135), <span class="SpellE">PayloadLen</span>=0, <span class="SpellE">Seq</span>=2468100403, <b><span style="background: red">Ack=1441552687</span>,</b> Win=256 (scale factor 0x8) = 65536</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.4.1.40, <span class="SpellE">Dest</span> = 10.1.1.10, Next Protocol = TCP,
Packet ID = 16376,</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: lime">Identification:
16376 (0x3FF8</span></b><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">)</span></b></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TimeToLive</span></b></span><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">: 128 (0x80)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 0in 0in 0in 0in">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04 sends ACK which also arrives fine on frame 2000 (see table below). Compare both IP Identification numbers which we have: <b><span style="background: lime">16376</span>. </b>The issue here is this Acknowledges the SEQ + 1 of invalid ACK SYN from frame 134.</span></p>
</td>
</tr>
<tr>
<td width="4%" valign="top" style="width: 4.6%;border: solid windowtext 1.0pt;border-top: none;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="right" style="margin: 0in;margin-bottom: .0001pt;text-align: right;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">136</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="4%" valign="top" style="width: 4.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.18%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP:Flags</span></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">=...A..S., <span class="SpellE">SrcPort</span>=DCE endpoint resolution(135), <span class="SpellE">DstPort</span>=57865, <span class="SpellE">PayloadLen</span>=0, <span class="SpellE"><b><span style="background: fuchsia">Seq</span></b></span><b><span style="background: fuchsia">=2228284491</span>, Ack=2468100403,</b> Win=8192 ( Negotiated scale factor 0x8 ) = 2097152</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.1.1.10, <span class="SpellE">Dest</span> = 10.4.1.40, Next Protocol = TCP, Packet ID = 20767,     <span class="SpellE">TotalLength</span>: 52 (0x34)</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Identification: 20767 (0x511F)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: lime">TimeToLive</span></b></span><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: lime">: 121 (0x79)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 0in 0in 0in 0in">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">This is the real ACK SYN from DC01. Thepoint here is DC04 send SEQ <span class="SpellE"><b><span style="background: fuchsia">Seq</span></b></span><b><span style="background: fuchsia">=2228284491 </span></b>but in next frame we see DC01 stick on ACK of the spoofed frame 134 which has <span class="SpellE"><b><span style="background: red">Seq</span></b></span><b><span style="background: red">=1441552686.</span></b><span>              </span></span></p>
</td>
</tr>
<tr>
<td width="4%" valign="top" style="width: 4.6%;border: solid windowtext 1.0pt;border-top: none;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="right" style="margin: 0in;margin-bottom: .0001pt;text-align: right;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">137</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="4%" valign="top" style="width: 4.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.18%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: yellow">TCP:[Dup Ack #135]</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Flags=...A....,<span class="SpellE">SrcPort</span>=57865, <span class="SpellE">DstPort</span>=DCE endpoint resolution(135), <span class="SpellE">PayloadLen</span>=0, <span class="SpellE">Seq</span>=2468100403, <b><span style="background: red">Ack=1441552687</span></b>, Win=256 (scale factor 0x8) =
65536</span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.4.1.40, <span class="SpellE">Dest</span> = 10.1.1.10, Next Protocol = TCP,
Packet ID = 16377, Total IP Length = 40</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Identification: 16377 (0x3FF9)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"><span class="SpellE">TimeToLive</span>: 128 (0x80)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 0in 0in 0in 0in">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Here is DUP ACK which basically replies the ACK for invalid frame 134 <b><span style="background: red">Ack=1441552687</span>.
</b></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"> </span></b></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Note that ACK increments in 1 on TCP 3-way  handshake (see: </span></b><span style="font-size: 10.0pt;line-height: 105%"><a href="https://wiki.wireshark.org/TCP_3_way_handshaking"><span style="line-height: 105%;font-family: 'Calibri',sans-serif">https://wiki.wireshark.org/TCP_3_way_handshaking</span></a></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">) </span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
</tr>
<tr>
<td width="4%" valign="top" style="width: 4.6%;border: solid windowtext 1.0pt;border-top: none;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="right" style="margin: 0in;margin-bottom: .0001pt;text-align: right;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">138</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="4%" valign="top" style="width: 4.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.18%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: yellow">TCP:Flags</span></b></span><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: yellow">=.....R..,</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"> <span class="SpellE">SrcPort</span>=DCE endpoint resolution(135), <span class="SpellE">DstPort</span>=57865, <span class="SpellE">PayloadLen</span>=0, <span class="SpellE">Seq</span>=1441552687, Ack=1441552687, Win=0 (scale factor 0x8) = 0</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.1.1.10, <span class="SpellE">Dest</span> = 10.4.1.40, Next Protocol = TCP, Packet ID = 20768, Total IP Length = 40</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Identification: 20768 (0x5120)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"><span class="SpellE">TimeToLive</span>: 121 (0x79)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 0in 0in 0in 0in">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01 sends a <b><span style="background: yellow">RESET </span></b>because that ACK does not match with the SEQ of real ACK SYN of frame 136.
So, DC01 expects DC04 to ACK (Acknowledge) Sequence:  <span class="SpellE"><b><span style="background: fuchsia">Seq</span></b></span><b><span style="background: fuchsia">=2228284491</span></b><span>             </span></span></p>
</td>
</tr>
</tbody>
</table>
<p style="margin: 0in 0in .0001pt 18.15pt"><span style="font-size: 11.0pt;font-family: 'Calibri',sans-serif"></span><span style="font-size: 11.0pt;font-family: 'Calibri',sans-serif"> </span></p>
<p style="margin: 0in;margin-bottom: .0001pt"><b><span style="font-family: 'Calibri',sans-serif">Review of Capture Taken on </span></b><b><span style="font-size: 11.0pt;font-family: 'Calibri',sans-serif">DC01</span></b><span style="font-size: 11.0pt;font-family: 'Calibri',sans-serif"> </span></p>

<table width="98%" class="MsoNormalTable" style="width: 98.06%;border-collapse: collapse;border: none" border="1" cellspacing="0" cellpadding="0">
<thead>
<tr>
<td width="4%" valign="top" style="width: 4.62%;border: solid windowtext 1.0pt;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Frame</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Src</span></b></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Dest</span></b></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="4%" valign="top" style="width: 4.16%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Proto</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="46%" valign="top" style="width: 46.2%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Description</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.14%;border: solid windowtext 1.0pt;border-left: none;background: black;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p align="center" style="margin: 0in;margin-bottom: .0001pt;text-align: center;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white">Comments</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: white"></span></p>
</td>
</tr>
</thead>
<tbody>
<tr>
<td width="4%" valign="top" style="width: 4.62%;border: solid windowtext 1.0pt;border-top: none;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">1998</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="4%" valign="top" style="width: 4.16%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP:Flags</span></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">=......S., <span class="SpellE">SrcPort</span>=57865, <span class="SpellE">DstPort</span>=DCE endpoint resolution(135), <span class="SpellE">PayloadLen</span>=0, <span class="SpellE">Seq</span>=2468100402, Ack=0, Win=8192 ( Negotiating scale
factor 0x8 ) = 8192</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.4.1.40, <span class="SpellE">Dest</span> = 10.1.1.10, Next Protocol = TCP, Packet ID = 16375, <b><span style="background: lime">Identification: 16375 (0x3FF7)</span></b></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TimeToLive</span></b></span><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">: 122 (0x7A)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">This is valid the frame from DC04 which is the same as frame 133 from the table above.
Identification is the same and TTL decrements due network hops between both DCs.</span></p>
</td>
</tr>
<tr>
<td width="4%" valign="top" style="width: 4.62%;border: solid windowtext 1.0pt;border-top: none;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">1999</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="4%" valign="top" style="width: 4.16%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP:Flags</span></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">=...A..S., <span class="SpellE">SrcPort</span>=DCE endpoint resolution(135), <span class="SpellE">DstPort</span>=57865, <span class="SpellE">PayloadLen</span>=0, <span class="SpellE">Seq</span>=2228284491, Ack=2468100403, Win=8192 ( Negotiated scale factor 0x8 ) = 2097152</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.1.1.10, <span class="SpellE">Dest</span> =10.4.1.40, Next Protocol = TCP, Packet ID = 20767, Total IP Length = 52</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Identification: 20767 (0x511F)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TimeToLive</span></b></span><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">: 128 (0x80)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">This is the real ACK SYN which is the same frame as frame <b>136 </b>seen on Network capture taken on DC04 (See the table above)</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"> </span></p>
</td>
</tr>
<tr>
<td width="4%" valign="top" style="width: 4.62%;border: solid windowtext 1.0pt;border-top: none;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">2000</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="4%" valign="top" style="width: 4.16%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP:Flags</span></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">=...A...., <span class="SpellE">SrcPort</span>=57865, <span class="SpellE">DstPort</span>=DCE endpoint resolution(135), <span class="SpellE">PayloadLen</span>=0, <span class="SpellE">Seq</span>=2468100403, <b><span style="background: red">Ack=1441552687</span></b>, Win=256 (scale factor 0x8) =
65536</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.4.1.40, <span class="SpellE">Dest</span> =10.1.1.10, Next Protocol = TCP, Packet ID = 16376, Total IP Length = 40</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: lime">Identification: 16376 (0x3FF8)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TimeToLive</span></b></span><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">: 122 (0x7A)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;background: #D9D9D9;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">This is the ACK from DC04, frame 135 but it contains the invalid ACK number <b><span style="background: red">Ack=1441552687</span></b>. Again, this is due the fact of an unknown device (Frame 134 of DC04 Table) has started the invalid conversation.</span></p>
</td>
</tr>
<tr>
<td width="4%" valign="top" style="width: 4.62%;border: solid windowtext 1.0pt;border-top: none;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">2001</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC01</span></p>
</td>
<td width="3%" valign="top" style="width: 3.94%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">DC04</span></p>
</td>
<td width="4%" valign="top" style="width: 4.16%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">TCP</span></p>
</td>
<td width="46%" valign="top" style="width: 46.2%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span class="SpellE"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: yellow">TCP:Flags</span></b></span><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black;background: yellow">=.....R..,</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"> <span class="SpellE">SrcPort</span>=DCE endpoint resolution(135), <span class="SpellE">DstPort</span>=57865, <span class="SpellE">PayloadLen</span>=0, <span class="SpellE">Seq</span>=1441552687, Ack=1441552687, Win=0 (scale factor 0x8) = 0</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">- Ipv4: <span class="SpellE">Src</span> = 10.1.1.10, <span class="SpellE">Dest</span> =10.4.1.40, Next Protocol = TCP, Packet ID = 20768, Total IP Length = 40</span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">Identification: 20768 (0x5120)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"><span class="SpellE">TimeToLive</span>: 128 (0x80)</span></b><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black"></span></p>
</td>
<td width="37%" valign="top" style="width: 37.14%;border-top: none;border-left: none;border-bottom: solid windowtext 1.0pt;border-right: solid windowtext 1.0pt;padding: 2.0pt 3.0pt 2.0pt 3.0pt">
<p style="margin: 0in;margin-bottom: .0001pt;line-height: 105%"><span style="font-size: 10.0pt;line-height: 105%;font-family: 'Calibri',sans-serif;color: black">This is TCP <strong><span style="background: yellow">RESET</span> </strong>sent from DC01 which does not recognize the ACK sent by DC04. Again, DC04 was misled by invalid ACK SYN sent by an unknown device in the middle of the communication.</span></p>
</td>
</tr>
</tbody>
</table>
<p style="margin: 0in;margin-bottom: .0001pt"><span style="font-size: 9.0pt;font-family: 'Calibri',sans-serif"> </span></p>
<p style="margin: 0in;margin-bottom: .0001pt">Let's review helpful points in thew review which are relevant when reviewing this kind of issue:</p>

</div>
<ol>
 	<li>Check on both captures for IP Identification numbers and see if they match on both side. We can see above that <strong>frame 134</strong> has an IP <strong>Identification number 65534</strong> that has been not sent by DC01.</li>
 	<li><strong>Check SEQ and ACK numbers</strong>. This initially looks hard but you get used to it. Practice makes perfect :-)</li>
 	<li>There's an evidence on <strong>frame 134</strong> another device is intercepting the communication. So, TTL in Windows is always 128. <strong>TTL on frame 134 is 54</strong> which tell us this is not a Windows machine. Also, TTL for right frame decrements from 128 leaving DC01 and arrives on DC04 with 121, which means about 7 hops or 7 routers in the middle decremented the TTL (We can also explorer this in another article).</li>
 	<li>TCP Zero Window is another important which tells us basically that device is asking Windows not to send traffic because we see TCP Zero Window on <strong>frame 134</strong>.</li>
</ol>
<h4><span style="text-decoration: underline"><strong>Conclusion</strong></span></h4>
The capture shows clearly that an unknown network device between DC01 and DC04 is causing the issue. The conversation is tricked due the fact of ACK SYN packet sent by DC01 the unknown device misleads DC04 to send invalid ACK numbers and the end results in abrupt end of communication (TCP Reset) sent by DC01 because it does not understand them. It is clear in this scenario that TCP Port 135 (RPC Endpoint Mapper) is not blocked and another device by reviewing the TTL.

As side note, reviewing the whole capture we figure out also that behavior only for port TCP 135 (RPC) and did not happen on other TCP ports for conversation between DC01 and DC04. It is very common security devices perform RPC inspection in the network.

I hope you enjoy this article and let us know in the comments what you think. Stay tuned for more troubleshooting articles like this.
Special thanks for Support Escalation Engineer <a href="https://twitter.com/DapiElMago">Daniel Pires</a> (another networking geek), for the insights on this article.
