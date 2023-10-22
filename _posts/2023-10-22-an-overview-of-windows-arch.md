---
layout: post
title: "An overview of Windows architecture"
image: ""
date: 2023-10-22 10:10:02
tags:
  - Windows Internals
  - Architecture
  - C Language
  - Assembly

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

<p>Hello folks, welcome back, and have a nice day! 😃. Today, I will explain some internal aspects of Windows architecture, providing an overview of how processes work and memory allocation. You will need to know some basic aspects of <a href="https://x64dbg.com">x64dbg</a> to follow this article. If you haven't already read my previous article on <a href="https://nickvourd.github.io/x64dbg-intro/">the introduction of x64dbg</a>, you should do so first. So, get ready, boyz/girlz, for the next round!</p>

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/7WvAUvZZTRpSuudobh" width="480" height="350" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<h3>x86/x86-64</h3>

<p>There are various computer architectures, and each has its own unique instruction set and design. The most popular architectures that we will discuss in this article are x86 and x86-64. What do x86 and x86-64 mean? Well, the terms x86 and x86-64 refer to computer processor architectures.<br /></p>
<ul>
	<li>
		<p><strong>x86</strong> typically refers to a family of backward compatible instruction set architectures based on the Intel 8086 CPU. It has become the standard for many personal computers and servers.</p>
	</li>
	<li>
		<p><strong>x86-64</strong>, also known as x64 or AMD64, is an extension of the x86 instruction set, allowing for 64-bit addressing. This extension enables processors to handle larger amounts of memory and perform more complex calculations compared to earlier 32-bit versions of the x86 architecture. Most modern desktops, laptops, and servers utilize the x86-64 architecture.</p>
	</li>
</ul>

<p>The following table presents the main differences of x86 and x86-64 architectures:</p>

<table border="1">
    <tr>
        <th>Characteristic</th>
        <th>x86 (32-bit)</th>
        <th>x86-64 (64-bit)</th>
    </tr>
    <tr>
        <td>Word Size</td>
        <td>32-bit</td>
        <td>64-bit</td>
    </tr>
    <tr>
        <td>Memory Limit</td>
        <td>4 GB</td>
        <td>Several TB</td>
    </tr>
    <tr>
        <td>Performance</td>
        <td>Limited by 32-bit processing</td>
        <td>Enhanced performance due to 64-bit processing and larger memory addressing</td>
    </tr>
    <tr>
        <td>Compatibility</td>
        <td>32-bit applications can generally run on both 32-bit and 64-bit processors</td>
        <td>64-bit applications can only run on 64-bit processors</td>
    </tr>
    <tr>
        <td>Register Set</td>
        <td>Smaller number of general-purpose registers</td>
        <td>More general-purpose registers, leading to potential performance improvements for specific tasks</td>
    </tr>
</table>

<h3>Basic Terms</h3>

<p>Before we move on to the next section, I would like to explain some basic terms:</p><br />
<ul>
	<li>
		<p><strong>Windows Process</strong>: A process in the Windows operating system is an instance of a computer program that is being executed. It has its own memory space, system resources, and is managed by the Windows kernel.</p>
	</li>
	<li>
		<p><strong>Threads</strong>: Threads are the smallest unit of execution within a process in the Windows operating system. They allow for concurrent execution of multiple tasks within the same process.</p>
	</li>
	<li>
		<p><strong>Windows Processor</strong>: The Windows Processor refers to the CPU (Central Processing Unit) within a Windows-based system. It executes instructions and processes data within the operating system.</p>
	</li>
	<li>
		<p><strong>Windows Service</strong>: A Windows service is a program that runs in the background, independently of any user and without the need for a user to log in to the PC. Services are typically used for tasks such as managing network connections, web servers, and other system functions.</p>
	</li>
	<li>
		<p><strong>Windows Kernel</strong>:The Windows Kernel is the core of the Windows operating system. It is responsible for low-level tasks such as hardware management, process management, memory management, and security.</p>
	</li>
	<li>
		<p><strong>Handle</strong>: In the context of Windows, a handle is a reference to an object that can be manipulated by the operating system. It allows processes to interact with system resources like files, devices, or other objects.</p>
	</li>
	<li>
		<p><strong>Windows API</strong>: The Windows API (Application Programming Interface) is a set of functions and data structures that provide an interface for programmers to interact with the Windows operating system. It allows applications to access and use operating system services.</p>
	</li>
	<li>
		<p><strong>Native API</strong>: The Native API is a lower-level interface used by the Windows operating system internally. It provides direct access to system functions that are not accessible through the standard Windows API. It is primarily used for system-level programming and development.</p>
	</li>
	<li>
		<p><strong>DLL</strong>: A DLL (Dynamic Link Library) is a file that contains code and data that can be used by multiple programs at the same time. It allows software to modularize functionalities and share resources, promoting code reuse and efficient memory usage.</p>
	</li>
	<li>
		<p><strong>Syscall</strong>: A syscall (system call) is a request made by an active process to the Windows Kernel. It allows user-level programs to request services from the operating system, such as file operations, input/output operations, and process control.</p>
	</li>
</ul>

<h3>User mode vs Kernel mode</h3>

<p>According to the book <a href="https://www.amazon.com/Windows-Internals-Part-architecture-management/dp/0735684189">'Windows Internals, Part 1'</a> by <a href="https://twitter.com/zodiacon">Pavel Yosifovich</a>, there are two main access modes that a Windows processor uses: User mode (a.k.a. User Land) and Kernel mode (a.k.a. Kernel Land).</p><br />

<p>The main reason for this distinction is to protect user applications from accessing and modifying critical Operating System data. User application code runs in User Land, while operating system componets, such as system services and device drivers, operates in Kernel Land.</p><br />

<p>Kernel Land is an execution mode in a processor that allows access to all system memory and CPU instructions. However, some processors use alternative terms for this mode, such as supervisor mode, privilege level, or ring level. Regardless of the nomenclature, the primary purpose of this CPU design is to elevate the kernel to a higher privilege level than user mode applications. This is done to ensure that a misbehaving application cannot compromise the overall stability of the system.</p><br />

<p>Personally, I prefer the term 'ring level' for kernel mode. It reminds me of one of my favorite movies, The Lord of the Rings! 😂</p>

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/VfwIk1LD84CI" width="480" height="350" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<p>At this point, it is important to note that both the x86 and x86-64 architectures define four (4) ring/privilege levels to protect system code from malicious actions or misbehaviors of the applications. The ring 0 is used for Kernel mode and ring 3 is used for user mode. The main reason Windows uses only two levels is that some hardware architectures, such as ARM, implement only two privilege levels and there are no intermediate levels.</p><br />

<h3>Function Call Flow</h3>

<p>User applications do not have the capability to perform tasks independently; the kernel is the only entity that can complete any task. Consequently, applications typically follow a standard function call flow to execute tasks. Generally, this flow allows any user application to call a function through the Windows API. After a series of steps, the request reaches the kernel, which then executes it. The following diagram illustrates this general approach. I inspired it from <a href="https://maldevacademy.com">Maldev academy</a>, and I recommend it to anyone looking to learn more about evasion.</p>

<img src="/assets/img/post-img/22-10-2023/Diagram-1.png" class="post-images" alt="General Diagram Function Call Flow">

<p>Let's explain this diagram:<br /><br /></p>

<ul>
	<li>
		<p><strong>User Process</strong>: An application that is executed by the user (i.e., Notepad, Firefox).</p>
	</li>
	<li>
		<p><strong>SubSystem DLLs</strong>: Dynamic Link Libraries (DLLs) containing Windows API functions that are invoked by user processes (i.e., kernel32.dll for CreateFileW API).</p>
	</li>
	<li>
		<p><strong>ntdll.dll</strong>: A system-wide DLL which is the lowest layer available in user land. This is a special DLL that creates the transition from user kernel to kernel land.</p>
	</li>
	<li>
		<p><strong>Kernel</strong>: The Windows Kernel calls drivers and other modules to complete tasks.</p>
	</li>
</ul>

<p>ℹ️ The Windows kernel is partly stored in a file named "ntoskrnl.exe" located in "C:\Windows\System32".<br /></p>

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/xT9KVqOt8xuRYhNpq8" width="480" height="350" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<p>Before proceeding with a practical example, I want to clarify a crucial point. I intend to provide more details about this function call flow to enhance familiarity with it.<br /><br />
Let's consider a sample C application that utilizes the CreateDirectoryW Windows API, to create a new directory:</p>

```
#include <stdio.h>
#include <windows.h>

int main() {
    LPCWSTR dirPath = L"C:\\Users\\nickvourd\\Desktop\\NewDirectory";

    if (CreateDirectoryW(dirPath, NULL)) {
        wprintf(L"[+] Directory '%s' created successfully.\n", dirPath);
    } else {
        wprintf(L"[-] CreateDirectoryW API Function Failed with Error: %d\n", GetLastError());
        return -1;
    }

    return 0;
}
```
<p>ℹ️ The "windows.h" header file includes DLLs related to Windows API functions.<br /><br />

Referring to the earlier diagram, when the user executes this C program, <span style="color: red;">please remember the following five (5) steps (as general overview)</span>:</p>

<ol>
	<li>
		<p>The C program calls subsystem DLLs, such as kernel32.dll.</p>
	</li>
	<li>
		<p>The kernel32.dll contains the CreateDirectoryW Windows API.</p>
	</li>
	<li>
		<p>The CreateDirectoryW API calls the Native API named ZwCreateDirectory, which is part of ntdll.dll.</p>
	</li>
	<li>
		<p>The ntdll.dll executes an assembly sysenter (x86) or syscall (x64) instruction, transferring the execution to kernel land.</p>
	</li>
	<li>
		<p>Finally, the kernel employs ZwCreateDirectory to invoke other modules and drivers in order to execute the task.</p>
	</li>
</ol>

<p>The following diagram illustrates the aforementioned five (5) steps:</p>

<img src="/assets/img/post-img/22-10-2023/Diagram-2.png" class="post-images" alt="Example Diagram Function Call Flow">

<p>ℹ️ It is essential to understand that applications have the capability to directly invoke syscalls (i.e., NTDLL functions) without necessarily navigating through the Windows API. The Windows API essentially functions as a wrapper for the Native API.</p>

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/2599kjzl9M5anzPSbh" width="480" height="350" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<p>Alright, let's proceed with a practical example! 😛<br /><br />

We will compile the above example C code into Demo.exe and then analyze it in x64dbg.</p>
<ul>
	<li>
		<p>The user application calls the CreateDirectoryW Windows API.</p>
	</li>
</ul>

<img src="/assets/img/post-img/22-10-2023/x64dbg-CreateDirectoryW-Call.png" class="post-images" alt="x64dbg Call instruction of CreateDirectoryW">

<ul>
	<li>
		<p>Next, CreateDirectoryW calls its equivalent NTAPI function, ZwCreateDirectory.</p>
	</li>
</ul>

<img src="/assets/img/post-img/22-10-2023/x64dbg-NtDirectory-Call.png" class="post-images" alt="x64dbg Call instruction of ZwDirectory">

<ul>
	<li>
		<p>Last but not least, the ZwCreateDirectory function uses a syscall assembly instruction to transition from user land to kernel land. The kernel will be the entity that creates the new directory.</p>
	</li>
</ul>

<img src="/assets/img/post-img/22-10-2023/x64dbg-Syscall.png" class="post-images" alt="x64dbg Syscall">

<!-- add the button!-->
<div>
<applause-button style="width: 58px; height: 58px;" color="#5d4d7a" url="https://nickvourd.github.io/win-arch-intro"/>
</div>
