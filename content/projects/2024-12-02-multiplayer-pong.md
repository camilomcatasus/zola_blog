+++
title = "Multiplayer Pong"
[taxonomies]
tags = ["Game", "Godot", "Multiplayer"]
+++
Game Project to learn about real time multiplayer techniques and strategies

**Goals**:
- [x] Easy Host and Join Using Room Codes
- [x] Implement Reconcilliation
- [ ] Implement Interpolation
- [ ] Implement Lag Compensation
- [ ] 2-8 Player Multiplayer
- [ ] Player Spectating
- [ ] Map selection

## Why
I am trying to write a multiplayer game and was struggling with wrapping my head around everything. 
I decided it would be better to try and implement *everything* in a simpler game. 
So I can have a strong foundation of understanding when I'm programming my real game.

## How everything works
So I mostly followed the [this site](https://www.gabrielgambetta.com/client-server-game-architecture.html) for guidance on how to implement multiplayer myself.
Step by step, these are those most important things I did.

### 1. Decide client server architecture

I had really two choices for how client server would work here. 
Either I could have dedicated servers where the game logic would be run separately without the rendering logic, 
or I could have players host the game and other players would connect to them through a relay server. 
Another alternative would be to use [WebRTC](https://webrtc.org) to get direct P2P, 
but I didn't want to have to deal with any of the complexity.

I decided to go for the client-host architecture with a [relay server](/projects/easy-relay-server). 
This way I could run a lot of connections off of my single vps if I need to. 

### 2. Write most of the communication protocol
First we have to encode message we're sending into a byte array that we can send to our peer.
For that I created a class called Encoder that we can utilize to encode messages on the fly.
The class has various functions that we can use that let us encode whatever it is we need to.
If we need to encode a new data type, we can just add a new function to the class. 
```gd
extends Object
class_name Encoder 
var packet: PackedByteArray;
func _init():
	packet = PackedByteArray();

func encode_u8(u8: int):
	var temp_packet = PackedByteArray()
	temp_packet.resize(1);
	temp_packet.encode_u8(0, u8);
	packet.append_array(temp_packet);

func encode_u16(u16: int):
	var temp_packet = PackedByteArray()
	temp_packet.resize(2);
	temp_packet.encode_u16(0, u16);
	packet.append_array(temp_packet);

func encode_u32(u32: int):
	var temp_packet = PackedByteArray()
	temp_packet.resize(4);
	temp_packet.encode_u32(0, u32);
	packet.append_array(temp_packet);
```

Next we have to decode our messages on the receiver side, for that I created a class called Decoder. 
It has a byte index variable used to track which byte we're on. We can then use the decode functions Godot has for the PackedByteArray.


```gd
extends Object
class_name Decoder

var packet: PackedByteArray;
var byte_index = 0;

func _init(new_packet: PackedByteArray):
	packet = new_packet;
	
func decode_u8() -> int:
	var decoded_u8 = packet.decode_u8(byte_index);
	byte_index += 1;
	return decoded_u8;

func decode_u16() -> int:
	var decoded_u16 = packet.decode_u16(byte_index);
	byte_index += 2;
	return decoded_u16;
	
func decode_u32() -> int:
	var decoded_u32 = packet.decode_u32(byte_index);
	byte_index += 4;
	return decoded_u32;
	
func decode_u64() -> int:
	var decoded_u64 = packet.decode_u64(byte_index);
	byte_index += 8;
	return decoded_u64;
```

### 3. Getting a Connection
After a connection is established, every peer needs to identify itself to the host with some unique identifier because the host is 
getting all of its information from one websocket connection and has no way to differentiate peers otherwise. 
Also, the host broadcasts all of their messages because of the way the relay works, so all of our messages cannot give information that will let one player impersonate another.
After a connection is made a peer sends a new connection message with two unique identifiers. The first identifier is used to inform the peer that the connection message was accepted, 
the second identifier is what the peer will use to identify all their messages. Could I have done something more complicated like Diffie-Hellman? Probably, I just figured this was easier and good enough.

### 4. Updating Positions
Now comes the hard part. Updating the positions of people who will be seeing those updates moments after they occur. The server will see the clients movement milliseconds after the client sends input,
and the client will only know if they're inputs have been accepted milliseconds after the host receives the input. In a real time multiplayer game where position and velocity are bounded, 
players should not be an authority on their own position because it would enable cheating. So instead players send the difference between their old position and their new one to the server, 
and the server at that moment determines if that offset/message is valid. Now, there are a lot of questions that this leads to. For example, when do we update the clients positions visually?

#### Client Prediction
There are two options for updating the client's position on the client's machine. The first is the simplest and used by a lot of games actually, wait for the server to send us the updated position,
then update the client's position. This works perfectly for multiplayer turn based games like Civ where the player won't mind the delay. It can also work for real time games like Starcraft,
however, it's not ideal. Our other options is to predict where our client will be client-side, run all of our game logic on the players input, then place our player at its resulting position.
Now, if the server responds to one of our position updates with a client position that doesn't match our predicted position, we can just set it to the server position. This leads to its own problem,
being out of sync with server. If we send a message of our updated position and then we get a response from the server for our last updated position, 
we would have to set our position to the one we got from the server because the two positions differ.

#### Reconcilliation
The way to fix the issues with client side prediction is to reconcile the new position. We can either send a timestamp or a counter with our position data, 
now the server can send its response with the last timestamp/count it received from the client. Finally, we have to keep a record of all our unacknowledged position differences we've sent, 
then whenever we receive a position response from the server we look at the timestamp/count in the message and get rid of all our stored position differences up until the first unacknowledged position difference.
Next we reapply the unacknowledged position differences to the position the server sent, and that is our new *true* position. Moving the player client-side looks really smooth now, 
but things seem jittery when looking at the one player from another player's perspective.

#### Interpolation
To fix jittering we can


