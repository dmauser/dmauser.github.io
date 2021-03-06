---
title: Use AzureRMAppGWCert to list certificate details of Azure App Gateway
---
Authors: <a href="https://www.linkedin.com/in/damauser/">Daniel Mauser</a> and <a href="https://www.linkedin.com/in/victor-santana-6482227/">Victor Santana</a>
<h4><strong>Introduction</strong></h4>
Here is Daniel again and it's been a while I don't publish a blog here. The main reason for my absence is I moved my to a new role as Support Escalation Engineer in Microsoft Support called <a href="https://azure.microsoft.com/en-us/support/plans/response/">Azure Rapid Response</a> or ARR. I've been working in few Azure Networking cases and today I would like to present a new module worked by ARR  team to make possible to list and visualize certificates imported in Azure Application Gateway (AppGW).
<h4><strong>Quick overview about Application Getaway certificates.</strong></h4>
There are two types of certificates that can be used by Application Gateway.
<ul>
 	<li><strong>HTTP Listener certificate</strong> - This is PFX certificate you import to have your TLS/SSL connection to Application Gateway. This certificate includes private key of the certificated (basically the same kind of certificate you use on your web server).</li>
 	<li><strong>Backend Certificates</strong> - This is the certificate which contains public key and you use .CER format to upload the certificate which Application Gateway needs to reach the backend.
More information about AppGW certificates see: <a href="https://docs.microsoft.com/en-us/azure/application-gateway/application-gateway-ssl-portal">Create an application gateway with SSL termination</a></li>
</ul>
<h4><strong>Why we created this PowerShell Module?</strong></h4>
The main reason we created is there's no way at this time to list in printable format, either via Portal or PowerShell, certificate information once imported to Application Gateway.  We've seen couple customer creating support incident cases where they were unable to determine which certificate has been correctly uploaded to Application Gateway. Here is an example inside Listeners blade:

<img src="https://msdnshared.blob.core.windows.net/media/2018/02/Example-AppGW1.png" alt="" width="574" height="463" class="alignnone wp-image-975" />

<strong>*Note:</strong> PowerShell AzureRM command <a href="https://docs.microsoft.com/en-us/powershell/module/azurerm.network/get-azurermapplicationgateway">Get-AzureRmApplicationGateway</a> lists all configuration and certificate information is encoded as base64.
<h4><strong>How this module works?</strong></h4>
This module incorporates an application function to convert base64 format in certificate printable format by using: <em>[System.Security.Cryptography.X509Certificates.X509Certificate2]([System.Convert]::FromBase64String.</em>

See: <span><a href="https://msdn.microsoft.com/en-us/library/system.security.cryptography.x509certificates.x509certificate2(v=vs.110).aspx">X509Certificate2 Class</a> </span>for more information.

<strong>*Note1:</strong> Keep in mind this is PowerShell Module that is not officially support by Microsoft.

<strong>*Note2:</strong> This module may be incorporated in future releases of official AzureRM and while is not there yet you can leverage this module. Once that happens we will update this blog post to let you know.
<h4><strong>How to use AzureRMAppGWCert module?</strong></h4>
AzureRMAppGWCert module has been published in PSGallery and ready available to you and customers. See output example and other details below (extracted from <a href="https://github.com/Welasco/AzureRMAppGWCert">GitHub</a>):

AzureRMAppGWCert
Powershell Module to list all certificates from an Azure Application Gateway.
<h5><span style="text-decoration: underline"><b>Prerequisite</b></span></h5>
This module requires you have AzureRM installed. Please refer to the following instruction before you proceed: <a href="https://docs.microsoft.com/en-us/powershell/azure/install-azurerm-ps">Install and configure Azure PowerShell</a>.
<h5><span style="text-decoration: underline"><strong>How to Install</strong></span></h5>
This Module is Published at https://www.powershellgallery.com/packages/AzureRMAppGWCert
In order to install just open the powershell as Administrator and type:
<strong>Install-Module AzureRMAppGWCert</strong>
<strong>Import-Module AzureRMAppGWCert</strong>
<h5><span style="text-decoration: underline"><strong>Example: Listing all digital certificates associated with a single Application Gateway:</strong></span></h5>
This Module will list all certificates associated with Application Gateway and generate an output like this:
PS C:\&gt; <strong>Get-AzureRMAppGWCert -RG OfficeClient -AppGWName AppGateway</strong>
<pre> AppGWName : AppGateway
 ListnerName : appGatewayHttpListener
 Subject : CN=*.hepoca.com, O=Hepoca Armarios e Servicos Ltda - EPP, L=Taguatinga, S=Distrito Federal, C=BR
 Issuer : CN=DigiCert SHA2 Secure Server CA, O=DigiCert Inc, C=US
 SerialNumber : 0E99D5E2EBBE329CFE2DDE29C1D7D343
 Thumbprint : 5FD6F2A7BC4BD095198AE55D1A0A76D46365C6B9
 NotBefore : 3/13/2017 7:00:00 PM
 NotAfter : 5/2/2018 7:00:00 AM

 AppGWName : AppGateway
 ListnerName : HTTPs8080
 Subject : CN=*.hepoca.com, O=Hepoca Armarios e Servicos Ltda - EPP, L=Taguatinga, S=Distrito Federal, C=BR
 Issuer : CN=DigiCert SHA2 Secure Server CA, O=DigiCert Inc, C=US
 SerialNumber : 0E99D5E2EBBE329CFE2DDE29C1D7D343
 Thumbprint : 5FD6F2A7BC4BD095198AE55D1A0A76D46365C6B9
 NotBefore : 3/13/2017 7:00:00 PM
 NotAfter : 5/2/2018 7:00:00 AM

 AppGWName : AppGateway
 HTTPSetting : appGatewayBackendHttpSettings
 RuleName : rule1
 BackendCertName : webjson-pub
 Subject : E=a@a.com, CN=webjson.arr.local, OU=Arr, O=ARR, L=Irving, S=TX, C=US
 Issuer : E=a@a.com, CN=webjson.arr.local, OU=Arr, O=ARR, L=Irving, S=TX, C=US
 SerialNumber : 00B1722AB4D0FB8CAA
 Thumbprint : 573C70769A40CF4D01769926A212009598462436
 NotBefore : 11/28/2017 12:45:23 PM
 NotAfter : 11/28/2018 12:45:23 PM</pre>
<h5><span style="text-decoration: underline"><strong>Pratical Examples:</strong></span></h5>
<ol>
 	<li>This Example will get all Azure Application Gateways and list all certificates associated with all of them:
Get-AzureRMAppGWCert</li>
 	<li>Listing Application Gateway Certificates in a Resource Group:
Get-AzureRMAppGWCert -RG &lt;Resource Group Name&gt;</li>
 	<li>This Example will list all certificates associated with a specific Application Gateway:
Get-AzureRMAppGWCert -RG &lt;Resource Group Name&gt; -AppGWName &lt;Application Gateway Name&gt;</li>
 	<li>Listing all Application Gateway Certificates and exports all of them to certificate .cer format.
Get-AzureRMAppGWCert -Export</li>
 	<li>Listing all Application Gateway Certificates and show all details (all certificate attributes).
Get-AzureRMAppGWCert -Details</li>
</ol>
<h4>Demo</h4>
<img src="https://msdnshared.blob.core.windows.net/media/2018/02/AppGWCert2.gif" alt="" width="730" height="643" class="alignnone wp-image-955" />
<h4><strong>Conclusion</strong></h4>
In this article we described a new <strong>AzureRMAppGWCert</strong> Powershell module that can be used to list digital certificates used in your Azure Application Gateway. We hope this module is useful and help you better to manager digital certificates on Azure Application Gateway.
