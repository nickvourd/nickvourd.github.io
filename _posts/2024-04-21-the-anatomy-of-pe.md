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

<p>Examples of PE files can include <b>.exe</b>, <b>.dll</b>, <b>.sys</b>, and <b>.scr</b> files etc. However, I think Sector 7 offers the best introduction to the PE format that I have ever come across!<br /><br />What I mean?<br /><br />Well, the PE structure is very complicated, but with any complex topic, you should change your approach to viewing it!<br/>The following picture shows the original (complex) structure view of PE. It is important to note that this image is from the <a href="https://en.wikipedia.org/wiki/Portable_Executable">Wikipedia</a> article:</p>

<img src="/assets/img/post-img/21-04-2024/Portable-ExecucatbleFormat.svg.png" class="post-images" alt="PE Structure from Wikipedia">

<p>As you can see, the structure of a PE file is really complicated. According to Sector 7, a better approach is to view it as a book. Personally, I think it's a great idea! Let's assume that a PE is a book. This book contains two parts: <b>data</b> and <b>metadata</b>. As for the data, we assume the author's text/content, while for the metadata, we consider the title of the book, the author's name, the publisher's name, the ISBN, the release date, and the table of contents etc.<br /><br />The following picture depicts a custom diagram representing a PE file as a book:</p>

<img src="/assets/img/post-img/21-04-2024/PE-Format-Simple.png" class="post-images" alt="PE-Structure-Custom-Simple" height="700" weight="700">

<p>Another great resource for presenting PE structure is the following image by <a href="https://github.com/corkami">corkami</a>:</p>

<img src="/assets/img/post-img/21-04-2024/pe101.png" class="post-images" alt="PE-Format-Corcami" height="500" weight="500">

<p>If you try to view the image above vertically, you'll see what I mean—it resembles a book. 😉</p>

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

<!-- add the button!-->
<div>
<applause-button style="width: 58px; height: 58px;" color="#5d4d7a" url="https://nickvourd.github.io/the-anatomy-of-pe/"/>
</div>