---
layout: post
title: "Basics of multiplayer : local movement"
date: 2013-11-27 13:03:58 +0000
author: Peter Gledhill
comments: true
categories: [Javascript, Node.js, HTML5, Websockets, Games]
---

When playing a multiplayer game you generally give instructions to your avatar through some input device and see the results immediately on screen, we'll call that 'local movement'.  You can also see the movement of other players avatars, we'll call that 'remote movement'. These two types of movement (local and remote) are modelled very differently.  This article focuses on 'local movement'. 

## The Basic idea



### On the client
1. On the update loop - your keypresses are captured and recorded as a 'move'.  
2. The result of the move is calculated and the position of your avatar is updated.
3. The move is stored along with the time it was created.
4. The move is sent to the server along with the result of the move.

### On the server
1. The server received the move and immediately calculates the result of the move on the player.
2. The result is checked against the result which was sent by the client.  
    1. If the results match then the server sends a message to the client to say the result was good.
    2. If the server disagrees about the result then it informs the client of what the result should have been.

### Back on the client
Note: While the server was calculating the result, the client will have generated new moves and which it will have sent to the server.

1. If a good move is acknowledged then no action is required.
2. If a move is corrected then update the position based on the result from the server.  Any moves which have occurred since this corrected move must be calculated again.
