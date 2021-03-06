---
layout: post
title: Basic PS script to perform a Network Capture (packet sniffing)
date: 2017-04-27 03:50
author: dmauser@hotmail.com
comments: true
categories: [Network Analysis, Network Capture, OnPrem, PowerShell]
---
<span style="font-family: 'Segoe UI',sans-serif;color: #333333"><span>Hello all</span>, here is <a href="https://twitter.com/danmauser">Daniel Mauser</a> again and today I'm </span><span style="font-family: 'Segoe UI',sans-serif;color: #333333">going to show you how you can leverage network capture traces using native PowerShell cmdlet. Before </span><span style="font-family: 'Segoe UI',sans-serif;color: #333333">that we need to reference you, just as quick recap, to a great article from Hey </span><span style="font-family: 'Segoe UI',sans-serif;color: #333333">Scripting Guy! where he shows how to get a network capture using PowerShell (<a href="https://blogs.technet.microsoft.com/heyscriptingguy/2015/10/12/packet-sniffing-with-powershell-getting-started/">Packet </a></span><span style="font-family: 'Segoe UI',sans-serif;color: #333333"><a href="https://blogs.technet.microsoft.com/heyscriptingguy/2015/10/12/packet-sniffing-with-powershell-getting-started/">Sniffing with PowerShell: Getting Started</a>). On this article he demonstrate </span><span style="font-family: 'Segoe UI',sans-serif;color: #333333">using relevant Network Provides such as Microsoft-Windows-TCPIP but the end </span><span style="font-family: 'Segoe UI',sans-serif;color: #333333">result capture it does not look like the same capture taken by <span class="SpellE"><b>netsh</b></span><b> trace start capture=yes</b>.</span>
<div class="WordSection1">
<p class="MsoNormal" style="line-height: normal"><b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">Is there any way to do it via PowerShell?</span></u></b></p>
<p class="MsoNormal" style="line-height: normal"><span style="font-family: 'Segoe UI',sans-serif;color: #333333"> The short answer is yes. We developed a very basic script demonstrating how to do that. The trick part is to get the right ETW provider which is: <b>Microsoft-Windows-NDIS-<span class="SpellE">PacketCapture, </span></b><span class="SpellE">more details to come. </span></span></p>
<p class="MsoNormal" style="line-height: normal"><span style="font-family: 'Segoe UI',sans-serif;color: #333333"><span class="SpellE">W</span></span><span style="font-family: 'Segoe UI',sans-serif;color: #333333">e will go over a step-by-step demonstrating how to save a network capture in ETL file including a bonus of adding a time stamp and maximum size of 512 MB circular:</span></p>
<p class="MsoNormal" style="line-height: normal"><b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">Define Timestamp variable</span></u></b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">
</span></u><span style="font-family: 'Segoe UI',sans-serif;color: #333333">This is going to be to append to the output ETL capture file.</span></p>
<p style="line-height: normal"><em><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333">PS C:\&gt; $timestamp = Get-Date -f yyyy-MM-dd_HH-mm-ss</span></em><span style="font-family: 'Segoe UI',sans-serif;color: #333333">
</span></p>
<p class="MsoNormal" style="line-height: normal"><b><span style="font-family: 'Segoe UI',sans-serif;color: #333333">Note:</span></b><span style="font-family: 'Segoe UI',sans-serif;color: #333333"> PS commands listed below work with Windows 8.1 / Windows Server 2012 R2 and earlier Windows versions.</span></p>
<p class="MsoNormal" style="line-height: normal"><b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">Create a new Session1
</span></u></b><span style="font-family: 'Segoe UI',sans-serif;color: #333333">Now let's define the new capture session adding computer name and timestamp to the ETL file being created</span></p>
<p class="MsoNormal" style="line-height: normal"><i><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333">PS C:\&gt; New-<span class="SpellE">NetEventSession</span> -Name Session1 -<span class="SpellE">LocalFilePath</span> c:\$env:computername-netcap-$timestamp.etl -<span class="SpellE">MaxFileSize</span> 512</span></i></p>
<p class="MsoNormal" style="line-height: normal"><i><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333">Name               : Session1
<span class="SpellE">CaptureMode</span>        : <span class="SpellE">SaveToFile</span>
<span class="SpellE">LocalFilePath</span>      : c:\W10LAB-netcap-2017-04-26_19-45-17.etl
<span class="SpellE">MaxFileSize</span>        : 512 MB
<span class="SpellE">TraceBufferSize</span>    : 0 KB
<span class="SpellE">MaxNumberOfBuffers</span> : 0
<span class="SpellE">SessionStatus</span>      : <span class="SpellE">NotRunning</span></span></i></p>
<p class="MsoNormal" style="line-height: normal"><b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">Adding provider </span></u></b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">
</span></u><span style="font-family: 'Segoe UI',sans-serif;color: #333333">In this case is necessary add the associated GUID to this provider "Microsoft-Windows-NDIS-<span class="SpellE">PacketCapture</span>" or use Add-NetEventpacketCaptureProvider:</span></p>
<p class="MsoNormal" style="line-height: normal"><i><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333">PS C:\&gt; <span class="pl-c1">Add-NetEventPacketCaptureProvider</span><span> </span><span class="pl-k">-</span><span>SessionName Session1 </span></span></i><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333"></span></p>
<p class="MsoNormal" style="line-height: normal"><b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">Starting a Network Capture Session
</span></u></b><span style="font-family: 'Segoe UI',sans-serif;color: #333333">Now it is time to start the network capture by running:
</span></p>
<p class="MsoNormal" style="line-height: normal"><em><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333">PS C:\&gt; Start-<span>NetEventSession -Name Session1</span></span></em></p>
<p class="MsoNormal" style="margin-bottom: .0001pt;line-height: normal"><b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">Check status of the capture</span></u></b><b><span style="font-family: 'Segoe UI',sans-serif;color: #333333">
</span></b><span style="font-family: 'Segoe UI',sans-serif;color: #333333">Ensure the capture is running the command below and check last output line named <em>SessionStatus</em>:</span></p>
<p class="MsoNormal" style="margin-bottom: .0001pt;line-height: normal"><i><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333">PS C:\&gt; Get-<span class="SpellE">NetEventSession</span></span></i></p>
<p class="MsoNormal" style="margin-bottom: .0001pt;line-height: normal"><i><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333">Name              : Session1
<span class="SpellE">CaptureMode</span>       : <span class="SpellE">SaveToFile</span>
<span class="SpellE">LocalFilePath</span>     : c:\W10LAB-netcap-2017-04-26_19-45-17.etl
<span class="SpellE">MaxFileSize</span>       : 512 MB
<span class="SpellE">TraceBufferSize</span>    : 64 KB
<span class="SpellE">MaxNumberOfBuffers</span> : 30
<strong><span class="SpellE">SessionStatus</span>      : Running</strong></span></i></p>
<p class="MsoNormal" style="line-height: normal"><b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">Stopping the Capture</span></u></b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">
</span></u><span style="font-family: 'Segoe UI',sans-serif;color: #333333">After sometime running your capture, you can stop the capture just run the following:
</span><i><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333">PS C:\&gt; Stop-<span class="SpellE">NetEventSession</span> -Name Session1</span></i><i><span style="font-family: 'Courier New';color: #333333"></span></i></p>
<p class="MsoNormal" style="line-height: normal"><b><u><span style="font-size: 12.0pt;font-family: 'Segoe UI',sans-serif;color: #333333">Remove the Session
</span></u></b><span style="font-family: 'Segoe UI',sans-serif;color: #333333">Now you can start over the whole thing by removing the session and making other customizations or if you need to start a new file with a new timestamp.<b>
</b></span><i><span style="font-size: 10.0pt;font-family: 'Courier New';color: #333333">PS C:\&gt; Remove-<span class="SpellE">NetEventSession</span> -Name session1 </span></i></p>
<p class="MsoNormal" style="line-height: normal"><span style="font-family: 'Segoe UI',sans-serif;color: #333333"><strong>Note:</strong> Script has been posted in this GitHub Repository (<a href="https://github.com/dmauser/PS-Network-Capture/blob/master/Basic-Net-Capture.ps1"><strong class="final-path">Basic-Net-Capture.ps1</strong></a>) for your reference. Some updates will be incorporated based on feedback.</span></p>
<b><u><span style="font-family: 'Segoe UI',sans-serif;color: #333333">Final Considerations
</span></u></b><span style="font-family: 'Segoe UI',sans-serif;color: #333333">We can re-use the same session by starting the capture again using <i>Start-<span class="SpellE">NetEventSession</span>-Name Session1 </i>but keep in mind we defined the timestamp of the output file on the <i>New-<span class="SpellE">NetEventSession</span>. </i>In order to create a new timestamp file, you need to remove Sesson1 and re-created it again. You can also figure out other ways to do that and feel free post in the comments below. I hope you learnt something new today.</span>

</div>
