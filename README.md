# .NET 6 Minimal API, AKS and More...

## Overview

With the release of .NET 6 comes the ability for API developers to create true "minimal" APIs in .NET.  This repo serves as an example that will introduce you to .NET 6 Minimal APIs in a simple, straightforward way.  As you work your way through the content below, you will also be guided through taking this simple API and deploying it to Azure Kubernetes Service (AKS) as well as a few other topics to help you get fully up and running using Minimal APIs.

## Goals

- Run your first .NET 6 Minimal API
- Containerize and deploy a .NET 6 Minimal API to AKS
- Create a new template for `dotnet new` using the code sample in this repo
- Generate a client SDK using OpenAPI Generator

## Prerequisites

- [VSCode](https://code.visualstudio.com/download)
- [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0)
- Azure Subscription (if you would like to try out deploying to AKS)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli#install)
- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
- [Docker](https://docs.docker.com/get-docker/)

## Getting Started

Clone this repository and open the cloned folder in VSCode.  Upon opening the project, go ahead and restore any dependencies as VSCode prompts you to do so.  Before diving in too deep, lets take a tour of the project structure:

[image of project structure here....]

- **.template.config**
  - This will be important when you create a 'dotnet new' template using this code base.  More on that later...
- **deployment**
  - This folder contains the bicep script that we will use later to stand up some Azure infrastructure for deploying this Minimal API to AKS
  - Within this folder you will also find some yaml defining our kubernetes deployment and service.
- **Features and Shared folders**
  - We will talk more about the contents of these folders in just a moment when we cover the topic of organizing functionality within a Minimal API.  While it might be fun to have 1,000 lines of code in your Program.cs representing all of the endpoints of your Minimal API, we'll bring some sanity to the world by breaking out functionality in to more maintainable chunks.
- **Dockerfile**
  - This is a typical Dockerfile to be used for containerizing our API.  Take note of our dependency on .NET 6 here.
- **Program.cs**
  - This is the main entry point for our Minimal API.

## Running the Minimal API

At this point, open Program.cs and take a minute to familiarize yourself with its contents.  A few things of note here:

- There are a few external dependencies we have brought in:
  - [FluentValidation](https://fluentvalidation.net/) is brought in to help with various validation needs throughout the API.  Minimal API does not come packaged with any built-in validation mechanisms, however, FluentValidation is a tried-and-true, community accepted, validation mechanism.
  - [Carter](https://github.com/CarterCommunity/Carter) is being used to help bring some additional organization to the functionality within the API.  For the purposes of this example, you will find some endpoints represented directly in Program.cs as well as some endpoints that exist within Carter Modules.  The endpoints that exist in Program.cs serve the purpose of showing just how easily it is to begin developing functionality using .NET 6 Minimal APIs.  That said, carry this path forward in your mind and you will see that you could, potentially, end up with an unwieldy, lengthy and unmaintainable Program.cs.  Carter aims to solve this issue by making it easy to segment your functionality away from the main entry point in Program.cs.
  - Swagger/OpenAPI is registered as a service and is then used to expose an OpenAPI endpoint for end users to explore our API.
  - Application Insights has been brought in to provide full telemetry logging across the API.
- `app.MapCarter();` is all that is needed to trigger a scan of the current assembly to discover any Carter Modules that exist. 
- The "Hello, World!"-style endpoints you find in Program.cs leave you with an example of some very simple endpoints.

Go ahead and run the API (F5 or switch over to the "Run and Debug" screen in VSCode).  Upon successful build and run of the API, you should be presented with the Swagger/OpenAPI UI where you can test out a few of the endpoints.

## Organizing Functionality

As was mentioned previously, loading up your Program.cs with 1,000 lines worth of API functionality likely isn't the best idea.  To help with project organization, we can use Carter Modules and some simple folder organization within our project.  

While there are a few philosophies for how to best organize functionality within an API, this repo contains an example of a [Vertical Slice Architecture](https://jimmybogard.com/vertical-slice-architecture/) approach.  Put simply, this just means that we are organizing functionality within our API around each use-case that is being served through the API.  You will find each "use-case" represented as a separate feature within the **Features** folder (i.e.: Inventory, Orders)

### Inventory Feature

The Inventory feature is a straightforward Carter Module that contains one endpoint.  Notice that `InventoryModule` implements `ICarterModule`.  This interface presents us with the opportunity to implement the `AddRoutes` method where we can register our API endpoints.  Remember `app.MapCarter();` from Programs.cs? Because we have provided an implementation of an `ICarterModule` within this same assesmbly, anything defined with `AddRoutes()` will be auto-discovered and served up as an API endpoint.

### Orders Feature



## Deploy to AKS

### Setup Azure Infrastructure

### Containerize the API

### Push Container to Azure Container Registry (ACR)

### Test the API in AKS

## Create a `dotnet new` template

## Generate a client SDK using OpenAPI Generator
