[![AsyncAPI logo](./assets/github-repobanner-generic.png)](https://www.asyncapi.com)
<!-- toc is generated with GitHub Actions do not remove toc markers -->

<!-- toc -->

- [Overview](#overview)
- [Template Output](#template-output)
- [Technical requirements](#technical-requirements)
- [Async API Requirements](#async-api-requirements)
- [Supported protocols](#supported-protocols)
- [How to use the template](#how-to-use-the-template)
  * [CLI](#cli)
    + [Run the following command to generate a Go module](#run-the-following-command-to-generate-a-go-module)
    + [How to use the generated code](#how-to-use-the-generated-code)
      - [Pre-requisites](#pre-requisites)
      - [Running the code](#running-the-code)
- [Template configuration](#template-configuration)
- [Contribution guide](#contribution-guide)

<!-- tocstop -->

## Overview

This template generates a Go module that uses [watermill](https://github.com/ThreeDotsLabs/watermill) as library for building event-driven applications. The reason to choose [watermill](https://github.com/ThreeDotsLabs/watermill) is because it provides an abstraction to messaging supporting multiple protocols. This will help this generator support multiple protocols.

## Template Output

The `go` code generated by this template has the following structure

- __asyncapi__
  - [handlers.go](asyncapi/handlers.go) -- handlers for publishers and subscribers
  - [payloads.go](asyncapi/payloads.go) -- generated `go` structs using Modelina
  - [publishers.go](asyncapi/publishers.go)
  - [router.go](asyncapi/router.go) -- configures [watermill](https://watermill.io/) router, needed only for subscribers
  - [server.go](asyncapi/server.go) -- server config
  - [subscribers.go](asyncapi/subscribers.go)
- [go.mod](go.mod)
- [go.sum](go.sum)
- [main.go](main.go)

## Technical requirements

- 1.1.0 =< [Generator](https://github.com/asyncapi/generator/) < 2.0.0,
- Generator specific [requirements](https://github.com/asyncapi/generator/#requirements)

## Async API Requirements

To have correctly generated code, your AsyncAPI file MUST define operationId for every operation.

Example:
```
channels:
  light/measured:
    subscribe:
      operationId: LumenPublish
```

## Supported protocols

|  Protocol |  Subscriber | Publishers   |
|---|---|---|
|  AMQP | Yes   | Yes   |


## How to use the template

This template must be used with the AsyncAPI Generator. You can find all available options [here](https://github.com/asyncapi/generator/).

### CLI

This template has been tested to generate an AMQP subscriber for [this asyncapi.yaml file](./test/asyncapi.yaml)

#### Run the following command to generate a Go module

```bash
npm install -g @asyncapi/generator
# clone this repository and navigate to this repository
ag test/asyncapi.yaml @asyncapi/go-watermill-template -o /path/to/generated-code -p moduleName=your-go-module-name
```

Following are the options that can be passed to the generator

1. moduleName: name of the go module to be generated

#### How to use the generated code

The above code currently generates a Go module that has a AMQP subscriber.

##### Pre-requisites
To run the generated code the following needs to be installed

1. go 1.16 +
2. rabbitmq-server OR docker

##### Running the code

1. Navigate to the path where the code was generated
2. Run the following commands to download the dependencies
```bash
go mod download
go mod tidy
```
3. Currently the code does not utilize the server bindings to generate the server URI. It is currently hardcoded to point to a local instance of `rabbitmq`. It is hardcoded as `"amqp://guest:guest@localhost:5672/"` at `<generated-code>/config/server.go`. Change it as per your rabbitmq instance requirements
4. Finally to execute the code run
```bash
go run main.go
```
5. Running local instance of `rabbitmq`, navigate to it using `http://localhost:15672/` with username and password `guest`/ `guest` (These are default rabbitmq credentials).
   FYI one can start an instance of `rabbitmq` using  `docker` as follow
   ```
   docker run -d -p 15672:15672 -p 5672:5672 rabbitmq:3-management
   ```
6. Create a queue as per the AsyncAPI spec.
   This can be done either of the following ways
   -  Using the UI: Refer to this [article](https://www.cloudamqp.com/blog/part3-rabbitmq-for-beginners_the-management-interface.html) that walks through the process of how this can be done in the UI / RabbitMQ Admin
   - `cURL` request. Default rabbitmq user is `guest` and password is `guest`
   ```
    curl --user <rabbit-user>:<rabbit-password> -X PUT \
      http://localhost:15672/api/queues/%2f/<queue-name> \
      -H 'cache-control: no-cache' \
      -H 'content-type: application/json' \
      -d '{
    "auto_delete":false,
    "durable":true
    }'
   ```
7. Publish a message to the queue as per the AsyncAPI spec. This can be done either of the following ways
   - Using the UI: Refer to this [article](https://www.cloudamqp.com/blog/part3-rabbitmq-for-beginners_the-management-interface.html) that walks through the process of how this can be done in the UI / RabbitMQ Admin
   - `cURL` request. Default rabbitmq user is `guest` and password is `guest`
   ```
    curl --user <rabbit-user>:<rabbit-password> -X POST \
      http://localhost:15672/api/exchanges/%2f/amq.default/publish \
      -H 'cache-control: no-cache' \
      -H 'content-type: application/json' \
      -d ' {
    "properties":{},
    "routing_key":"light/measured",
    "payload":"{\"id\":1,\"lumens\":2,\"sentAt\":\"2021-09-21\"}",
    "payload_encoding":"string"
    }'
   ```
8. Check the output at the terminal where `go run main.go` was running and the published message should be printed

## Template configuration

You can configure this template by passing different parameters in the Generator CLI: `-p PARAM1_NAME=PARAM1_VALUE -p PARAM2_NAME=PARAM2_VALUE`

|Name|Description|Required|Example|
|---|---|---|---|
|moduleName|Name for the generated Go module|false|`my-app`|

## Contribution guide

If you are interested in contributing to this repo refer to the [contributing](https://github.com/asyncapi/go-watermill-template/docs/contributing.md) docs
