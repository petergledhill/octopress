---
layout: post
title: "Basics of Multi-player in Node.js : Part 1"
date: 2013-11-28 13:03:58 +0000
author: Peter Gledhill
comments: true
categories: [Javascript, Node.js, HTML5, Websockets, Games]
---

## Introduction

The challenge for multi-player gaming is to ensure that all players see the same game state at roughly the same time. The game must also be playable on a slow connection and cheating should not be trivial. 

In many multi-player games this is achieved by having a server which all clients connect to. The server maintains the correct game state and updates all the clients to keep them in sync.  This is called a client with authoritative server model and is generally used for first person shooters (Real time strategy games often use a peer-to-peer model with the [lockstep protocol](http://en.wikipedia.org/wiki/Lockstep_protocol)).

### Dumb client
Doom was one of the first games to use this model and supported up to 4 players at once.  In Dooms networking model the client computers are dumb, the client feeds instructions to the server and the server responds with the new game state.  

For example the player presses the right arrow, the instruction is sent to the server and the server sends back a message saying 'your position is now X,Y'.  This method works well when the game is played over a local area network (LAN) as there is little delay between the client and the server.

Over the Internet this method doesn't work.  By the time the client sends the message to the server and receives back the new state several hundred milliseconds may have passed. This results in very jerky movement. 

## Client-side prediction

The solution to this is to calculate the result of the move both locally and on the server. When a key is pressed, the client calculates the result locally and updates the position immediately.  The move and the client result are then sent to the server.  

The server recalculates the result and either acknowledges the clients good move or sends a correction.  This way the client can react immediately to input and the server can maintain an authoritative state - the best of both worlds. 

> A move would be an action such as 'move right'. In a 2D game the result of this move might be a new position like x:10, y:0.  It's important to note that the move is a relative action and the result is an absolute.

There are a few things to mention with this model:

### Determinism

The game logic must be [deterministic](http://en.wikipedia.org/wiki/Deterministic_system).  This means each time we put in the same input we must get the same result back; there can not be a random element to the logic. If there was a random element then the client and server would often disagree about the result and the client would constantly need correcting, and the client-side prediction would have no benefit. 

### Snapping
After the client sends a move to the server it will continue to generate new moves.  By the time the client receives a response from the server several hundred milliseconds will have past and the client will be receiving a correction / confirmation of a move which happened in the past.  In the case that a correction is required this causes a problem.  If we update to the servers position the client will appear to jump back in time or 'snap' to a new location.  

To get around this problem the client can store all the moves which are made, when a correction is made to a move in the past we can then replay all future moves.  This works because the moves a relative actions such as 'move left' and so can be reapplied from the corrected position.

## Breakdown

### On the client
1. On the update loop - your keypresses are captured and recorded as a 'move'. e.g. 'move left' .
2. The result of the move is calculated and the position of your avatar is updated.
3. The move is stored along with the time it was created.
4. The move is sent to the server along with the result of the move.

### On the server
1. The server receives the move and immediately calculates the result of the move on the avatar.
2. The result is checked against the clients result.  
    1. If the results match then the server sends a message to the client to say the result was good.
    2. If the server disagrees about the result then it informs the client of what the result should have been.

### Back on the client
Note: While the server was calculating the result, the client will have generated new moves and which it will have sent to the server.

1. If a good move is acknowledged then no action is required.
2. If a move is corrected then update the position based on the result from the server.  Any moves which have occurred since this corrected move must be calculated again on the client.


{% codeblock game-client.js lang:javascript %}
// this function is called roughly every 15 ms
update: function(){

  // get current client time
  var t = this.time();

  // get move based on inputs
  // this will be an array of direction to move such as ['l', 'u'] for left and up
  var move = input.get_move();

  // save move
  this.me.moves.add(move, t);

  // apply the move locally
  // we use a fixed delta time because we have physics based movement
  // physics delta is hardcoded to 0.015 - every 15 ms
  this.move_autonomous( move, this.data.physics_delta );

  // inform the server of our move
  this.server_move(t, move, this.me.accel, this.me.pos);
},
{% endcodeblock %}

{% codeblock game-server.js lang:javascript %}

server_move: function(client_id, player, time, move, client_accel, client_pos){
  player.controller.apply_move(move, this.data.physics_delta);

  if(player.pos.equals(client_pos)){
    this.transport.good_move( client_id, time );
  }
  else{
    // this be passed to move_corrected in game-client
    this.transport.ajust_move(
      client_id,
      {
        t: time,
        p: player.pos.toObject(),
        v: player.vel.toObject()
      }
    );
  }
},

{% endcodeblock %}

{% codeblock game-client.js lang:javascript %}
move_corrected: function(time, pos, vel){
  // update position / velocity to match server values for this move
  this.me.pos.set( pos );
  this.me.vel.set( vel );
  this.me.moves.clear_from_time( time );

  // rerun all moves which were made after this one        
  var moves = this.me.moves.all();
  var l = moves.length;

  for(var i = 0; i < l; i++){
    this.move_autonomous( moves[i].move, this.data.physics_delta );
  }
},

{% endcodeblock %}