---
layout: post
title: "First steps into real time multiplayer with node.js"
date: 2013-05-13 15:01
author: Peter Gledhill
comments: true
categories: [Javascript, Node.js, HTML5, Websockets]
---
## Tl;Dr
In my spare time I've been playing around with node.js and websockets to learn more about how multiplayer games work.  This is still work in progress and very basic but for anyone interested it's at [https://github.com/petergledhill/basic-realtime-multiplayer](https://github.com/petergledhill/basic-realtime-multiplayer).

## Story mode
I've been wanting to write some serious javascript and am interested in how multiplayer games work.

I was aware that they operate under an authoritative server model; the server is responsible for recording state of the game and then broadcasts it to the clients (this helps to prevent cheating).  However I had no idea how to implement this in practice. 

I came across this [article by Sven Bergström](http://buildnewgames.com/real-time-multiplayer/) which was very helpful in explaining how it all works. It even comes with source code written in Javascript for node.js and the browser.  

Looking at the source I decided I wasn't going to get my head around the intricacies by hacking at it.  So I decided to have a go at writing something similar from scratch.  

The aim is to show the principles of multiplayer gaming through code, it's not a game itself but I hope to fully document the code and write a proper post about what I've learned in the near future.



