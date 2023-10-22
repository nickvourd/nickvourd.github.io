---
layout: post
title: "An overview of Windows architecture & memory management"
image: ""
date: 2023-10-22 10:10:02
tags:
  - Windows Internals
  - Architecture
  - Memory Management 
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
		<p><strong>x86-64</strong>, also known as x64, is an extension of the x86 instruction set, allowing for 64-bit addressing. This extension enables processors to handle larger amounts of memory and perform more complex calculations compared to earlier 32-bit versions of the x86 architecture. Most modern desktops, laptops, and servers utilize the x86-64 architecture.</p>
	</li>
</ul>
<br />



<div align="center">
<!-- add the button!-->
<applause-button style="width: 58px; height: 58px;" color="#5d4d7a" url="https://nickvourd.github.io/win-arch-intro"/>
</div>
