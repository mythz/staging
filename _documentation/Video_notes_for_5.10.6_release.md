---
slug: video-notes-for-5-10-6-release
title: Video notes for 5.10.6
---

## GitHub Actions Templates with walk through videos

We've created 4 GitHub Action `mix` templates to help you setup your CI environment on GitHub Actions quickly for new and existing ServiceStack templated projects.
They are designed to get you started with a CI environment earlier by making it quicker to setup with cost effective hosting. 
Rather than a focus on the ability to scale with cloud vendor offerings, they provide a pattern to dockerize your prototype applications from the start and host them on a single cost effective server you can reuse.

These GitHub Action release templates use the naming convention of `release-{docker image repository}-{host type}` so that we can create templates with multiple provider options.

### GitHub Container Registry deployed to a Linux server via SSH

First up is our `release-ghr-vanilla` template which uses GitHub's own Container Repository (ghcr.io) and a standalone "vanilla" Linux server that can host multiple prototype or low traffic applications for a cost effective option. 
GitHub Actions deploys to this stand alone Linux server via SSH and an `nginx-proxy` container along with a LetsEncrypt companion container that takes care of the TLS certificates automatically.

<iframe width="896" height="525" src="https://www.youtube.com/embed/0PvzcnxlBvc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

> [Full written tutorial and example available here](https://docs.servicestack.net/do-github-action-mix-deployment).

There are two other variations on this template where alternate Docker container repositories can be used.

- `release-ecr-vanilla` - AWS Elastic Container Repository
- `release-hub-vanilla` - Docker Hub Container Repository

### AWS ECS without an Application Load Balancer (ALB)
Another template we have created is the `release-ecr-aws` template that uses AWS ECS (EC2 Container Service) to handle deployments but uses `nginx-proxy` container with Lets Encrypt conpanion to handle the routing.

This is a cheap way to get multiple prototype applications onto ECS while avoiding ALB costs for low traffic applications that only need part of a EC2 instance to run.

The GitHub Actions can then be easily modified to support the more traditional ECS setup with an ALB if/when the scaling needs arise.

<iframe width="896" height="525" src="https://www.youtube.com/embed/Eh4tvLN8i8g" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

> [Full written tutorial and example available here](https://docs.servicestack.net/mix-github-actions-aws-ecs).

## Servicify and Deploy SQLite's Chinook Sample Database with AutoQuery

A new tutorial with a full walk through of taking an existing database and using AutoQuery and related `x` tooling to "Servicifiy" a database quickly while still retaining flexibility and features of standard ServiceStack services.

The Chinook SQLite sample database represents a digital store, and by starting with a new ServiceStack `web` project, we can instantly make this data fully queryable as well as manageable via the `autocrudgen` mix template.

<iframe width="896" height="525" src="https://www.youtube.com/embed/NaJ7TW-Q_pU" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

> [Full written tutorial and example available here](https://github.com/NetCoreApps/Chinook).


## ServiceStack Studio Features Highlight Video

On the theme of AutoQuery, we've made a feature highlight video focusing on how AutoQuery services can help developers and non-developers alike to manage service access with our ServiceStack Studio application. 

ServiceStack Studio can be used for prototyping or even managing services, data, users, permissions and validation. There are quite a lot of features, so we've tried to make the capabilities of AutoQuery + ServiceStack Studio clearer with this demonstration video.

<iframe width="896" height="525" src="https://www.youtube.com/embed/kN7371bqUII" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

> [ServiceStack Studio documentation can be found here](https://docs.servicestack.net/studio)

## Instant Client Apps

<iframe width="896" height="525" src="https://www.youtube.com/embed/5crcDfl467Q" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
