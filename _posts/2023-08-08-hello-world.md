---
layout: post
title: "Hello World"
image: ""
date: 2023-08-08 00:06:02
tags:
  - introduction
  - base64

description: ""
categories:
  - introduction
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
	
	.help-img {
		max-width: 75%;
	}

	.post-content img { 
		margin: 1.875rem auto;
		display: block;
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

<p>VGhpcyBpcyBvbmx5IHRoZSBiZWdpbm5pbmchISE=</p><br/>

<div align="center">
<!-- add the button!-->
<applause-button style="width: 58px; height: 58px;" color="#5d4d7a" url="https://nickvourd.github.io/hello-world"/>
</div>

