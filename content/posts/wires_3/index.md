---
title: "Wires #3: Local Setup and Development Tools"
date: 2024-01-01T18:01:37+02:00
draft: true
toc:
  enable: true
  auto: false
share:
  enable: true
authors: []
description: ""

tags: []
categories: []
series: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: "featured-image."
featuredImagePreview: "featured-image."
---

In this blog, we will install all the tools, development packages and to test you'll create a simple server in C#.

<!--more-->

## Getting Started
Since i am developing this project on my archlinux btw, with very minimal setup tools, I won't use any bloated IDEs. Feel free to use whatever you want!
Before getting started, make sure to install:
- `git`
- `go`
- `dotnet`
- `docker`
- `docker-compose`

## Directory Structure
I will structure my project as modules, each module has other sub modules.
```
.
├── backend/
├── engine/
├── frontend/
└── README.md
```

## Backend Setup
I am writing the backend in go and C#, for me here are my installed vesions.
- C# for the web API: `.NET 8.0.100`.
- Go for the simulation engine: `1.21.5 linux/amd64`
- Docker and docker-compose : `24.0.7`, `2.23.3`

Let's create the initial backend, we'll use : `dotent new webapi` to create a barebone API.

As it's shown below, the template API created by .NET has one simple GET route called : `/weatherforecast` which returns random values.
{{< image src="backend.png" position="center" style="border-radius: 8px;" caption="The init API" >}}

.NET creates launchSettings:
```json
    "http": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "http://localhost:5086",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
```

Let's run (`dotnet run`) and call the API:
{{< image src="api_call.png" position="center" style="border-radius: 8px;" caption="Calling the API for the first time!" >}}

## Engine Setup
Let's create a simple go project, contains just single `main.go`. Make sure to run : `go mod init github.com/AYehia0/wires/engine`

This creates a file called `go.mod` (will contain all the dependencies)

```go
package main

import "fmt"

func main() {
	fmt.Println("Engine has been started!")
}
```
Run the engine using : `go run .`

## Frontend
As I am a backend guy + I hate frontend frameworks, I will force myself to use the easiest thing ever (not vanila JS). Idk now lol

## Resources
- [Create a Web API with ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-web-api?view=aspnetcore-8.0)
