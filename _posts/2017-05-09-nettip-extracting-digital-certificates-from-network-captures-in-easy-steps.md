---
layout: post
title: NetTIP - Extracting Digital Certificates from Network Captures in Easy Steps
date: 2017-05-09 10:54
author: dmauser@hotmail.com
comments: true
categories: [Azure, Certificates, NetTip, Network Analysis, Network Security, OnPrem]
---
<span style="font-size: 12pt;text-decoration: underline"><strong>Introduction
</strong></span>

Troubleshooting connectivity issues in scenarios involving SSL/TLS communications usually involve an important point to verify whether the certificates exchanged during that process (TLS Client/Server Hello) is the correct one and has the attributes or extensions in place such as EKU, SAN, and others.

In this article, we will show you how to extract digital certificate information by using two network excellent and well known analysis tools: Microsoft Message Analyzer and Wireshark. The good news is you can also get some hands on and review the same captures used this article. Just go to <a href="https://gallery.technet.microsoft.com/Network-Captures-Library-29061327">Network Capture Library</a> to have access to them.

<span style="text-decoration: underline"><strong>Side Note:</strong></span> Other captures will be added soon in the Network Capture Library with different scenario where you can download, review and learn from them.

The main goal of this article is more educational on teaching you how to extract information of digital certificates from a network capture. I would like to anticipate that if you search Internet you can find several ready tools and other methodologies to perform same action. However, most of those tools are focus on TLS for HTTP traffic (HTTPS) and we will explore here other scenarios you may face, for example, 802.1x that will explore in this article.

<span style="font-size: 12pt;text-decoration: underline"><strong>Exporting Web Server Certificate using Message Analyzer
</strong></span>

We are going to open capture named <a href="https://gallery.technet.microsoft.com/Network-Captures-Library-29061327"><strong>HTTPS-TLS-Certificate.cap</strong></a><strong>
</strong>

After opening the capture in Message Analyzer follow these steps:
<ol>
 	<li>Apply the following filters just to show frames where certificate information is <strong><em>*Summary contains "Certificate"</em></strong></li>
 	<li>Select the frame and start to expand each property on the package focusing on Certificate.</li>
 	<li>Select <strong>in x509_cert</strong>.</li>
 	<li>Automatically Message Data 1 window will highlight in the grey portion of the capture with the certificate to be exported.
Note: If Message Data 1 is not displayed just proceed to menu <span style="text-decoration: underline">Tools -&gt; Message Data – Message Data 1</span></li>
 	<li>Right-click in any gray area of Message Data 1 window as <strong><em>Save Selected Bytes As… </em></strong>as type any file name and just make sure the extension is .CER</li>
</ol>
<img alt="" src="https://msdnshared.blob.core.windows.net/media/2017/05/050917_1550_NetTIPExtra1.png" />
<ol>
 	<li>
<div>After successful get the file exported. Just go to the folder and double click in the file and you will be able to see it.</div>
<img alt="" src="https://msdnshared.blob.core.windows.net/media/2017/05/050917_1550_NetTIPExtra2.png" />

<strong>Note:</strong> In your review, you may get a warning message while opening the certificate because your system does not trust on it. So, this may be expected.</li>
</ol>
<span style="font-size: 12pt;text-decoration: underline"><strong>Exporting Client and RADIUS Server Ceritificates from EAP-TLS traffic (802.1x) using Wireshark
</strong></span>

We are going to open capture named <a href="https://gallery.technet.microsoft.com/Network-Captures-Library-29061327"><strong>EAP-TLS-Certificate.cap</strong></a>.<span style="font-size: 12pt;text-decoration: underline"><strong>
</strong></span>

The same process above you can perform with Wireshark. In this scenario, Client and RADIUS Server Certificate during the process of EAP-TLS authentication are going to be extracted:
<ol>
 	<li>Select frames where we have Certificate explicitly in the information. For RADIUS server certificate, select frame 17. Client certificate can be done by selecting frame 21.</li>
 	<li>Expand bottom part on Packet details window and select <strong>Certificate.</strong></li>
 	<li>Right Click and Export Packet Bytes (CTRL + H) and name it and add extension .<strong>CER</strong>.</li>
</ol>
<img alt="" src="https://msdnshared.blob.core.windows.net/media/2017/05/050917_1550_NetTIPExtra3.png" /><strong>
</strong>

<strong>*Side Note:</strong> You may be wondering why do we have source and destination shown as MAC address? Again this 802.1x and machine is authenticating to get an IP.
<ol>
 	<li>
<div>Repeat same steps above to export the client certificate and final output both exported <strong>.CER </strong>files are:</div>
<img alt="" src="https://msdnshared.blob.core.windows.net/media/2017/05/050917_1550_NetTIPExtra4.png" /></li>
</ol>
<span style="font-size: 12pt;text-decoration: underline"><strong>Wireshark Extra Bonus Tip:
</strong></span>

One cool very cool tip I would like to present is a way to expose the certificate names in Wireshark as a Column. That will come in handy loading a capture and digital certificate name will be exposed to the column:
<ol>
 	<li>Add a custom type column in via menu Edit – Preferences and add in Fields <strong>x509sat.printableString</strong> as shown in this picture:</li>
</ol>
<img alt="" src="https://msdnshared.blob.core.windows.net/media/2017/05/050917_1550_NetTIPExtra5.png" /><strong>
</strong>      2. Certificate information will be exposed to the Column with the Certification Authority (issuer) and certificate as shown below:

<img alt="" src="https://msdnshared.blob.core.windows.net/media/2017/05/050917_1550_NetTIPExtra6.png" /><strong>
</strong>

In the example above: <span style="text-decoration: underline">Contoso-EntCA</span> is the Issuer Certification Authority (CA), <span style="text-decoration: underline">WADC01.contoso.corp</span> is the certificate of NPS Server (RADIUS) and <span style="text-decoration: underline">WA-W10CLI2.contoso.corp</span> is the certificate from the client.<span style="text-decoration: underline"><strong>
</strong></span>

<span style="font-size: 12pt;text-decoration: underline"><strong>Conclusion
</strong></span>

This post described in easy steps on how to export digital certificates from network captures by using either Microsoft Message Analyzer or Wireshark. We will be posting in the next few days a troubleshooting case scenario that will explore and resolve an issue that will leverage digital certificate information. Stay Tune! Thanks for visiting us and I hope you learned something new today.
