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

By utilizing virtual memory addressing and allowing the operating system's Memory Management Unit (MMU) to handle the mapping of virtual addresses to physical memory addresses, modern operating systems can efficiently manage physical memory utilization. This approach enables the system to allocate memory resources dynamically based on the current needs of processes and applications, maximizing the use of available physical memory and minimizing waste. Additionally, virtual memory addressing facilitates memory protection and isolation between processes, enhancing system stability and security (We will discuss memory protections later in this article).</p><br /><br />

<h3>Memory Structures</h3>

<p>Before proceeding to technical details and protections regarding virtual memory, it is important to explain the basic virtual memory structures.<br /><br />

For years, I've been hearing about terms like stack, heap, and more from hardcore cybersecurity colleagues, and of course, from <a href="https://twitter.com/0xvm">Lovely Uncle Bill</a>. To be honest, all this stuff seemed very confusing to me. So, I started from the basics to understand what they are.<br /><br />

The following picture depicts an overview of the virtual memory layout of a process (x86). Also, it is important to note that this picture inspired by the great blog post <a href="https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/">Exploit writing tutorial part 1 : Stack Based Overflows</a> by <a href="https://twitter.com/corelanconsultn">@corelanconsultn</a>:</p>

<img src="/assets/img/post-img/17-04-2024/Memory-Strucutre.png" class="post-images" alt="Exploit writing tutorial part 1 : Stack Based Overflows (Corelan)" height="500" weight="500">

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
