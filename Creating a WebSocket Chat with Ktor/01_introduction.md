# Introduction

In this hands-on, we will learn how to create a simple chat application which uses WebSockets. We will develop both the client and server application using [Ktor](https://ktor.io/) – an asynchronous Kotlin framework for creating web applications.

## What we will build

Throughout this tutorial, we will implement a simple chat service, which will consist of two applications:

- The **chat server application** will accept and manage connections from our chat users, receive messages, and distribute them to all connected clients.
- The **chat client application** will allow users to join a common chat server, send messages to other users, and see messages from other users in the terminal.

![app_in_action](./assets/app_in_action.gif)

For both parts of the application, we will make use of Ktor's support for [WebSockets](https://ktor.io/docs/servers-features-websockets.html). Because Ktor is both a server-side and client-side framework, we will be able to reuse the knowledge we acquire building the chat server when it comes to building the client.

After completing this hands-on, you should have a basic understanding of how to work with WebSockets using Ktor and Kotlin, how to exchange information between client and server, and get a basic idea of how to manage multiple connections at the same time.

## Why WebSockets?

WebSockets are a great fit for applications like chats or simple games. Chat sessions are usually long-lived, with the client receiving messages from other participants over a long period of time. Chat sessions are also bidirectional – clients want to send chat messages, and see chat messages from others.

Unlike regular HTTP requests, WebSocket connections can be kept open for a long time and have an easy interface for exchanging data between client and server in the form of frames. We can think of frames as WebSocket messages which come in different types (text, binary, close, ping/pong). Because Ktor provides high-level abstractions over the WebSocket protocol, we can even concentrate on text and binary frames, and leave the handling of other frames to the framework.

WebSockets are also a widely supported technology. All modern browsers can work with WebSockets out of the box, and frameworks to work with WebSockets exist in many programming languages and on many platforms.

Now that we have confidence in the technology we want to use for the implementation of our project, let’s start with the set up!
