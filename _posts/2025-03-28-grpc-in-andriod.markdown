---
layout: post
title:  "gRPC in Android: Building a Bidirectional messaging"
date:   2025-03-28 22:22:00 GMT+2
categories: kotlin android
---
![starting-image](/assets/images/post/grpc_message.webp){: width="512" height="512" }

In this article, we'll explore how to implement a simple chat messaging using gRPC in Android. We'll discuss the pros and cons of gRPC, explain its protocol, and then walk through the code step by step for the Android client logic. The goal is to send and receive text messages in a bidirectional manner.

## What is gRPC?

gRPC (Google Remote Procedure Call) is an open-source RPC framework that enables efficient communication between client and server applications. It uses HTTP/2 for transport, Protocol Buffers (protobuf) for message serialization, and supports multiple programming languages.

### Key Features of gRPC
- **Uses HTTP/2**: Supports multiplexing, reducing latency and improving performance.
- **Supports Streaming**: Enables real-time bidirectional communication between client and server.
- **Code Generation**: Proto files define structured messages, automatically generating client and server stubs.
- **Cross-Language Support**: Allows seamless communication between different programming languages.

## Why Use gRPC for Chat Applications?

### Pros
- **Efficient Communication:** Uses HTTP/2, enabling multiplexing and faster data transfer.
- **Code Generation:** Proto files generate client and server code automatically.
- **Strong Typing:** Ensures data consistency between client and server.
- **Bidirectional Streaming:** Ideal for chat applications with real-time data flow.

### Cons
- **Learning Curve:** Requires understanding of Protobuf syntax and gRPC concepts.
- **Setup Complexity:** Setting up gRPC services and dependencies can be tricky.
- **Limited Browser Support:** Unlike REST APIs, gRPC is less browser-friendly without gRPC-web.

## Step 1: Add Required Dependencies

Before proceeding with implementation, add the necessary dependencies to your `build.gradle` file:

```gradle
implementation("io.grpc:grpc-stub:1.58.0")
implementation("io.grpc:grpc-okhttp:1.58.0")
implementation("io.grpc:grpc-protobuf-lite:1.58.0")
```

## Step 2: Define the Proto File

Create a file named `chat.proto` in your `src/main/proto/` directory with the following content:

```proto
syntax = "proto3";

option java_package = "com.example.yourpackage";
option java_multiple_files = true;

message ChatMessage {
  string senderId = 1;
  string receiverId = 2;
  string message = 3;
  string timeStamp = 4;
}

service ChatService {
  rpc ChatStream(stream ChatMessage) returns (stream ChatMessage);
}
```
Once you re-build the application the proto generated files will be available in `build/generated/source/proto`.

This defines a bidirectional gRPC stream that allows sending and receiving `ChatMessage` objects.

## Step 3: Configure the Build

In your `build.gradle` file, add the following configuration to handle protobuf generation:

```gradle
protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:3.23.2"
    }
    plugins {
        create("grpc") {
            artifact = "io.grpc:protoc-gen-grpc-java:1.56.1"
        }
    }
    generateProtoTasks {
        all().forEach { task ->
            task.builtins {
                create("java") {
                    option("lite")
                }
            }
            task.plugins {
                create("grpc") {
                    option("lite")
                }
            }
        }
    }
}
```

## Step 4: Implement the gRPC Client

Here's the gRPC client code that handles message streaming:

```kotlin
class GRPCClient {

    private val channel: ManagedChannel = ManagedChannelBuilder.forAddress("10.0.2.2", 9090)
        .usePlaintext()
        .build()

    private val stub: ChatServiceGrpc.ChatServiceStub = ChatServiceGrpc.newStub(channel)

    fun createChatSession(): ChatSession {
        val messageFlow = MutableSharedFlow<ChatMessage>(replay = 1)

        val serviceStub = stub.chatStream(object : StreamObserver<ChatMessage> {
            override fun onNext(value: ChatMessage) {
                messageFlow.tryEmit(value)
            }

            override fun onError(t: Throwable) {
                Log.e("Chat", "gRPC Stream Error: ${t.message}", t)
            }

            override fun onCompleted() {
                Log.d("Chat", "gRPC Stream Completed")
            }
        })

        return ChatSession(serviceStub, messageFlow)
    }
}

class ChatSession(
    private val serviceStub: StreamObserver<ChatMessage>,
    private val _messageFlow: MutableSharedFlow<ChatMessage>
) {
    val messageFlow: SharedFlow<ChatMessage> = _messageFlow

    fun sendMessage(senderId: String, receiverId: String, message: String) {
        try {
            serviceStub.onNext(
                ChatMessage.newBuilder().apply {
                    setSenderId(senderId)
                    setReceiverId(receiverId)
                    setMessage(message)
                }.build()
            )
        } catch (e: Exception) {
            Log.e("Chat", "Failed to send message: ${e.message}", e)
        }
    }

    fun close() {
        try {
            serviceStub.onCompleted()
        } catch (e: Exception) {
            Log.e("Chat", "Error closing chat session: ${e.message}", e)
        }
    }
}
```

### Key Points

- **`10.0.2.2`**: Used to connect to the localhost from an Android Emulator. Instead, you can use your server address.
- **`9090`**: Server port.
- **`ChatSession`**: Holds the stub and a `MutableSharedFlow` to manage messages.
- **`StreamObserver`**: Handles data flow — `onNext` for incoming messages, `onError` for handling errors, and `onCompleted` when the stream ends.

## Conclusion

gRPC provides a powerful and efficient way to implement communication for applications. With its strong typing, automatic code generation, and HTTP/2-based transport, it offers several advantages over traditional REST APIs.

In this article, we set up an Android client for a gRPC-based chat system. In the next post, we will implement the server-side logic using Kotlin.


