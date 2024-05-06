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
  - Malware Dev Fundamentals
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

<p>However, I think <a href="https://institute.sektor7.net">Sector 7</a> offers the best introduction to the PE format that I have ever come across!<br /><br />What I mean?🤔<br /><br />Well, the PE structure is very complicated, but with any complex topic, you should change your approach to viewing it! The following picture shows the original (complex) structure view of PE. It is important to note that this image is from the <a href="https://en.wikipedia.org/wiki/Portable_Executable">Wikipedia</a> article:</p>

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

<p>Let's start x64dbg and attach it to the PE's running process. The base address of the global variable is <code> 0x00007FF6BA84C000</code>. As you can see in the following image, the base address (<code>0x00007FF6BA84C000</code>) corresponds to the <b>.data</b> section:</p>

<img src="/assets/img/post-img/21-04-2024/x64dbg-data-section.png" class="post-images" alt="x64dbg-data-section">

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

<p>Let's fire up x64dbg and attach it to the running process of the PE. The base address of the constant is <code>0x00007FF79C7D9BB0</code> according to our PE's output.</p>

<img src="/assets/img/post-img/21-04-2024/cmd-base-address-const.png" class="post-images" alt="cmd-base-address-const">

<p>However, according to x64dbg, the base address corresponding to the <b>.rdata</b> section is <code>0x00007FF79C7D9000</code>!</p>

<img src="/assets/img/post-img/21-04-2024/x64dbg-base-address-const.png" class="post-images" alt="x64dbg-base-address-const">

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/dDrgcii3mVnUhbjhGk" width="480" height="350" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<p>Of course not! The base addresses above differ by the last three characters, as you can see. I asked my friend named <a href="https://twitter.com/S1ckB0y1337">@S1ckB0y1337</a> about this, and he told me, "This is the Offset, bro!". So, what is an offset? An offset is the distance from the base address of a data structure to a specific element (e.g., variable, constant).<br /><br />As you can see from the above image in x64dbg, each memory address is associated with a memory size (e.g., 1000, 3000), indicating that the offsets will fall within this range. In our case, the memory size is <code>3000</code>, and the offset is 0xBB4, which is equivalent to <code>int("0xBB4", 16)</code>, resulting in <code>2996</code>. This is indeed true because the offset falls within the memory size range.</p>

<img src="/assets/img/post-img/21-04-2024/find-offset-python.png" class="post-images" alt="find-offset-python.png">

<h4>.text Section</h4>

<p>According to <a href="https://maldevacademy.com/">MalDev Academy</a>, to store the payload in the <b>.text</b> section, you must instruct the compiler to do this. More specifically, it differs from the declaration of global variables or constants. The compiler's instructions indicate that the variable is placed in the <b>.text</b> section and not in the <b>.rdata</b> or <b>.data</b> sections.</p><br />

<p>The following list describes the most important characteristics of the <b>.text</b> section:

<ul>
  <li>Declarations as per compiler's instructions</li>
  <li>Executable memory permissions (no need to edit memory permissions)</li>
  <li>Useful for small-size payloads, typically less than 10 bytes</li>
</ul>
</p>

```
#include <Windows.h>
#include <stdio.h>

// Set shellcode as constant with compiler's instructions (.text section)
#pragma section(".text")
__declspec(allocate(".text")) const unsigned char buf[] = 
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

<p>Before proceeding with debugging, let's explain the above code.<br /><br />The <code>#pragma section(".text")</code> directive is used to define a new section named <b>.text</b>. This directive is specific to certain compilers (like Microsoft Visual Studio's compiler) and allows you to specify custom section names in the object file produced by the compiler.<br /><br />The <code>__declspec(allocate(".text"))</code> attribute is used to instruct the compiler to allocate the variable (in this case, the shellcode) in the <b>.text</b> section of the executable. This ensures that the shellcode is placed in the executable code section, which typically contains executable instructions.<br /><br />Let's open x64dbg and attach it to the binary process. The base address of the constant is <code>0x00007FF775F317B0</code> according to our PE's output.</p>

<img src="/assets/img/post-img/21-04-2024/cmd-base-address-const-text.png" class="post-images" alt="cmd-base-address-const-text">

<p>From the x64dbg side, you can observe that the base address corresponding to the <b>.text</b> section is <code>00007FF775F31000</code>, which means that <code>0x07B0</code> (Decimal: <code>1968</code>) is the offset.</p>

<img src="/assets/img/post-img/21-04-2024/x64dbg-text-section.png" class="post-images" alt="cmd-base-address-const-text">

<h4>.rsrc Section</h4>

<p>The last section that can store a payload is <b>.rsrc</b>. According to <a href="https://maldevacademy.com/">MalDev Academy</a>, this section is very popular among malware developers. The main reason is its storage size. The <b>.rsrc</b> section can store larger payloads than the limited size that can be stored in the <b>.data</b> or <b>.rdata</b> sections.<br /><br />First of all, we will save the raw shellcode in a file with the extension <b>.ico</b>:</p>

```
msfvenom -p windows/x64/exec CMD=calc.exe -f raw -o calc.ico
```

<p>After that, in order to store the payload in the <b>.rsrc</b> section, we need to create a resource file in the project's Solution Explorer in Visual Studio. More specifically, in Visual Studio, right-click on <b>Resource Files</b>, then click <b>Add</b> > <b>New Item</b> and choose the <b>Resource</b> category and <b>Resource File (.rc)</b>.</p>

<img src="/assets/img/post-img/21-04-2024/create-resource-file-1.png" class="post-images" alt="create-resource-file-1">

<img src="/assets/img/post-img/21-04-2024/create-resource-file-2.png" class="post-images" alt="create-resource-file-2">

<p>After that, a new sidebar will appear named <b>Resource View</b>, which will contain your new <b>.rc</b> file. Right-click on the resource file and choose the option named <b>Add Resource...</b>.</p>

<img src="/assets/img/post-img/21-04-2024/create-resource-file-3.png" class="post-images" alt="create-resource-file-3">

<img src="/assets/img/post-img/21-04-2024/create-resource-file-4.png" class="post-images" alt="create-resource-file-4">

<p>A new window will appear, and we will use the <b>import</b> button to select the <b>calc.ico</b> file, which contains the raw payload as mentioned above.</p>

<img src="/assets/img/post-img/21-04-2024/create-resource-file-5.png" class="post-images" alt="create-resource-file-5">

<p>After importing the ".ico" file, a new window will appear asking for the <b>Resource type</b> of this ICO file. We need to provide the following value: <code>RCDATA</code>.</p>

<img src="/assets/img/post-img/21-04-2024/create-resource-file-6.png" class="post-images" alt="create-resource-file-6">

<img src="/assets/img/post-img/21-04-2024/create-resource-file-7.png" class="post-images" alt="create-resource-file-7">

<p>After that, we need to determine if the payload is stored in the <b>.rsrc</b> section. However, this cannot be done directly; only the following WinAPIs can be used to access it:<br />
<ul>
    <li><b>FindResourceW</b></li>
    <li><b>LoadResource</b></li>
    <li><b>LockResource </b></li>
    <li><b>SizeofResource </b></li>
</ul>
</p>

<p>As we can see from the names of the above WinAPIs, it's obvious what they are doing, but let's find out more details about them.<br /><br />The <a href="https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-findresourcew">FindResourceW</a> attempts to locate the location of a resource identified by a specified type and name within the given module. The identification of the resource is typically done using an ID defined in the <b>resource.h</b> header file.<br ><br />The <a href="https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadresource">LoadResource</a> retrieves a handle (<b>HGLOBAL</b>) that can be used to obtain the base address of the specified resource in memory.<br /><br />The <a href="https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-lockresource">LockResource</a> retrieves a pointer to the specified data in the resource section from its handle (<b>HGLOBAL</b>).<br /><br /><a href="https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-sizeofresource">SizeofResource</a> retrieves the size, in bytes, of the specified resource.<br /><br />The following code utilizes the aforementioned WinAPIs to access the <code>.rsrc</code> section:</p>

```
#include <Windows.h>
#include <stdio.h>
#include "resource.h"

int main() {
    // Declaring variables
    HRSRC hRsrc = NULL;
    HGLOBAL hGlobal = NULL;
    PVOID pPayloadAddress = NULL;
    SIZE_T sPayloadSize = 0;

    // Find the resource with the specified ID and type in the .rsrc section
    hRsrc = FindResourceW(NULL, MAKEINTRESOURCEW(IDR_RCDATA1), RT_RCDATA);

    // Load the specified resource into memory
    hGlobal = LoadResource(NULL, hRsrc);

    // Lock the resource and retrieve a pointer to its data
    pPayloadAddress = LockResource(hGlobal);

    // Get the size of the resource
    sPayloadSize = SizeofResource(NULL, hRsrc);

    // Print the address and size of the payload
    printf("[+] Payload Address : 0x%p \n", pPayloadAddress);
    printf("[+] Payload Size : %ld \n", sPayloadSize);
    printf("[+] Press <Enter> To Quit ...");
    getchar();

    return 0;
}
```

<p>Let's compile it and attach it to x64dbg to observe what is happening at a low level. As you can see, the payload's base address is <code>0x00007FF76ABE60A0</code>, and according to x64dbg, the base address is <code>00007FF76ABE6000</code>, indicating that the offset is <code>0x0A0</code> (Decimal: <code>160</code>). Moreover, as you can see, the payload is stored in the <b>.rsrc</b> section.</p>

<img src="/assets/img/post-img/21-04-2024/cmd-base-address-rsrc.png" class="post-images" alt="cmd-base-address-rsrc">

<p>As we already know, this section is not writable, so we cannot write to it directly. For this reason, if we need to edit the payload located in the <b>.rsrc</b> section, we need to move it to a temporary buffer. In order to do that, we need to allocate memory (from the heap memory section, as mentioned in <a href="https://nickvourd.github.io/an-introduction-to-windows-memory-management/">An Introduction to Windows Memory Management</a> blog post) and move the payload from the <b>.rsrc</b> section to the temporary buffer using <code>memcpy</code>. Let's edit the above code:</p>

```
#include <Windows.h>
#include <stdio.h>
#include "resource.h"

int main() {
    // Declaring variables
    HRSRC hRsrc = NULL;
    HGLOBAL hGlobal = NULL;
    PVOID pPayloadAddress = NULL;
    SIZE_T sPayloadSize = 0;

    // Find the resource with the specified ID and type in the .rsrc section
    hRsrc = FindResourceW(NULL, MAKEINTRESOURCEW(IDR_RCDATA1), RT_RCDATA);

    // Load the specified resource into memory
    hGlobal = LoadResource(NULL, hRsrc);

    // Lock the resource and retrieve a pointer to its data
    pPayloadAddress = LockResource(hGlobal);

    // Get the size of the resource
    sPayloadSize = SizeofResource(NULL, hRsrc);

    // Print the address and size of the payload
    printf("[+] Payload Address : 0x%p \n", pPayloadAddress);
    printf("[+] Payload Size : %ld \n", sPayloadSize);

    // Allocating memory using a HeapAlloc call
    PVOID pTmpBuffer = HeapAlloc(GetProcessHeap(), 0, sPayloadSize);

    if (pTmpBuffer != NULL) {
        // copying the payload from rsrc section to the new temp buffer
        memcpy(pTmpBuffer, pPayloadAddress, sPayloadSize);
    }

    // Printing the base address of temp buffer
    printf("[+] Temp buffer address : 0x%p \n", pTmpBuffer);

    // Freeing the allocated memory
    HeapFree(GetProcessHeap(), 0, pTmpBuffer);

    return 0;
}
```

<p>Let's add some breakpoints in our code to observe what's happening behind the scenes. We will add the first breakpoint in line 29 where we try to allocate memory. We'll add another breakpoint in line 40 where we free the allocated memory.</p>

<img src="/assets/img/post-img/21-04-2024/debugging-the-code.png" class="post-images" alt="debugging-the-code">

<p>As depicted in the following picture, the variable <code>pTmpBuffer</code> now points to a writable memory region containing the payload. This enables us to make any necessary updates to it:</p>

<img src="/assets/img/post-img/21-04-2024/debugging-the-code-2.png" class="post-images" alt="debugging-the-code-2">

<p>Let's press the <b>Continue</b> button to proceed and observe what happens at the next breakpoint.</p>

<img src="/assets/img/post-img/21-04-2024/debugging-the-code-3.png" class="post-images" alt="debugging-the-code-3">

<p>As we can see the payload is saved in the <code>pTmpBuffer</code> (base address: <code>0x00000234B75F1050</code>).</p>

<br /><br />

<!-- add the button!-->
<div>
<applause-button style="width: 58px; height: 58px;" color="#5d4d7a" url="https://nickvourd.github.io/the-anatomy-of-pe/"/>
</div>