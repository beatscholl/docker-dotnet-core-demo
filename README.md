# Create a ASP .NET Core Web App With Docker

## Introduction

This guide will lead you through the procedure how to create a ASP .NET Core Web App With Docker.

## Prerequisites

The only things we’ll need is a machine with Docker installed and an Internet connection.

## Step 1: Prepare the development container

To create a new a ASP .NET Core Web App we first need to prepare our development environment.

The following command will create and run a docker container containen the .NET Core SDK

``` console
docker-compose up --detach --build
```

Now attach a shell to the running container.

``` console
docker exec -it app-dev bash
```

### Create a ASP .NET Core Web App

To create a new ASP .NET Core Web App run the following command.

``` bash
dotnet new react
```

The project template creates an ASP .NET Core app and a React app. The ASP .NET Core app is intended to be used for data access, authorization, and other server-side concerns. The React app, residing in the ClientApp subdirectory, is intended to be used for all UI concerns.

### Install npm packages

To install third-party npm packages, use a command prompt in the ClientApp subdirectory.

``` bash
cd ClientApp
npm install
```

#### Update npm packages and fix vulnerabilities
At the time of writing, the installed npm packages contained several vulnerabilities. With the following command you can upgrade all local packages.

> We recommend regularly updating the local packages your project depends on to improve your code as improvements to its dependencies are made.

``` bash
npm update 
```

### Test your code

Use the following command to run and test the application.

``` bash
dotnet run
```

From your host machine browse to http://localhost to see your app in action.

### Commit your code

Now it's a good time to do your first commit to your source code repository.

Note that the src folder now contains all the necessary source code without the content of the obj, bin or node_modules folder.

## Prepare the runtime image
In the previous section we describe how to create your development environment. 

In this section we will discuss how to prepare your app to run in a productive environment.

### Multi-Stage builds
With multi-stage builds, you use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don’t want in the final image. To show how this works, let’s adapt the Dockerfile from the previous section to use multi-stage builds.

``` yaml
# Pull down an image from Docker Hub that includes the .NET core SDK: 
# https://hub.docker.com/_/microsoft-dotnet-core-sdk
# This is so we have all the tools necessary to compile the app.
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build

# Fetch and install Node 10. Make sure to include the --yes parameter 
# to automatically accept prompts during install, or it'll fail.
RUN curl --silent --location https://deb.nodesource.com/setup_10.x | bash -
RUN apt-get install --yes nodejs

WORKDIR usr/src/app

# Install dependencies
# https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-restore?tabs=netcore2x
COPY ./src/*.csproj ./
RUN dotnet restore

# Update npm packages
COPY ./src ./
RUN cd ./ClientApp && npm update

# Publishes the application and its dependencies to a folder for deployment to a hosting system.  
RUN dotnet publish --configuration Release --output /publish

# Pull down an image from Docker Hub that includes only the ASP.NET core runtime:
# https://hub.docker.com/_/microsoft-dotnet-core-aspnet/
# We don't need the SDK anymore, so this will produce a lighter-weight image that can still run the app.
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-buster-slim

WORKDIR /app

# Copy the published app to this new runtime-only container.
COPY --from=build /publish .

# To run the app, run `dotnet app.dll`, which we just copied over.
ENTRYPOINT ["dotnet", "app.dll"]
```

To build the docker image use the following command

``` console
docker build .  --file Dockerfile.build --tag app:latest
```

To run the image use the following command

``` console
docker run --detach --publish 80:80 app:lastest
```

Verify you can browse your app at http://localhost