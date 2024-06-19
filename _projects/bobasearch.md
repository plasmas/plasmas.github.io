---
layout: page
title: BobaSearch
description: Distributed analytics platform and search engine written in Java, featuring MapReduce, PageRank, and TF-IDF algorithms.
img: /assets/img/projects/bobasearch/thumbnail.png
importance: 1
category: Java
related_publications: false
---

[Source](https://github.com/plasmas/BobaSearch)

_Per course policy, access to code is only granted upon request._

---

## Highlights

- **Distributed Analytics Platform**: Implements a robust and scalable search engine and analytics platform using Java, featuring a distributed key-value store and computing framework.
- **Advanced Search Algorithms**: Utilizes MapReduce, PageRank, and TF-IDF algorithms for efficient indexing, ranking, and retrieval of search results.
- **High Performance and Fault Tolerance**: Designed to crawl and index one million web pages, leveraging sharding and a master-worker architecture to ensure high performance and fault tolerance.
- **Real-time Query Processing**: Provides real-time search results with efficient retrieval, score computation, and sorting algorithms for a responsive user experience.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/bobasearch/fast_food.png" title="search result for fast food" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/bobasearch/youth.png" title="search result for youth" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/bobasearch/music.png" title="search result for music" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Search result list for "fast food" (left), "youth" (middle), and "music" (right).
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/bobasearch/fast_food_detail.png" title="search result for fast food" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/bobasearch/youth_detail.png" title="search result for youth" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Top result page for "fast food" (left), "youth" (right).
</div>

---

## Overview

BobaSearch is a distributed analytics platform and search engine written in Java. It features a web crawler, a distributed indexing system, and a search engine that uses the PageRank algorithm to rank search results. The system is designed to crawl and index one million web pages, and provide real-time search results based on user queries.

The analytics platform is composed of the following components:

- **Distributed Key-Value Store (KVS)**: The KVS is used to store the crawled web pages, the index, and the PageRank values. It uses sharding to distribute the data across multiple nodes and ensure fault tolerance. For coordination, the KVS uses a master-worker architecture with a single master node and multiple worker nodes. The master node is responsible for managing the workers and distributing the data, while the worker nodes store the data and perform operations on it.
- **Flame**: Flame is a distributed computing framework that is used to run MapReduce jobs on the KVS. It provides a simple API for submitting jobs and managing their execution. The analytics platform uses Flame to run the web crawler, the indexer, and the PageRank algorithm.

---

## Challenges

- **Low throughput for crawler**. When trying to deploy our crawler, we found that the throughput of our crawler is very slow. To crawl one page, we sometimes need 3-5 seconds. Even worse is that our crawler worker would be stuck at certain pages for a prolonged period of time. We were very confused by this behavior, and had many hypotheses why it happened. For example, we first thought there could be some kind of synchronization problem, where two workers tried to access the same table and caused a deadlock. After some debugging, we finally found that the problem actually lied in the part where we requested robots.txt. If the website did not have a robots.txt, the HttpUrlConnection class would wait for a long time before it dropped the connection. An easy solution to this problem is to set a timeout for requests. With this fix, we can crawl more than 20 pages per second.
- **Crawler Design and Scalability**. Creating a crawler to fetch one million pages is a significant task. The crawler must be capable of efficiently navigating through web pages, following links, and managing possible issues such as loops or dead ends. Furthermore, the crawler must respect the robots.txt file and deal with different website structures, making this a challenging aspect of the project. The crawler also needs to be scalable and efficient, as crawling can be a time-consuming and resource-intensive process. The use of concurrent or distributed crawling may be required to reach the target of one million pages in a reasonable amount of time.
- **Indexing and Ranking Algorithm**. The second difficult aspect of the project is implementing the indexing and PageRank algorithm using Flame. The index must be designed in a way that allows for fast retrieval of relevant URLs based on the search query. The PageRank algorithm also involves complex computations to determine the importance of each page based on its incoming links. Both of these tasks require a deep understanding of information retrieval and search engine algorithms.
- **Real-time Query Processing**. Even though the indexing and PageRank calculation is done in a batch process using Flame, the actual retrieval of relevant URLs, score combination, and sorting must be done in real-time to provide a responsive user experience. The system must be optimized to read the search terms, look up the corresponding entries in the index, fetch the relevant PageRank values, compute the TF/IDF scores, combine the scores, sort the URLs, and assemble the results page as quickly as possible. Designing and implementing an efficient algorithm that can handle all these tasks in real-time is a significant challenge.

---

## Future Improvements

- **Distributed front-end service**. We could have added a load balancer and a few front-end servers. In this way, we could make the front end a distributed system in itself, and thus increase overall efficiency.
- **Indexing and Ranking Algorithm**. We could have tried different ways of calculating the TF score as well as different algorithms to calculate the overall score of each page to find the one with a better performance.

---

## How to Run

1. Use IntelliJ to compile the project into a JAR file called MiniGoogle.jar
2. Launch the masters and workers with the following run configurations and params:

```sh
# KVS Master
cis5550.kvs.Master 8000
# KVS Worker
cis5550.kvs.Worker 8001 worker1 localhost:8000
# Flame Master
cis5550.flame.Master 9000 localhost:8000
# Flame Worker
cis5550.flame.Worker 9001 localhost:9000
# Submit Crawler
cis5550.flame.FlameSubmit localhost:9000 crawler.jar cis5550.jobs.Crawler http://simple.crawltest.cis5550.net/
# Submit Indexer
cis5550.flame.FlameSubmit localhost:9000 indexer.jar cis5550.jobs.Indexer
# Submit PageRank
cis5550.flame.FlameSubmit localhost:9000 pagerank.jar cis5550.jobs.PageRank 0.01
# frontend server
cis5550.search.frontend.Server localhost:8000
```

3. Flame Jars can be compiled using the following commands:

```sh
# Pack Crawler
javac src/cis5550/jobs/Crawler.java -cp lib/webserver.jar:lib/kvs.jar:lib/flame.jar --source-path src && cd src && jar cvf crawler.jar . && mv crawler.jar ../
# Pack Indexer
javac src/cis5550/jobs/Indexer.java -cp lib/webserver.jar:lib/kvs.jar:lib/flame.jar --source-path src && cd src && jar cvf indexer.jar . && mv indexer.jar ../
# Pack PageRank
javac src/cis5550/jobs/PageRank.java -cp lib/webserver.jar:lib/kvs.jar:lib/flame.jar --source-path src && cd src && jar cvf pagerank.jar . && mv pagerank.jar ../
```
