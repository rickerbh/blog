---
layout: post
title: "Refactoring Hard to Test Code"
date: 2016-08-25 20:47:08 +1000
comments: true
categories: 
---
As part of the [Apprenticeship](https://hamishrickerby.com/2016/08/20/software-apprenticeships/), I need to create an [HTTP server](https://github.com/rickerbh/http_server_java). The purpose of the activity is to create a relatively complex, real-world system with TDD, following SOLID/clean code principles. It's going well so far. There is a FitNesse spec for acceptance tests that the server needs to pass, and I have to create it in Java. I'm getting there.

However, I recently found that I was having to jump through some hoops to test my socket connection completion handler, and my mentor suggested this might be a code smell, and that I should look at refactoring the code to make it easier to test. _As an aside, I'm using the "new" non-blocking IO socket classes rather than the traditional socket classes, so that multithreading should be simpler as I shouldn't need to manage threadpools._

The design I had come up with for handling HTTP requests and responses was quite tightly tied to the Java [`CompletionHandler`](https://docs.oracle.com/javase/7/docs/api/java/nio/channels/CompletionHandler.html) generic interface, and specifically the `completed` method that receives [`AsynchronousSocketChannel`](https://docs.oracle.com/javase/7/docs/api/java/nio/channels/AsynchronousSocketChannel.html)s. Testing it, by generating fake `AsynchronousSocketChannels` felt super hacky. The code for dealing with processing requests and responses was also tied in with the channel handling code, so the responsibility of the class was not very clear.

```java
public class HTTPCompletionHandler implements CompletionHandler<AsynchronousSocketChannel, Void> {
    ResponseFactory responseFactory;
    AsynchronousServerSocketChannel listeningChannel;

    public HTTPCompletionHandler(String rootDirectory, AsynchronousServerSocketChannel listeningChannel) {
        responseFactory = new ResponseFactory(rootDirectory);
        this.listeningChannel = listeningChannel;
    }

    @Override
    public void completed(AsynchronousSocketChannel ch, Void attachment) {
        listeningChannel.accept(null, this);
        String requestText = extractRequestText(ch);

        Request request = new Request(requestText);
        Response response = responseFactory.makeResponse(request);

        sendResponse(ch, response);

        closeChannel(ch);
    }

@Override
    public void failed(Throwable exc, Void attachment) {

    }

    private String extractRequestText(AsynchronousSocketChannel ch) {
        ByteBuffer buffer = ByteBuffer.allocate(8192);
        byte[] requestBytes = null;
        try {
            int bytesRead = ch.read(buffer).get(20, TimeUnit.SECONDS);
            requestBytes = new byte[bytesRead];

            if (bytesRead > 0 && buffer.position() > 2) {
                buffer.flip();
                buffer.get(requestBytes, 0, bytesRead);
                buffer.clear();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            return new String(requestBytes);
        }
    }

    private void sendResponse(AsynchronousSocketChannel ch, Response response) {
        ch.write(ByteBuffer.wrap(response.getBytes()));
    }

    private void closeChannel(AsynchronousSocketChannel ch) {
        if (ch.isOpen()) {
            try {
                ch.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    
}
```

## Time to refactor

Below are a series of steps that I went through to refactor this class to ensure single-responsibility, and make it significantly easier to test the functionality. 

_Links are provided at the end to github for the steps as commits. Github also contains the tests. 100% TDD baby!_

### Create interfaces

First of all, I needed to break the dependency between the HTTP Request/Repsonse code, and the `AsynchronousSocketChannel` code. I introduced an abstraction for reading and writing data.

```java
public interface ByteReader {
     byte[] read();
}
```

```java
public interface ByteWriter {
    void write(byte[] bytes);
}
```

I could have put these in a single interface, but splitting them gives easier support for different implementation models, such as reading from a socket, and writing out to a file.

### Concrete implementations

Then, implement classes that support these interfaces that use the `AsynchronousSocketChannel` to read and write from.

```java
public class AsynchronousSocketChannelReader implements ByteReader {

    AsynchronousSocketChannel channel;

    public AsynchronousSocketChannelReader(AsynchronousSocketChannel channel) {
        this.channel = channel;
    }

    @Override
    public byte[] read() {
        ByteBuffer buffer = ByteBuffer.allocate(8192);
        byte[] requestBytes = null;
        try {
            int bytesRead = channel.read(buffer).get(20, TimeUnit.SECONDS);
            requestBytes = new byte[bytesRead];

            if (bytesRead > 0 && buffer.position() > 2) {
                buffer.flip();
                buffer.get(requestBytes, 0, bytesRead);
                buffer.clear();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            return requestBytes;
        }
    }
}
```

```java
public class AsynchronousSocketChannelWriter implements ByteWriter {
    private AsynchronousSocketChannel channel;

    public AsynchronousSocketChannelWriter(AsynchronousSocketChannel channel) {
        this.channel = channel;
    }

    @Override
    public void write(byte[] bytes) {
        channel.write(ByteBuffer.wrap(bytes));
    }
}
```

### Integrate these back in

The next step is to start to use these within the `HTTPCompletionHandler` class. The bulk of the `extractRequestText` and `sendResponse` functions can also be dropped from the `HTTPCompletionHandler`.

```java
...

    private String extractRequestText(AsynchronousSocketChannel ch) {
        ByteReader reader = new AsynchronousSocketChannelReader(ch);
        return new String(reader.read());
    }

...

    private void sendResponse(AsynchronousSocketChannel ch, Response response) {
        ch.write(ByteBuffer.wrap(response.getBytes()));
        ByteWriter writer = new AsynchronousSocketChannelWriter(ch);
        writer.write(response.getBytes());
      }

...

```

The code in `sendResponse` at this interim step has grown longer, but what we've actually done here is enable a very clean break in the dependency between the HTTP request/response processing, and the `AsynchronousSocketChannel`, as we'll see in the next step.

### Extract HTTP Request/Response handling

Now, we need to remove the HTTP Request and Response handling out of this completion handler. The completion handler will soon have a very specific responsibility in providing an interface adapter between `AsynchronousSocketChannel`s and processing that data. The reading, writing, or orchestration of the data flow will no longer be part of that classes responsibility.

We're creating a new class called `HTTPHandler`. This class will take the configuration required (a directory) on instantiation, and for each request it needs to process it'll receive a `ByteReader` and `ByteWriter`.

```java
public class HTTPHandler {
    ResponseFactory responseFactory;

    public HTTPHandler(String rootDirectory) {
        responseFactory = new ResponseFactory(rootDirectory);
    }

    public void run(ByteReader reader, ByteWriter writer) {
        String requestText = extractRequestText(reader);

        Request request = new Request(requestText);
        Response response = responseFactory.makeResponse(request);

        sendResponse(writer, response);
    }

    private String extractRequestText(ByteReader reader) {
        return new String(reader.read());
    }

    private void sendResponse(ByteWriter writer, Response response) {
        writer.write(response.getBytes());
    }
}
```

The beauty in creating a class like this where the reader and writer are abstract and provided to it, is that it's not dependent on any particular type of Socket, or Stream, of Buffer. 

### Fake Reader and Writer for testing

The interface for these two types is very simple, and very, very, very easy to create Fake versions of to test.

```java
public class FakeReader implements ByteReader {

    private byte[] byteData;

    public FakeReader(String data) {
        byteData = data.getBytes();
    }

    @Override
    public byte[] read() {
        return byteData;
    }
}

public class FakeWriter implements ByteWriter {

    private byte[] byteData;

    @Override
    public void write(byte[] bytes) {
        byteData = bytes;
    }

    public byte[] readWrittenBytes() {
        return byteData;
    }
}
```

### Remove HTTP functionality from CompletionHandler

We can drop the `extractRequestText` and `sendResponse` methods from the `HTTPCompletionHandler` altogether, as these are now provided by the `HTTPHandler`.

The relevant parts of the `HTTPCompletionHandler` will change as below.

```java
public class HTTPCompletionHandler implements CompletionHandler<AsynchronousSocketChannel, Void> {
HTTPHandler handler;
...

    public HTTPCompletionHandler(String rootDirectory, AsynchronousServerSocketChannel listeningChannel) {
        responseFactory = new ResponseFactory(rootDirectory);
        handler = new HTTPHandler(rootDirectory);
        this.listeningChannel = listeningChannel;
        }

    @Override
    public void completed(AsynchronousSocketChannel ch, Void attachment) {
        listeningChannel.accept(null, this);
        
        ByteReader reader = new AsynchronousSocketChannelReader(ch);
        ByteWriter writer = new AsynchronousSocketChannelWriter(ch);
        handler.run(reader, writer);
    
        closeChannel(ch);
    }

...
}

```

### Complete

The `completed` method now only instatiates `ByteReader` and `ByteWriter`s that can handle `AsynchronousSocketChannel`s, and asks the instantiated handler to begin processing with the reader and writers.

The concrete `ByteReader` and `ByteWriter`s now encapsulate the logic for reading from and writing to `AsynchronousSocketChannel`s, rather than this being tied with with dealing with HTTP requests and responses.

The `HTTPHandler` now supports any sort of interface that can conform to `ByteReader` and `ByteWriter` for receiving and sending HTTP content.

And, all classes are small, all methods are small, and due to the interface abstractions, super simple to test.

## End

Refactoring is never really over, but I hope this has been a useful example of how you could refactor a class to

- extract dependencies
- provide a simpler API
- ensure single-responsibility, and 
- make functionality easier to test

If you want to see the whole refactor as a single diff, check out [this commit](https://github.com/rickerbh/http_server_java/commit/d524a31562969bb39351f7cbd0567b9db503d815), or if you're interested in seeing each individual step, check the [last 8 commits on this branch](https://github.com/rickerbh/http_server_java/commits/feature/refactor-completion-handler).
