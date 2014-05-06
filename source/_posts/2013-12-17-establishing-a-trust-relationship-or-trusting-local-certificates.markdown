---
layout: post
title: "Establishing a Trust Relationship or Trusting Local Certificates"
date: 2013-12-17 10:52:17 -0400
comments: true
categories: 
---

At work, the certificates in our lab environment are signed by a local certificate authority. Additionally, we do not have the lab intermediate certificate authorities on our local workstations. This means that when we call WCF or REST services we sometimes see the following error:

	Could not establish trust relationship for the SSL/TLS secure channel with authority 'someserver.whereiwork.com'.

This error can be resolved either by importing the intermediate certificate authorities on your workstation, or by using the following code.

Please note that this code should only be executed **once** in your application (`global.asax`) and **never** in a production environment because it short circuits the protection offered by the PKI. 

C#:

{% codeblock lang:c# C# %}
#if DEBUG
	ServicePointManager.ServerCertificateValidationCallback += (sender, cert, chain, sslPolicyErrors) => true;
#endif
{% endcodeblock %}

If you have to do this in Visual Basic, use the following:

{% codeblock lang:vbnet VB.NET %}
#If DEBUG Then
	ServicePointManager.ServerCertificateValidationCallback = AddressOf AcceptAllCertificates
#End If

Private Function AcceptAllCertificates(ByVal sender As Object, ByVal cert As X509Certificate, ByVal chain As X509Chain, ByVal sslPolicyErrors As SslPolicyErrors) As Boolean
    Return True
End Function
{% endcodeblock %}
