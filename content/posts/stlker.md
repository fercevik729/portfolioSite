---
title: "STLKER"
date: 2022-10-08T12:57:06-07:00
draft: false
url: "/projects/stlker"
cover:
    image: /img/stlker-cli.jpeg
    alt: 'STLKER'
    caption: 'STLKER CLI output'
tags: ["golang", "docker", "grpc", "rest", "aws", "swagger", "finance"]
---
# STLKER
STLKER is a stock tracking application that utilizes RESTful and gRPC microservices to serve content neatly in the command line via `curl` and `grpcurl`
## About
This project is mostly made in Golang using the Gorilla Mux and GORM frameworks. It also implements several other features that are common in many webservices today. Moreover, this project was completed in order to develop a more complete understanding of microservice architecture, as well as to create something that can be used directly from the command line without much parsing.

## Structure
The application consists of 4 separate services that were eventually all linked as Docker containers with `docker compose`.

### gRPC
The gRPC service was developed using protocol buffers and Golang. It supports bidirectional streaming as well as typical unary services. However, bidirectional streaming is difficult if not impossible to implement with current frameworks for REST. So in order to access the streaming service the user must utilize `grpcurl`:
```
  grpcurl --plaintext -msg-template -d @ localhost:9090 Watcher/SubscribeTicker
```
This will open a "streaming session" from which users will send requests of the following format and receive updated financial information.
```
  {"Ticker": "MSFT", "Destination": "EUR"}
```
Users can also directly request the unary service:
```  
grpcurl --plaintext -msg-template -d '{"Ticker": "MSFT", "Destination":"USD"}' localhost:9090 Watcher/GetInfo
```
and
```
  grpcurl --plaintext -msg-template -d '{"Ticker": "MSFT", "Destination":"USD"}' localhost:9090 Watcher/MoreInfo
```
This microservice forwards these requests in an appropriate format to a 3rd party API--Alpha Vantage--from which it receives the information, processes it, and sends it to the caller.

### Control
The control service is the RESTful API service that most requests will be sent to. It will then send some of these to the gRPC service and return the desired response.

This microservice is also responsible for caching some of the requests in a Redis cache to improve latency and minimize the number of requests to AV's API. Additionally, it allows users to create accounts, log themselves in via JWT authentication, create, update, delete and view portfolios of their own. This additional data is persisted in a PostgreSQL database, where any sensitive data is encrypted using the standard crypto package. 

In addition, this application implements various middlewares and serves documentation at the /docs endpoint for more information about request and response format

### Docker
In order to dockerize the application I had to create a docker-compose.yml file with containers for the gRPC, control, Redis, and PostgreSQL services. For the gRPC and control services I had to build and publish custom images so that I could use them later when I deployed my app to AWS's ECS and CloudFormation.

## Disclaimer
The financial data that this application provides is often late and should not be used to make real world decisions on. For more information please view the github link of this project [here](https://github.com/fercevik729/STLKER).


