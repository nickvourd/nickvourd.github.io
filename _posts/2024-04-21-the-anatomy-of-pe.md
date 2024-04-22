---
layout: post
title: "The Anatomy of PE"
image: ""
date: 2024-04-21 10:10:02
tags:
  - Windows Internals
  - C Language
  - Debugger
  - x64dbg
  - Architecture
  - metasploit

description: ""
categories:
  - Fundamentals
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

<p>Hello, world! Hello, world! <a href="https://twitter.com/nickvourd">@nickvourd</a> is calling! Welcome back to another blog post. Today, I want to discuss the Portable Executable (PE) file format. So, get ready, boyz/girlz, for the next round!</p>

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/mYtcJJZmkQOBgHRT2a" width="480" height="350" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<p>So, what is a PE?</p><br />

<p>The first time I heard about PE was in the course titled <a href="https://institute.sektor7.net/red-team-operator-malware-development-essentials">RED TEAM Operator: Malware Development Essentials Course</a> by <a href="https://institute.sektor7.net">Sector 7</a> (Which I totally recommend to anyone interested).<br /><br />➡️ PE is a way to organize executable code in a file on disk.<br /><br />And how does this actually work?<br /><br />➡️ The Windows Operating System has a component named the loader, which reads the PE from disk and then loads it into memory as a process and starts to execute it.</p><br />

<p>Examples of PE files can include <b>.exe</b>, <b>.dll</b>, <b>.sys</b>, and <b>.scr</b> files etc. </p><br />

<h3>PE Format</h3>

<p>However, I think Sector 7 offers the best introduction to the PE format that I have ever come across!<br /><br />What I mean?🤔<br /><br />Well, the PE structure is very complicated, but with any complex topic, you should change your approach to viewing it! The following picture shows the original (complex) structure view of PE. It is important to note that this image is from the <a href="https://en.wikipedia.org/wiki/Portable_Executable">Wikipedia</a> article:</p>

<img src="/assets/img/post-img/21-04-2024/Portable-ExecucatbleFormat.svg.png" class="post-images" alt="PE Structure from Wikipedia">

<p>As you can see, the structure of a PE file is really complicated. According to Sector 7, a better approach is to view it as a book. Personally, I think it's a great idea! Let's assume that a PE is a book. This book contains two parts: <b>data</b> and <b>metadata</b>. As for the data, we assume the author's text/content, while for the metadata, we consider the title of the book, the author's name, the publisher's name, the ISBN, the release date, and the table of contents etc.<br /><br />The following picture depicts a custom diagram representing a PE file as a book:</p>

<img src="/assets/img/post-img/21-04-2024/PE-Format-Simple.png" class="post-images" alt="PE-Structure-Custom-Simple" height="700" weight="700">

<p>Another great resource for presenting PE structure is the following image by <a href="https://github.com/corkami">corkami</a>:</p>

<img src="/assets/img/post-img/21-04-2024/pe101.png" class="post-images" alt="PE-Format-Corcami" height="500" weight="500">

<p>If you try to view the image above vertically, you'll see what I mean—it resembles a book. 😉</p><br />

<h3>PE Header Sections</h3>

<p>A PE file has several header sections, each serving specific purposes. Some of the most important ones include:

<ul>
  <li><b>.text</b>: This section contains executable code, typically the actual instructions to be executed by the processor. It is often marked as read-only and executable.</li>
  <li><b>.data</b>: The .data section holds initialized data that is used by the program during execution. This can include variables, constants, and other data structures that are initialized with specific values.</li>
  <li><b>.rdata</b>: This section stores read-only data, such as constant strings and other immutable values used by the program.</li>
  <li><b>.rsrc</b>: The .rsrc section is where resources like icons, images, dialogs, and version information are stored. These resources can be accessed by the program during runtime.</li>
  <li><b>.reloc</b>: This section contains relocation information, which is used by the loader to adjust memory addresses when the executable is loaded at a different base address than the one it was originally compiled for.</li>
  <li><b>.pdata</b>: The .pdata section, or Procedure Data, is used for exception handling and runtime function table information. It contains information about functions that have exception handling associated with them.</li>
</ul>
</p>

<p>From a malware development perspective, there are several options for storing the payload within different sections of the PE file, such as <b>.data</b>, <b>.rdata</b>, <b>.text</b>, and <b>.rsrc</b>. The following image shows the sections locations where a payload can be stored:</p>

<img src="/assets/img/post-img/21-04-2024/sections-payloads.drawio.png" class="post-images" alt="Sections-for-Payloads" height="500" weight="500">

<p>Before proceeding to detailed explanations of each section's payload storing implementation, we need to generate a shellcode. We will use the msfvenom module of <a href="https://docs.rapid7.com/metasploit/msf-overview/">Metasploit</a>. The following command will generate a calc shellcode:</p>

```
msfvenom -p windows/x64/exec CMD=calc.exe -f c 
```

<p>Outcome:</p>

<img src="/assets/img/post-img/21-04-2024/msfvenom-payload-generation.png" class="post-images" alt="msfvenom-payload-generation" height="500" weight="500">

<h4>.data Section</h4>

<p>The following list describes the most important characteristics of the <b>.data</b> section (It is easier to remember them when presented as a list rather than a paragraph):

<ul>
  <li>Initialized global variables</li>
  <li>Readable and writable section</li>
</ul>
</p>

```
#include <Windows.h>
#include <stdio.h>

// Set shellcode as global variable (.data section)
unsigned char buf[] = 
	"\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
	"\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52"
	"\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a"
	"\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41"
	"\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52"
	"\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48"
	"\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40"
	"\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48"
	"\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41"
	"\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1"
	"\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c"
	"\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01"
	"\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a"
	"\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b"
	"\x12\xe9\x57\xff\xff\xff\x5d\x48\xba\x01\x00\x00\x00\x00"
	"\x00\x00\x00\x48\x8d\x8d\x01\x01\x00\x00\x41\xba\x31\x8b"
	"\x6f\x87\xff\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd"
	"\x9d\xff\xd5\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0"
	"\x75\x05\xbb\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff"
	"\xd5\x63\x61\x6c\x63\x2e\x65\x78\x65\x00";

int main() {
	printf("[+] Global Variable Base Address: 0x%p\n", buf);
	printf("[*] Press <Enter> To Exit ...");
	getchar();
	return 0;
}
```

<p>Let's start x64dbg and attach it to the PE's running process. The base address of the global variable is <code> 0x00007FF65083C000</code>. As you can see in the following image, the base address (<code>0x00007FF65083C000</code>) corresponds to the <b>.data</b> section:</p>

<img src="/assets/img/post-img/21-04-2024/x64dbg-data-section.png" class="post-images" alt="x64dbg-data-section" height="500" weight="500">

<h4>.rdata Section</h4>

<p>The following list describes the most important characteristics of the <b>.rdata</b> section:

<ul>
  <li>Initialized constants</li>
  <li>Readable section</li>
</ul>
</p>

```
#include <Windows.h>
#include <stdio.h>

// Set shellcode as constant (.rdata section)
const unsigned char buf[] = 
	"\xfc\x48\x83\xe4\xf0\xe8\xc0\x00\x00\x00\x41\x51\x41\x50"
	"\x52\x51\x56\x48\x31\xd2\x65\x48\x8b\x52\x60\x48\x8b\x52"
	"\x18\x48\x8b\x52\x20\x48\x8b\x72\x50\x48\x0f\xb7\x4a\x4a"
	"\x4d\x31\xc9\x48\x31\xc0\xac\x3c\x61\x7c\x02\x2c\x20\x41"
	"\xc1\xc9\x0d\x41\x01\xc1\xe2\xed\x52\x41\x51\x48\x8b\x52"
	"\x20\x8b\x42\x3c\x48\x01\xd0\x8b\x80\x88\x00\x00\x00\x48"
	"\x85\xc0\x74\x67\x48\x01\xd0\x50\x8b\x48\x18\x44\x8b\x40"
	"\x20\x49\x01\xd0\xe3\x56\x48\xff\xc9\x41\x8b\x34\x88\x48"
	"\x01\xd6\x4d\x31\xc9\x48\x31\xc0\xac\x41\xc1\xc9\x0d\x41"
	"\x01\xc1\x38\xe0\x75\xf1\x4c\x03\x4c\x24\x08\x45\x39\xd1"
	"\x75\xd8\x58\x44\x8b\x40\x24\x49\x01\xd0\x66\x41\x8b\x0c"
	"\x48\x44\x8b\x40\x1c\x49\x01\xd0\x41\x8b\x04\x88\x48\x01"
	"\xd0\x41\x58\x41\x58\x5e\x59\x5a\x41\x58\x41\x59\x41\x5a"
	"\x48\x83\xec\x20\x41\x52\xff\xe0\x58\x41\x59\x5a\x48\x8b"
	"\x12\xe9\x57\xff\xff\xff\x5d\x48\xba\x01\x00\x00\x00\x00"
	"\x00\x00\x00\x48\x8d\x8d\x01\x01\x00\x00\x41\xba\x31\x8b"
	"\x6f\x87\xff\xd5\xbb\xf0\xb5\xa2\x56\x41\xba\xa6\x95\xbd"
	"\x9d\xff\xd5\x48\x83\xc4\x28\x3c\x06\x7c\x0a\x80\xfb\xe0"
	"\x75\x05\xbb\x47\x13\x72\x6f\x6a\x00\x59\x41\x89\xda\xff"
	"\xd5\x63\x61\x6c\x63\x2e\x65\x78\x65\x00";

int main() {
	printf("[+] Constant Base Address: 0x%p\n", buf);
	printf("[*] Press <Enter> To Exit ...");
	getchar();
	return 0;
}
```

<br /><br />

<!-- add the button!-->
<div>
<applause-button style="width: 58px; height: 58px;" color="#5d4d7a" url="https://nickvourd.github.io/the-anatomy-of-pe/"/>
</div>