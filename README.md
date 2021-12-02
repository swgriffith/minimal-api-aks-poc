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

![Source Files](/assets/project-organization.png)

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

The Inventory feature is a straightforward Carter Module that contains one endpoint.  Notice that `InventoryModule` implements `ICarterModule`.  This interface presents us with the opportunity to implement the `AddRoutes()` method where we can register our API endpoints.  Remember `app.MapCarter();` from Programs.cs? Because we have provided an implementation of an `ICarterModule` within this same assesmbly, anything defined within `AddRoutes()` will be auto-discovered and served up as an API endpoint.

`app.MapGet("/inventory", GetInventory).WithTags("Inventory");`

This line will serve up an API endpoint at `/inventory` that will call the `GetInventory` method.  `.WithTags("Inventory")` allows us to take advantage of some built-in OpenAPI classification features within Minimal API.  To see this in use, take another look at the Swagger/OpenAPI UI that is shown when launching the API.  You will see that this `/inventory` endpoint is classified under the **Inventory** category.

### Orders Feature

The Orders feature is implemented as a Carter Module as well.  Take a minute to look over the functionality within this module.  You will notice that this module uses an implementation of `IOrderService` (which was registered in Program.cs) that provides a simple caching mechanism for demonstration purposes.  A few things of note:

- `.WithName` is used to assign a name to a few endpoints.  This is used to provide a convenient mechanism for generating links within the API.  Notice this usage within the `NewOrder` method.
- You likely noticed an additional folder within the **Orders** feature folder named **Validators**.  This folder contains some reusable FluentValidation validators.  You can see these validators being used throughout the `OrdersModule`.

## Next

Now that you have run the API locally, lets [deploy it to AKS](deploy-to-aks.md).

## Create a `dotnet new` template

What if you want to share a Minimal API template with your teammates?  Augmenting `dotnet new` with an additional template is a simple, easy way to do just that.

In the repo, you will notice that there is a `.template.config` folder at the root of the project that contains a single `template.json` file.  The contents of this file defines metadata that will be used during the creation of a `dotnet new` template.

To install a new template, navigate to the root of the API project (meaning, one level above the .template.config folder) and run the following command:

`dotnet new --install .\`

NOTE: If you are running on OSX the command would look like this: `dotnet new --install ./`

A new template will now exist in `dotnet new` that will contain the contents of the current API project.  You can check that the template has been installed by executing `dotnet new --list`.  You should see a template named **minimalapistarter** (as it was named in the template.json file).

If you would like to test out using the template, navigate to an empty folder and execute the following command:

`dotnet new minimalapistarter`

Congratulations!  You now have a templatized project you can share with teammates and re-use.

## Generate a client SDK using OpenAPI Generator

Last but not least, since we have created an API that exposes an OpenAPI definition, lets go through the exercise of generating a client SDK from this API definition.  To do this, we are going to use [OpenAPI Generator](https://openapi-generator.tech/).

### Installation

To install OpenAPI Generator, you can install it as an npm package using the following:

`npm install @openapitools/openapi-generator-cli -g`

NOTE: If you are running on OSX and are a hombrew user, you can execute the following:

`brew install openapi-generator`

### Generate an SDK

At this point, you need to decide what API location you will be pointing at to generate your SDK.  Meaning, in this sample repository, we have run the Minimal API both locally and in AKS.  You can generate a client SDK from either.  You will be utilizing the swagger.json endpoint of the exposed OpenAPI specification on your API.

#### API Hosted in AKS

To generate an SDK from the API hosted in AKS, run the following command:

`openapi-generator generate -i https://[YourKubernetesEXTERNAL-IPHere]:8080/swagger/v1/swagger.json -g csharp -o ./tmp/apisdk`

The previous command will place the generated C# SDK in a /tmp/apidk folder relative to the current location where you ran this command.  Browse the generated SDK to learn more about what OpenAPI Generator generates for you.  Note: OpenAPI Generator also generates documentation specific to your API located in the /docs folder.

#### API Hosted Locally

To generate an SDK while running the API locally, run the following command (fill in the port that your API is running on):

`openapi-generator generate -i https://localhost:[YourPortHere]/swagger/v1/swagger.json -g csharp -o ./tmp/apisdk`

NOTE: If you encounter an error indicating a SSHHandshakeException, you will need to temporarily disable certificate verification by adjusting the following environment variable.

Windows/Powershell

`[System.Environment]::SetEnvironmentVariable('JAVA_OPTS','-Dio.swagger.parser.util.RemoteUrl.trustAll=true -Dio.swagger.v3.parser.util.RemoteUrl.trustAll=true')`

OSX

`export JAVA_OPTS="-Dio.swagger.parser.util.RemoteUrl.trustAll=true -Dio.swagger.v3.parser.util.RemoteUrl.trustAll=true"`

### Additional Information

The above examples generate C# client SDKs, however, the OpenAPI Generator can generate SDKs in many more languages.  Please reference [this documentation](https://openapi-generator.tech/docs/generators/) for further information.
