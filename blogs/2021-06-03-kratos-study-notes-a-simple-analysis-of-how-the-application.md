---
title: "Kratos Study Notes - A Simple Analysis of How the Application Runs via Layout"
url: "https://go-kratos.dev/blog/kratos/go-layout-operation-process/"
date: "Thu, 03 Jun 2021 00:00:00 GMT"
author: ""
feed_url: "https://go-kratos.dev/blog/rss"
---
0X01 Exploring Kratos Operation Principles via Layout (kratos v2.0.0-beta4) Create Project First, install the necessary dependencies and tools: go protoc protoc-gen-go # Create project template kratos new helloworld cd helloworld # Pull project dependencies go mod download # Generate proto template kratos proto add api/helloworld/v1/helloworld.proto # Generate proto source code kratos proto client api/helloworld/v1/helloworld.proto # Generate server template kratos proto server api/helloworld/v1/helloworld.proto -t internal/service After executing the commands, a service project will be…
