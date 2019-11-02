---
layout: post
title: gRPC with Node.js and TypeScript
comments: true
---

The quest for optimizing the communication over the network has been going on for ages. 1990s brought us TCP/IP protocols for networking and we saw the rise of technologies such as CORBA, DCOM, and Java RMI. With the evolution of web in 2000s, HTTP started to become the defacto for communication and people started to use XML over HTTP for the communication and we saw this combination giving boom to SOAP and WSDL which provided language agnostic communication between the systems. Moving forward, as the web evolved, JavaScript and JSON started to become popular and JSON started to *replace* XML as the preferred wire-transfer format and resulted in an unofficial standard called REST. It did not completely replace SOAP but most of the developer focus shifted towards REST while the enterprise applications and corporates which require strict adherence to standards and schema definitions, stayed with SOAP.

The recent hotness in the industry is gRPC which is a lightweight communication protocol from Google with a support for over a dozen languages. I recently got the chance to work with gRPC in Node.js – this article is a brief introduction to gRPC and how to use it with Node.js and TypeScript.

## gRPC and Proto Files

gRPC is basically a high performance RPC framework created by Google. It runs over HTTP2 and it is the default protocol that is used instead of JSON on the network. By default gRPC uses [protocol buffers](https://developers.google.com/protocol-buffers/) as IDL (Interface Definition Language) to define the structure for the service interface and structure for the payload messages. Using the IDL we can generate type safe DTO's (Data transfer object) and client server implementations in multiple languages (like, Go, PHP, Ruby, Python, Objective-C, Node.js, Java, C, C#, Java). The types are converted to binary format when calling the remote procedures. So a server generated in C++ can communicate transparently with a client written in Java or Ruby. 

The image given below gives an conceptual overview.

<figure align="center"> 
    <img src="https://i.imgur.com/FqWxYpZ.png" style="max-width:500px;"/>
</figure>

One of the biggest difference between gRPC and REST is the format of the payload. REST messages typically contain JSON. There's no defined interface for the request and response so it's safe to say that you can send anything in request and response. Where gRPC on the other hand uses defined interfaces for request and response that are defined using protocol buffers. This gives you a huge win over the REST API's and calling the services in gRPC is just like calling a local function. Also in terms of [benchmark gRPC is much faster than REST](https://medium.com/@EmperorRXF/evaluating-performance-of-rest-vs-grpc-1b8bdf0b22da).

So enough with the talk let's start with the actual work. We will create a greeter gRPC service that will accept your name and in reply will greet you like `Hi, Adnan`

## Setting up a Project

Let's begin by creating and empty project

```shell
mkdir ts-grpc
```
Now go inside the directory and create the directory structure similar to the one given below

<script src="https://gist.github.com/Idnan/ce5669e2cdf133e8eb9f5dc8121de5d4.js"></script>

Now lets install the dependencies

```shell
npm init -y
npm install grpc google-protobuf dotenv
npm install typescript @types/node @types/google-protobuf @types/dotenv --save-dev
```

Here's the explanation of the packages that we have installed

- [grpc](https://github.com/grpc/grpc-node) to use gRPC with Node.js
- [google-protobuf](https://github.com/protocolbuffers/protobuf/tree/master/js) to use Protocol Buffers (.proto) with javascript
- [dotenv](https://github.com/motdotla/dotenv) to load environment variables from .env
- TypeScript and typedefinitions to help in development

Initialize typescript so that later on we can compile the project

```shell
npx tsc --init
```

After initializing typescript add the below content to your `tsconfig.json` file

<script src="https://gist.github.com/Idnan/48d0ec2a2743aa2db81a6d275f379a6c.js"></script>

Now open the `greeter.proto` file and add the below content in it

<script src="https://gist.github.com/Idnan/0fb71d23bb3ca8b0a1cf76065c87db8c.js"></script>

It creates one service `SayHello` that accepts requests of type `HelloRequest` with one field `name` and gives response of type `HelloResponse`. You can read more about the `proto` file syntax here [https://developers.google.com/protocol-buffers/docs/proto](https://developers.google.com/protocol-buffers/docs/proto) 

## Generating TypeScript definitions

Now let's generate the typescript definitions against the gRPC service by using the proto file. To generate the typescript we need some kind of compiler that will translate the `greeter.proto` to typescript definition. Here's the dependencies that we have to install in order to compile the `proto` file

```shell
npm install grpc-tools grpc_tools_node_protoc_ts --save-dev
```

Here we installed few more dev dependencies

- [grpc-tools](https://github.com/grpc/grpc-node) generate javascript files for the proto files
- [grpc_tools_node_protoc_ts](https://github.com/agreatfool/grpc_tools_node_protoc_ts) generate corresponding typescript d.ts codes according to js codes generated by grpc-tools

<br/>

After installing the dependencies now we have to write a script that will loop over the all the available `proto` files in the `src/proto/**/*.proto` and compile them. For that open `protoc.sh` and replace with below
<script src="https://gist.github.com/Idnan/d521870ef2fcd0e406fa7ab521b3407f.js"></script>

Now we need to make this script executable so that we can actually use it. Run the below command to make it executable

```shell
sudo chmod +x ./scripts/protoc.sh
``` 

Now run the script and it will generate our typescript definition files and there respective javascript files in `src/proto/greeter`. Each time you will update your proto file you have run this script to generate new typescript definitions.
```
./scripts/protoc.sh
```

After the typescript definitions have been generated, we need to tell our typescript compiler to include these files during the compilation phase. In order to do that, open `src/proto/index.ts` and replace with below
<script src="https://gist.github.com/Idnan/847086df16203c03a5ccd4d4332316ae.js"></script>
You need to generate the typescript definitions and update this file whenever you create new proto files.

## Creating handlers
Let's write our greeter handler to define our `SayHello` service so open `src/handlers/greeter.ts` and replace it with below. It creates and handler that will handle all the requests against the greeter service. So later on if you will add new rpc services in your proto file `greeter.proto`, you have to define there respective definition here in this handler.
<script src="https://gist.github.com/Idnan/a467d9028dbbae5b34e8e98287c60316.js"></script>

See `sayHello` it has the implementation of our `SayHello` rpc service. Which is getting name from the request `HelloRequest` and provides response of type `HelloResponse`

## Writing the Server
Now let's write a gRPC server. Open `src/server.ts` and replace with below
<script src="https://gist.github.com/Idnan/bc54353b41d7d5883bb5a135aed59e69.js"></script>

As you can already guess from the code, we are just creating an instance of gRPC server, registering our greeter service handler and then just starting the server.

## Testing our Implementation
Now let's test it. So to test it we have to update our `package.json` `scripts` section. So add following.
```json
...
"scripts": {
    "build": "npx tsc --skipLibCheck",
    "start": "npx tsc --skipLibCheck && node ./dist/server.js"
}
...
```

Now open terminal and run the build command. This will create a `dist` folder and will compile typescript to generate javascript files. 
```shell
npm run build
```

Once the build is finished, let's test our implementation by running the server. Run the command below to start the server
```shell
npm run start
```
If everything goes well, you should see the message `gRPC listening on 50051`

In order to test our server, I am going to use [BloomRPC](https://github.com/uw-labs/bloomrpc) which is a GUI client to test RPC services.

Follow the [installation guide](https://github.com/uw-labs/bloomrpc#installation), import the greeter.proto file, update the URL to be `127.0.0.1:50051` and click the PLAY icon and you will see the output similar to the one given below
<figure align="center"> 
    <img src="https://i.imgur.com/bsvIC1U.png" style="max-width:635px;"/>
</figure>

And that wraps it up. You can find the source code from the article [here](https://github.com/Idnan/ts-grpc-example).

Feel free to leave your feedback or questions in the comments section below.
