---
layout: page
title: Group-Based Social Network Web App
description: Single-page application for group-based social networking similar to Reddit, built with React and JavaScript
img: /assets/img/projects/gingerapp/thumbnail.png
importance: 3
category: Web Development
related_publications: false
---

[Demo](https://ginger.fly.dev/) \| [Source](https://github.com/plasmas/Ginger)

_Per course policy, access to code is only granted upon request._

---

## Highlights

- Created Ginger, a group-based social network application using the MERN web development technology stack.
- Built backend using Express and MongoDB, exposing REST APIs for object creation, modification, and deletion.
- Deployed WebSockets for push notifications and infinite scroll, enabling live post feeds on client side.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/thumbnail.png" title="group main page" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Group main page
</div>

---

## Features

### User Operations

- **Sign Up**. Users can sign up for an account by providing a username, email, and password.
- **Log In**. Users can log in to their account using their username and password.
- **Log Out**. Users can log out of their account.
- **Edit Profile**. Users can edit their profile information, including their username, email, and password.
- **Deactivate Account**. Users can deactivate their account, which will hide their profile and posts from other users.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/signup.png" title="Signup page" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/profile.png" title="Profile page" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/update_profile.png" title="Profile update page" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Signup page; Middle: Profile page; Right: Profile update page.
</div>

### Group Operations

- **Create Group**. Users can create a new group by providing a name, icon, labels. Groups can either be public or private.
- **Invite Users**. Group owners can invite other users to join their group.
- **Manage Group**. Group admins can add new admins and remove members from the group.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/group_creation.png" title="Group creation page" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/invite_user.png" title="Invite users to group" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/manage_admins.png" title="Manage Group admins" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Group creation page; Middle: Invite users to group; Right: Manage Group admins.
</div>

### Post Operations

- **Create Post**. Users can create a new post in a group by providing a title, content, and optional media files, including images, videos, and audio.
- **Delete Post**. Users can delete their own posts.
- **Flag or Hide Post**. Users can flag or hide posts that violate community guidelines. Admins of the group will be notified and can take action.
- **Comment on Post**. Users can comment on posts to share their thoughts and opinions.
- **Mentions**. Users can mention other users in their posts and comments using the `@` symbol. The user mentioned will receive a notification.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/post_creation.png" title="Post creation page" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/post.png" title="Post presentation page with comments" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Post creation page; Right: Post presentation page with comments.
</div>

### Other Features

- **Infinite Scroll**. Posts are loaded dynamically as the user scrolls down the page.
- **Push Notifications**. Users receive real-time notifications for new posts, comments, and mentions. This is made possible using **WebSockets**.
- **Suggestions**. Users will receive suggestions for groups to join based on the labels they are already in.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/flag_noti.png" title="Notification of flags" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="/assets/img/projects/gingerapp/mention_noti.png" title="Notification of mentions" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Left: Notification of flags; Right: Notification of mentions.
</div>

---

## Technologies

This project uses the MERN stack: MongoDB, Express, React, and Node.js.

- Frontend. The frontend is built with React using JavaScript. Material-UI is used for UI elements.
- Backend. The backend is built with Node.js and Express. MongoDB is used as the database. Multimedia files can be stored in either **GridFS** or **AWS S3**.
