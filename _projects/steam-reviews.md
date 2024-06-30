---
layout: page
title: Steam Reviews Analyzer
description: Web app for browsing and analyzing Steam games and reviews. Leveraging Svelte, Tailwind CSS, Express, MySQL, MongoDB, and Redis.
img: /assets/img/projects/steam-reviews/thumbnail.png
importance: 2
category: Web Development
related_publications: false
---

[Demo](http://us.yqwong.com/) \| [Source](https://github.com/TongHuoAo/CIS5500_Project_Group5)

_Per course policy, access to code is only granted upon request._

---

## Highlights

- Developed a smooth frontend using **Svelte** with **TypeScript**, featuring advanced review search with analytics via **D3**.
- Built **REST APIs** with **Express**, **MySQL**, and **MongoDB** for efficient storage and retrieval of over 100M reviews.
- Optimized SQL queries by **14x** using composite indexes and utilized **Redis** for fast repeated request handling.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include video.liquid path="https://www.youtube.com/embed/c3cQtNUhF3A?si=PYQ_XbOFGEoLV9U9" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/steam-reviews/intro.png" title="homepage image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    App Homepage
</div>

---

## Goal

Steam is the largest PC gaming platform, providing browsing, purchasing, downloading, and reviewing functionality. It also contains a wealth of information about PC games as well as player feedback and reviews. However, the platform currently does not allow users to search for and learn about games in greater depth based on user reviews and game companies.

As a result, we created a platform called the Steam Reviews to provide users with a more diverse platform for searching games other than just the standard game metadata. This enables users to better understand and expand their gaming platforms and perspectives (games they are or may be interested in) by reading other users' reviews and reactions to games and games released by a specific company.

---

## Features

With our application, users can:

- Search games with complex filters
- Search reviews based on a plethora of criteria
- View game details and review statistics of each game, across languages and time
- Interactively learn about each company, including their game development focus (genres), and localization efforts
- View top games list curated by us; and watch reviews for each game in random.

---

## Technologies & Architecture

Our application is composed of frontend and backend parts, which uses different technologies.

The frontend is responsible for user interaction, and efficient data fetching from the backend. Technologies of the frontend include:

- **TypeScript**. Due to strict type checking, we deem TypeScript superior to JavaScript as it significantly improves the development experience. We declared entity types that are shared between the frontend and the backend, according to API specification, which is helpful when integrating the frontend with the backend. Note that we also used TypeScript as the singular programming language in the backend.
- **Svelte & Svelte Kit**. Instead of React, we used Svelte as the framework for frontend development. Compared to vanilla React, Svelte provides more straightforward reactive actions, native filesystem-based routing, and server side rendering (SSR), which expands the frontier of web development and makes web experience even better.
- **Tailwind CSS**. CSS styling library that makes CSS a lot better to use.
- **Flowbite UI**. A UI library that provides Svelte compatible components.
- **Axios**. An excellent HTTP client. We choose Axios over Fetch due to its simplicity.
- **D3**. A JavaScript library for our statistical visualizations.

Technologies of the backend include:

- **Express**. Standard library for serving HTTP APIs.
- **MySQL**. SQL database for storing the majority of our data.
- **MongoDB**. NoSQL solution to store steam review contents due to their great variance in length.
- **Redis**. High-performance, in-memory key-value database for caching query results.

---

## Data

- [Steam Games Dataset](https://www.kaggle.com/datasets/fronkongames/steam-games-dataset)
  - Description:
    This dataset contains information about over 85,000 Steam games, including game names, release dates, expected owners, and prices.
  - Usage of Data:
    Break Games Dataset into four tables: games, companies, categories, genres.
- [Steam Reviews Dataset](https://www.kaggle.com/datasets/kieranpoc/steam-reviews)
  - Description:
    This dataset contains information and content about over 140m reviews of the games on Steam.
  - Usage of Data:
    Break Reviews Dataset into two tables: reviews, authors. Extract the review content for NoSQL solution.

---

## Database Design

MySQL is used to store the majority of our data, and the ER diagram is as follows:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/steam-reviews/er.png" title="homepage image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    ER Diagram
</div>

The corresponding database schema is as follows:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/steam-reviews/database.png" title="homepage image" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Database Table Schema
</div>

---

## Performance Evaluation

Following is the table of time for queries before and after optimization.

|                    | Original | +Query optimization | +Indexing | +Caching |
| ------------------ | -------- | ------------------- | --------- | -------- |
| review search      | ~ 229s   | ~ 221s              | ~ 16s     | < 1s     |
| best 10 companies  | ~ 15s    | /                   | /         | < 1s     |
| game search        | ~ 6s     | /                   | ~ 4s      | < 1s     |
| company statistics | ~ 2s     | /                   | /         | < 1s     |
| review timeline    | ~110s    | /                   | ~ 1s      | < 1s     |

---

## Optimizations

1. No Optimization/Original. After observation, most time-consuming queries have one of the following properties:
   - Selection not pushed in. This results in excessive tuples during JOIN operations. In “Review Search”, this is one of the issues with the original query.
   - Aggregation after JOIN. If the result of the query depends on the correct JOIN of large tables, pushing in selection will not be viable. In “Best 10 Companies”, this is the case and normal query optimization cannot improve the performance.
   - Inefficient indexes. If there are limited indexes, or indexes don't fit the query well, queries will still be slow. In query “Review Search”, “Game Statistics”, “Review Timeline”, introducing new indexes drastically improves the performance.
2. Query Optimization. Due to the inherent nature of each query, query optimization is only applicable to certain queries. In our app, we applied selection push-ins and we observed moderate performance improvements. This is because, by pushing the game name prefix selection into the Game table, it reduces the size of the resulting table that is on the left side of the subsequent join. This reduces the number of final tuples and hence removes the need for redundant selection after.
3. Indexes. If the query relies on range or equivalence selections, indexes are particularly powerful.
   - For “Review Search”, in the Game table, index on (game\*name) makes prefix search very fast. And in the Review table, the composite index (app_id, thumb_up) helps to find all reviews of a game in descending order of thumb ups, which hugely narrows the final range to filter.
   - For “Game Search”, in the Game table, index on (game_name) also speeds up prefix search. However, since the hotspot is calculating the positive rate of each game and ordering according to the positive rate, the index doesn’t provide drastic performance improvements.
   - For “Review Timeline”, in the Review table, index (app_id, time_created\* recommended) is exactly tailored for this query: It allows fast computation of positive/negative reviews in each time interval, for a given game.Therefore, the computation efficiency is greatly improved.
4. Caching. If the query is very computation-heavy but has limited input space, caching is very helpful.
5. In all 5 queries listed above and more, we introduced caching to reduce MySQL workload by caching the result in a Redis key-value database. The route and params form the key, while the JSON response is the value. If an incoming query matches exactly the route and params, then the JSON in Redis is returned directly without consulting MySQL. This caching strategy is reasonable because the MySQL database is mostly static and cached data has a long period of validity.
   - For “Best 10 Companies”, this strategy is particularly useful because the input space of this query is very small (only 1 integer parameter). All results can be covered in little time, which is quite beneficial for the frontend experience.
   - For “Review Search”, this method works but is not perfect, since there are many more parameters in the search functionality, and it is impossible to exhaust all of them. However, considering the time cost, it is still effective against potential popular searches.

---

## Acknowledgements

### Team Members

- Yuanqi Wang ([GitHub](https://github.com/plasmas))
- Tong Hu ([GitHub](https://github.com/TongHuoAo))
- Zhiyan Zeng ([GitHub](https://github.com/hailiezzy))
- Siyuan Fu ([GitHub](https://github.com/IwakuraRein))

### Mentors

- Akanksha Ashok
