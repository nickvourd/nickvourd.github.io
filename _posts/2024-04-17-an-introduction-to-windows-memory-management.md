---
layout: post
title: "An Introduction to Windows Memory Management (Part 2)"
image: ""
date: 2024-04-17 10:10:02
tags:
  - Windows Internals
  - Architecture
  - C Language

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

<p>Hello, folks! After a long time, I've officially found some time to continue this awesome journey. In my last blog post, we discussed some topics about Windows architecture, such as the difference between x86 and x86-64 architectures, some basic terms, and we examined a detailed example of a function call flow. If you haven't already read my previous article <a href="https://nickvourd.github.io/an-overview-of-windows-arch/">An Overview of Windows Architecture (Part 1)</a>, you should do so before continuing. This is officially Part 2. So, get ready, boyz/girlz, for the next round!</p>

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/Yh30S0qiIw1wsF1L0T" width="480" height="350" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<h3>Intro to Memory</h3>

<p>The first thing we need to discuss is what memory is! Memory in computing systems refers to the electronic components that store data and instructions for processing by the Central Processing Unit (CPU).Most people out there, when they hear the word "memory," typically think of RAM or a hard disk. To be honest, while these examples are contextualized to memory types, in Windows Internals, the meaning is slightly different...<br /><br />

According to the book <a href="https://www.amazon.com/Windows-Internals-Part-architecture-management/dp/0735684189">'Windows Internals, Part 1'</a> by <a href="https://twitter.com/zodiacon">Pavel Yosifovich</a>, modern operating systems do not use directly physical memory (i.e., RAM) for mapping. Instead, they utilize virtual memory addressing, where each process has its own virtual address space.<br /><br />

You may rightly wonder why it's so important to work this way rather than mapping directly to physical addresses. The answer is so simple; it's all about optimizing the use of physical memory resources to improve system performance and reliability.<br /><br />

By utilizing virtual memory addressing and allowing the operating system's Memory Management Unit (MMU) to handle the mapping of virtual addresses to physical memory addresses, modern operating systems can efficiently manage physical memory utilization. This approach enables the system to allocate memory resources dynamically based on the current needs of processes and applications, maximizing the use of available physical memory and minimizing waste. Additionally, virtual memory addressing facilitates memory protection and isolation between processes, enhancing system stability and security (We will discuss memory protections later in this article).</p><br />

<h3>Memory Structures</h3>

<p>Before proceeding to technical details and protections regarding virtual memory, it is important to explain the basic virtual memory structures.<br /><br />

For years, I've been hearing about terms like stack, heap, and more from hardcore cybersecurity colleagues, and of course, from <a href="https://twitter.com/0xvm">Lovely Uncle Bill</a>. To be honest, all this stuff seemed very confusing to me. So, I started from the basics to understand what they are.<br /><br />

The following picture depicts an overview of the virtual memory layout of a process (x86). Also, it is important to note that this picture inspired by the great blog post <a href="https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/">Exploit writing tutorial part 1 : Stack Based Overflows</a> by <a href="https://twitter.com/corelanconsultn">@corelanconsultn</a>:</p>

<img src="/assets/img/post-img/17-04-2024/Memory-Strucutre.png" class="post-images" alt="Exploit writing tutorial part 1 : Stack Based Overflows (Corelan)" height="500" weight="500">

<p>Let’s work our way up from the bottom (Kernel Land), starting with the portion of memory from 0xFFFFFFFF to 0x7FFFFFFF, and discuss the most important parts:</p><br />

<h4>Kernel Land</h4>

<p>As we discussed in my previous article titled <a href="(https://nickvourd.github.io/an-overview-of-windows-arch/">An Overview of Windows Architecture (Part 1)</a>, Kernel Land is an execution mode in a processor that allows access to all system memory and CPU instructions. This portion of memory (0xFFFFFFFF) is reserved by the Opertaing System for device drivers, system cache, paged/non-paged pool, etc. There is no user access to this portion of memory.</p><br />

<h4>Process Environment Block (PEB)</h4>

<p>The Process Environment Block (PEB) is a data structure used by the Windows operating system to store information about a process. It contains various attributes and parameters related to the execution environment of the process. Some of the information stored in the PEB includes process parameters, environment variables, the process heap pointer, and loader data.</p><br />

<h4>Thread Environment Block (TEB)</h4>

<p>The Thread Environment Block (TEB) is a data structure used by the Windows operating system to store information specific to a thread. Each thread in a Windows process has its own TEB. The TEB contains thread-specific information such as the thread ID, thread-local storage (TLS) data, exception handling information, and a pointer to the Process Environment Block (PEB) of the process to which the thread belongs. Additionally, the TEB may include other thread-related data required for efficient thread execution and management.</p><br />

<h4>Heap</h4>

<p>The heap is a region of memory used for dynamic memory allocation in computer programs. The heap allows for the allocation and deallocation of memory blocks at runtime. In Windows operating systems, processes can allocate memory from the heap using functions like `HeapAlloc` and `HeapFree`. The heap is managed by the operating system's Heap Manager, which tracks allocated and free memory blocks and ensures efficient memory usage.</p><br />

<h4>Stack</h4>

<p>The stack is a region of memory used for storing function call and local variable data during program execution. It operates on a last-in, first-out (LIFO) basis, meaning that the last item pushed onto the stack is the first one to be popped off.<br /><br />When a function is called, its parameters and local variables are typically allocated on the stack. As subsequent functions are called within the current function, additional stack frames are created, each containing the necessary information for the corresponding function call. When a function returns, its stack frame is removed from the stack, and control returns to the calling function.<br /><br />The stack is managed automatically by the CPU and is typically of fixed size, although it can grow dynamically in some systems.</p><br />

<h4>Memory Structures Summary</h4>

<p>I know that's a lot of information, and honestly, the main part of my job as an Offensive Security Consultant is to explain difficult topics to my clients and make them easy to understand. This is a skill that I learned from <a href="https://www.linkedin.com/in/panosstam/">Mr_P</a> and <a href="https://www.linkedin.com/in/aretis/">Dimitris Aretis</a>.<br /><br />

Let's try this together! When I mention a term, please keep two words in mind:<br />
<ul>
    <li><b>Kernel Land</b>: The lowest level managed by the operating system.</li>
    <li><b>PEB</b>: A data structure that contains information about processes.</li>
    <li><b>TEB</b>: A data structure that contains information about threads within a process.</li>
    <li><b>Heap</b>: A memory region used for dynamic memory allocation during runtime.</li>
    <li><b>Stack</b>: A memory region used for function calls and storing local variables during execution.</li>
</ul>
</p>

<h3>Memory Page States</h3>

<p>After explaining all of this, let's proceed to memory paging.<br /><br />

What is Virtual Memory Page? Virtual memory relies on the concept of memory paging, which involves dividing memory into fixed-size chunks called "pages." On most modern systems, these pages are typically 4KB in size, though the size can vary depending on the architecture and configuration. Memory paging allows the operating system to manage memory more efficiently by loading and unloading pages between physical memory (RAM) and disk storage as needed.<br /><br />

The following picture depicts a high-level overview of how virtual memory is mapped to physical memory using paging. Also, it is important to note that this picture is from the book <a href="https://www.amazon.com/Windows-Internals-Part-architecture-management/dp/0735684189">Windows Internals, Part 1</a> by <a href="https://twitter.com/zodiacon">Pavel Yosifovich</a>:</p> 

<img src="/assets/img/post-img/17-04-2024/Windows-Internal_mapping-Memory.png" class="post-images" alt="Mapping-Memory-Windows-Internals-Book" height="500" weight="500">

<p>According to <a href=" https://maldevacademy.com">MalDev Academy</a>, a great platform that I totally recommend to anyone who wants to enhance their malware developing skills, there are three paging states:<br />

<ul>
	<li>
		<b>Free</b>: A page of memory that is currently not allocated to any active process or stored data and is available for use.<br />
	</li>
	<li>
		<b>Reserved</b>: A page of memory that has been allocated by the operating system but is not currently in use by any active process.<br />
	</li>
	<li>
		 <b>Committed</b>: A page of memory that has been allocated and is actively being used by a process.
	</li>
</ul>
</p>

<h3>Memory Page Protections</h3>

<p>This section is specifically about the memory page state called <b>committed</b>. The most popular memory page protection options are:<br />
<ul>
    <li><b>PAGE_NOACCESS</b>: Specifies that a memory page cannot be accessed by any process. This means that attempting to read from or write to the memory page will result in an access violation error.</li>
    <li><b>PAGE_EXECUTE_READWRITE</b>: Specifies that a memory page can be executed from and also read from and written to by a process. This means that the code within the memory page can be executed as instructions, and data within the page can be both read from and written to.</li>
    <li><b>PAGE_READONLY</b>: Specifies that a memory page can be read from but not written to or executed. This means that data within the memory page can be read by a process, but attempts to modify the data or execute code from the page will result in access violation errors.</li>
</ul></p>

<p>However, if you want to find more memory page protection options, you can visit <a href="https://learn.microsoft.com/en-us/windows/win32/memory/memory-protection-constants">Microsoft's official website</a>.<br /><br /></p>

<h3>Memory Protections</h3>

<p>To protect against exploits and attacks, modern operating systems have built-in memory protections such as:
<ul>
    <li><b>Data Execution Prevention (DEP)</b>: DEP works by marking certain memory pages as non-executable, meaning that code cannot be executed from those pages.</li>
    <li><b>Address space layout randomization (ASLR)</b>: ASLR works by randomly positioning the memory layout of processes, making it difficult for attackers to predict the memory addresses of system components or injected code.</li>
</ul>
</p>

<h3>Memory in Action</h3>

<p>Let's dive into the technical world of memory allocation and see how it works in action!</p>

<div style="display: flex; justify-content: center;">
    <iframe src="https://giphy.com/embed/4Nq6L6m836paOHEjxy" width="480" height="350" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>
</div>

<p>It is important to note that the following example was inspired by <a href="https://maldevacademy.com">Maldev Academy</a>.</p>

<p>First of all, we need to know that there are several methods to allocate memory during runtime (heap). Some of them are:</p>

```
#include <stdio.h>
#include <windows.h>

int main() {
    HANDLE hFile = INVALID_HANDLE_VALUE;
    LPCWSTR filePath = L"C:\\Users\\nickvourd\\Desktop\\nickvourd.txt";

    hFile = CreateFileW(filePath, GENERIC_ALL, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);

    if (hFile != INVALID_HANDLE_VALUE) {
        wprintf(L"[+] File '%s' created successfully.\n", filePath);
        CloseHandle(hFile);
    } else {
        wprintf(L"[-] CreateFileW API Function Failed with Error: %d\n", GetLastError());
        return -1;
    }
}
```

<!-- add the button!-->
<div>
<applause-button style="width: 58px; height: 58px;" color="#5d4d7a" url="https://nickvourd.github.io/an-introduction-to-windows-memory-management/"/>
</div>
