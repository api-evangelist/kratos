---
title: "Go Engineering - Dependency Injection"
url: "https://go-kratos.dev/blog/kratos/go-project-wire/"
date: "Wed, 14 Jul 2021 00:00:00 GMT"
author: ""
feed_url: "https://go-kratos.dev/blog/rss"
---
In the default project template kratos-layout of the microservices framework kratos v2 , we use google/wire for dependency injection and recommend that developers use this tool when maintaining projects. At first glance, wire seems counter-intuitive, leading many developers to not understand why to use it or how to use it (including myself in the past). This article aims to help everyone understand the usage of wire.
