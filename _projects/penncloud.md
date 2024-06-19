---
layout: page
title: PennCloud
description: Cloud-based platform with mail, storage services. Built with C++ using pure POSIX standards.
img: /assets/img/projects/penncloud/architecture.png
importance: 1
category: C++
related_publications: false
---

[Source](https://github.com/plasmas/PennCloud)

_Per course policy, access to code is only granted upon request._

---

## Highlights

- **HTTP Server**: Built from scratch using POSIX standards, supporting GET, POST, PUT, DELETE, and OPTION requests. Usage similar to Express.js.
- **Mail Server**: Supports sending and receiving emails using SMTP and POP3 protocols.
- **Storage Service**: Provides functionality for file system operations including uploading, downloading, creating, deleting, renaming, and moving files and folders.
- **Dynamic Load Balancing**: Dispatcher supports dynamic joining and leaving of front-end servers, with versatile load-balancing algorithms (random selection and least load first).
- **Multithreaded Frontend Server**: Handles multiple client connections simultaneously with "Keep-Alive" and chunked encoding support.
- **KV Store**: Distributed key-value store with a master node and clusters of replicas, supporting GET, PUT, CPUT, and DELETE operations. Provides fault tolerance and sequential consistency.
- **Cookie and Session Management**: Secure authentication and session management using cookies, enabling user identity verification and access control.
- **Scalable Architecture**: Easily scalable by adding more clusters to serve additional tablets, with efficient load balancing and fault tolerance mechanisms.
- **Unified Controller Interface**: Simplified controller registration and management, ensuring consistent request handling and response composition across different services.

<div align="center">
<img
  src='/assets/img/projects/penncloud/login.png'
  style="width: 60%; height: auto;"
/>
<div class="caption">
    PennCloud Login Page
</div>
</div>

---

## Overview

PennCloud is a cloud-based platform that supports **mail service**, **storage service** and **admin functionality** through a single-page web application.

- Through the mail service, users can send emails to accounts both inside and outside the domain.
- The storage service allows users to create, delete, rename and move folders or files.
- Administrators can further monitor the status of frontend and backend servers, and even kill or restart some of the nodes. Administrators can also view raw bytes stored in tablet servers.

---

## User Action Control Flow

A user of our services first connects to the frontend dispatcher. The dispatcher is in charge of load balancing. It redirects client requests to idle frontend servers and lets the redirected servers handle the request. The client only knows the IP and port of the frontend dispatcher, but not any of the frontend servers.

A frontend server accepts a user connection dispatched from the dispatcher and processes HTTP requests from that user. It provides a webmail service and a storage service to users. After parsing user requests, the frontend server assigns each request to handlers for the specific service.

The services provided by the frontend server share instances of the KV store client. The client is responsible for connecting to the backend servers and performing appropriate operations on the data stored in the backend. The services then compose HTTP responses and send them back to the user.

The KV store servers consist of a master node and distributed clusters. Each cluster has three replicas, one of which is the primary and the others are secondary. The master is responsible for load balancing and also forwards requests from the admin console. Read requests are served by each replica independently while update requests are serialized through the primary node.

The KV store provides the table abstraction where values are identified by row and column. The KV store supports four operations: PUT(r,c), GET(r,c), CPUT(r,c,v1,v2), DELETE(r,c). Data are partitioned and replicated into different nodes based on their row identifier. The nodes in a cluster provide replication and fault tolerance using checkpoint and log files.

---

## Architecture

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/penncloud/architecture.png" title="homepage image" class="img-fluid rounded" zoomable=true %}
    </div>
</div>
<div class="caption">
    App Architecture
</div>

---

## Component Functionality

### Dispatcher

The dispatcher is responsible for knowing the current running frontend servers and redirects a user to one of the frontend servers by replying with HTTP 301. In our solution, the dispatcher has the following features:

1. **Dynamic joining and leaving**. Front-end servers can join or leave the pool of available servers at any time. Front-end servers wishing to join will continuously send heartbeat messages to the dispatcher, containing its public-accessible hostname and the number of connections it’s currently handling. The dispatcher will add the server to its pool of active servers and will select one of them when a user visits the dispatcher. If the dispatcher doesn’t hear from some frontend server for some period of time, it will remove it from the pool of active servers.

2. **Versatile load-balancing algorithm**. We implemented two algorithms for load-balancing:
   - **Random selection**. The dispatcher will select a random frontend server to delegate the HTTP request.
   - **Least load first**. The frontend server with the least load currently will be selected.

### Frontend Server

The frontend server accepts HTTP requests from the user browser and sends back HTTP responses. Our frontend server has the following characteristics:

1. **Multithreaded**. The server dispatches one thread per connected client (socket). The thread will be responsible for all HTTP requests coming from a single connection. All requests from the client will be processed and answered in serial. This design is inferior compared to event-driven design but is much easier to implement and the performance is satisfying when the number of connections stays low.
2. **“Keep-Alive” enabled**. The server preserves the TCP connection if the most recent HTTP request contains a “Keep-Alive” header. Else the socket/TCP connection will be closed and the thread responsible for the connection will be terminated.
3. **Chunked encoding**. The server accepts chunked encoding transfers from the browser to the client (not vice versa). When indicated by the encoding header, the server waits for the whole body to arrive before sending the request to the router.
4. **Request routing**. The server has a static router that dispatches incoming requests to corresponding controllers. The router checks the target URI of the request and finds the most appropriate endpoint for it.
5. **Load reporting**. The server knows how many connections it is currently handling and will report the load to the dispatcher every few seconds.

### Controllers

The controller layer is responsible for digesting HTTP requests sent by the router and composing appropriate HTTP responses. Each controller is specialized to handle requests for a single URI endpoint. The registration of controllers resembles Express.JS. The controllers have the following characteristics:

1. Unified interface. All controller classes inherit the Controller class, which has a pure virtual interface. This allows custom creation of different controllers but they can all be registered in the router.
2. Cookie enabled. Each time a user successfully logs in, a record will be created in the in-memory cookie service. The HTTP server will search for the “cookie” header in the request and add a new header indicating the user’s identity. Controllers will be able to check the user’s identity and send HTTP 403 forbidden responses if the identity is invalid.
3. HTTP “OPTIONS”, “HEAD” enabled. Controllers can process “OPTIONS” requests by sending back responses containing headers that indicate supported HTTP methods. The “HEAD” method is implemented using a similar way as the “GET” method, but explicitly excluding the response body.

### Services

The service layer is responsible for implementing the logic of PennCloud functionalities. There are 5 services: storage service, mail service, user service, admin service, and static service.

Static service is responsible for transmitting the static resources to the browser, which is basically React build files.

Storage service provides functionality for basic file system operations, including uploading and downloading files, creating folders, deleting files and folders, renaming files and folders, and moving files and folders. Storage service interacts with the KV store client directly.

Mail service builds on top of storage service, where mails for each user are stored in a file with the same name under the root folder of the row .email. Receiving email from outside PennCloud is done through a standalone SMTP server instance, and sending email to outside users is done through a SMTP client with DNS lookup capacity.

User service also builds on top of storage service. The password of each user is stored in a file with the same name under the root folder of the row .user. To authenticate a user, we simply compare their input password with the corresponding password stored in the system. Admin service interacts directly with KV store client, which has the capability to connect with the master server in the backend and the dispatch in the frontend. This service collects the corresponding data and exhibits them to the administrators.

### KV Store Servers

The KV store provides a table abstraction. It appears to the services as a giant table where values are identified by unique row-column pairs. It supports GET, PUT, CPUT, DELETE operations. Internally, the table is sparse, and the content of each row is stored as column-value tuples.

The KV store is distributed. The master is responsible for looking up the specific node for a row key. It determines the tablet to serve a row key by hashing the row key and modding the hash value by a fixed number of tablets. The KV store is easily scalable by adding more clusters that serve additional tablets.

Each cluster consists of a primary node and two secondary nodes. Thus, each data entry is replicated and stored by three nodes. Each cluster provides sequential consistency through remote writes protocol. The read requests are served by the replicas independently while update requests are serialized by the primary node.

The replicas of each cluster are fault-tolerant. For each tablet served, each replica maintains a checkpoint file and a log file. In the face of fault, the replica loads the most recent checkpoint file and replays the operations log to recover to its state.

### Front End React Application

The front-end web service is based on NodeJS and React. It provides an interface to allow users to interact with storage services, email services and administrate other services, with requirement of authentication.

In the front-end webpage, users are firstly required to login/signup with the system. Then users will be led to the home page and can choose the service they need. When finished interacting with the service, the user can jump back to the home page to choose another service. Users can also change their password when they are already logged into the system. When all processes are finished, users are allowed to logout and all cached data in the front-end will be cleared.

---

## Design Decisions

### Dispatcher

The dispatcher is in essence, a frontend server with only a single controller registered to handle all incoming requests. The controller asks for a frontend server to dispatch the current user and composes a HTTP 301 redirect request. The dispatch service knows the active frontend servers by receiving the heartbeat messages. We implemented both random selection and least-load selection algorithm for the dispatcher.

### Frontend Server

The frontend server is designed in a layered architecture, including the connection receiving components, routing components, controller layer and service layer. The main thread always listens for new connections, and once a new connection is established, it will spawn a child thread and delegate the connection to it. The child thread will handle all requests for this connection and will terminate itself once the connection closes. Since there will be multiple threads accessing the controller and services layer, the controllers and services are designed to be thread-safe. The frontend server does not need to know the logic in controllers or services - it only need to deliver requests to the right controllers and send the composed response back to the browser.

### Controller

Controllers are designed in a unified fashion - only one interface is exposed and used by the frontend server. This allows the user to register different controllers under different URI endpoints when building up the frontend server. As the layer between frontend server and services, controllers use services to access data and compose the right responses. We used the nlohmann/json package to parse incoming request bodies and construct response json bodies.

### Service

We build user and mail services on top of storage service so that we can use our file system for convenient file handling.

To support nested folders, we simply conceptualize each folder as a text file which stores a map from the files in the folder to their corresponding columns. To look up a file, we will need to recursively look it up starting from the root folder.

To support files with the same names but existing under different folders, we use hash strings as column keys. The hash is generated from the full path of the file, which is always unique.

### KV Store Server

Partitioning of the row keys among the tablets is based on C++ standard library hashing of the row key string. The hashing value is then modded by the total number of tablets. Each cluster has three replicas and serves three tablets. The client sends a LOOKUP request to the master, and the master responds with a random node that serves the tablet. Whether a node is a primary node or a secondary node is determined at server startup time by the command line input and the configuration file. The nodes then contact the master for the ids of the tablets that they should serve. The master determines this information by reading a configuration file.

The KV store provides sequential consistency, achieved by primary-based remote writes protocol. Read requests are served independently by each replica. The primary node serves update requests independently and then forwards these requests to the secondary nodes in the order that they have been processed on the primary node. For this order, the primary maintains a strictly increasing operation counter, which is the same as the order by which the operations log is written. The secondary nodes forward all update requests to the primary node.

On each replica, there is a mutex for the tablet in memory and a mutex for each row in the tablet. GET request acquires shared access to the tablet and to the specific row. PUT, CPUT, and DELETE requests acquire shared access to the tablet and exclusive access to the specific row. PUT request can potentially upgrade shared access to the tablet to exclusive access in case that a new row has to be added. Each update request is first force-wrote to the operations log and then processed.

Each tablet is checkpointed after a fixed number of update operations. Moreover, when a replica performs a checkpoint, it also forwards a checkpoint request to the primary node, which then forwards to all other nodes. If a node has this tablet in memory, it similarly performs a checkpoint. In this way, checkpointing is synchronized.

During recovery, the recovered node reestablishes connection with the master and gets the ids of the tablets that it should serve. It then contacts the primary node with checksums of its checkpoint file and log file for each tablet. The primary compares the checksums with its copy and instructs the recovered node to download appropriate files if there have been updates. Since checkpointing is synchronized, most recovery can be done using local copies. Eventually, the recovered node registers again with the master and comes back online.

### Front End React Application

The front-end is a single-page web service based on NodeJS and React, and uses Rest API to communicate with other services. The React application is built to a static HTTP file which can be served with our HTTP Server. To achieve better user experience, two libraries Ant Design and Shards UI are used to provide a better UI. RestAPI is used to communicate with backend services.

---

## Responsibilities

Yuanqi Wang

- HTTP Server
- Dispatcher
- Dispatch / Static / Cookie Service
- Dispatch / Static / User Controller

Xi Qiu

- KV Store Client
- Storage Controller/Service
- Mail Controller/Service
- User Service

Yuzhi Shao

- KV Store Servers
- Admin Controller/Service

Miaoyan Zhang

- React Single Page Application

\*Components not mentioned are implemented in a collaborative fashion.

---

## External Libraries

The following external libraries are used in our project:

- [Ant Design UI Library](https://github.com/ant-design/ant-design). For React single page application UI design.
- [Shards-React](https://github.com/DesignRevision/shards-react). For React single page application UI design.
- [C++ JSON](https://github.com/nlohmann/json). For HTTP body JSON parsing and composing. Only used in the controller layer.
