# Introduction

In this tutorial we'll see how to create a chat application using [Ktor](https://ktor.io). We'll be developing
both the client and server using this framework. 

### What we will build



### Why Ktor?

This could of course bring up a follow-up question:

### Why WebSockets?

WebSockets is a sub-protocol of HTTP. It starts as a normal HTTP request with an upgrade request header, and the connection switches to be a 
bidirectional communication instead of a request response one.

The smallest unit of transmission that can be sent as part of the WebSocket protocol, is a Frame. 
A WebSocket Frame defines a type, a length, and a payload that might be binary or text. Internally those frames might be transparently sent in several TCP packets. We can think of Frames as WebSocket messages. 
Frames could be the following types: 

* Text
* Binary
* Close
* Ping/Pong.

As consumers of Ktor, we'd normally be handling `Binary` and `Text` frames. The others are usually
handled by Ktor. 