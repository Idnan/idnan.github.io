---
layout: post
title: gRPC with NodeJs and Typescript
comments: true
---

Recently I got a chance to work with gRPC in NodeJs. Basically gRPC is an alternate of REST. gRPC works on HTTP2 and provides a-lot of benefits over the REST API's.

Implementing gRPC with NodeJs is simple. But there's an issue of no autocomplete, type hinting and also you have to do a-lot of work to just create client and server. And these issues can be solved by using some plugins that will generate the typescript definitions against `proto` files and than we can use these typescript files in our app to define the service definitions and creating server/client.

We will create a greeter gRPC service that will accept your name and in reply will greet you like `Hi, Adnan`

Let's begin by creating and empty project

```shell
mkdir ts-grpc
```

Consider the diagram below and create a similar structure

<script src="https://gist.github.com/Idnan/ce5669e2cdf133e8eb9f5dc8121de5d4.js"></script>

Now lets install the dependencies
```shell
npm init -y
npm install grpc google-protobuf dotenv
npm install typescript @types/node @types/google-protobuf @types/dotenv --save-dev
```

Initialize typescript so that later on we can compile the project
```shell
npx tsc --init
```

After initializing typescript replace `tsconfig.json` with below
<script src="https://gist.github.com/Idnan/48d0ec2a2743aa2db81a6d275f379a6c.js"></script>

Open `greeter.proto` and replace with below. It create one service `SayHello` that accepts requests of type `HelloRequest` with one field `name` and gives response of type `HelloResponse`. You can read about the `proto` file syntax here [https://developers.google.com/protocol-buffers/docs/proto](https://developers.google.com/protocol-buffers/docs/proto) 
<script src="https://gist.github.com/Idnan/0fb71d23bb3ca8b0a1cf76065c87db8c.js"></script>

Now let's generate the typescript definitions against the gRPC service by using the proto file. So to generate the typescript we need some kind of compiler that will translate the `greeter.proto` to typescript definition. Here's the dependencies that we have to install in order to compile `proto` file
```shell
npm install grpc-tools grpc_tools_node_protoc_ts --save-dev
```

After installing the dependencies now we have to write a script that will loop over the all the available `proto` files in the `src/proto/**/*.proto` and compile them. For that open `protoc.sh` and replace with below
<script src="https://gist.github.com/Idnan/d521870ef2fcd0e406fa7ab521b3407f.js"></script>

Now let's run it. In order to run it we have to change the permission of our `protoc.sh` to be executable. Therefore run below command
```shell
sudo chmod +x ./scripts/protoc.sh
``` 
Now run it and it will generate following files 2 typescript definition files and there respective javascript files in `src/proto/greeter`. Each time you will update your proto file you have run this script to generate new typescript definitions.
```
./scripts/protoc.sh
```

Now after comes a trick. After generating the typescript definitions where have to tell our typescript compiler to include these files while compiling phase. So open `src/proto/index.ts` and replace with below.
<script src="https://gist.github.com/Idnan/847086df16203c03a5ccd4d4332316ae.js"></script>

Now each time you create new proto and after generating there typescript definitions you have to update this file and import there respective definitions here.

Let's write our greeter handler to define our `SayHello` service so open `src/handlers/greeter.ts` and replace it with below. It creates and handler that will handle all the requests against the greeter service. So later on if you will add new rpc services in your proto file `greeter.proto`, you have to define there respective definition here in this handler.
<script src="https://gist.github.com/Idnan/a467d9028dbbae5b34e8e98287c60316.js"></script>

See `sayHello` it has the implementation of our `SayHello` rpc service. Which is getting name from the request `HelloRequest` and provides response of type `HelloResponse`

Now let's write a gRPC server. Open `src/server.ts` and replace with below
<script src="https://gist.github.com/Idnan/bc54353b41d7d5883bb5a135aed59e69.js"></script>

The code is self explanatory we are just create an instance of gRPC server and then registering our greeter service handler, than assigning host/port and running it.

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

After building, let's start the gRPC server. Running the below command will give you following output saying `gRPC listening on 50051`. Which means everything went success and we can test our greeter server.
```shell
npm run start
```

To test the greeter server I am not going to write a client. I am going to use [BloomRPC](https://github.com/uw-labs/bloomrpc) which is a gui client to test rpc services. Follow the [installation guide](https://github.com/uw-labs/bloomrpc#installation), import the `greeter.proto` file, update the url to be `127.0.0.1:50051` and click the PLAY icon and you will see a similar output like in the image below. 
<figure align="center"> 
    <img src="https://i.imgur.com/bsvIC1U.png" style="max-width:635px;"/>
</figure>

Check out the repo here for the [source code](https://github.com/Idnan/ts-grpc-example).

And that wraps it up for this article. Feel free to leave your feedback or questions in the comments section.
