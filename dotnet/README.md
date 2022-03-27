# Distroless .NET docker image

## Overview
.NET docker image is built on top of `base-debian11` distroless image. .NET binary is configured to run as nonroot user. Binary is copied from the official microsoft aspnet6.0 image.

## Usage 

### From DockerHub

```
docker run -it --rm -p PORT:PORT <repo>/aspnet:6.0-distroless
```

### Example

We will use hello-world .NET app for this example.
Dockerfile example:

```

# We are going to use debian based images for the builder stage
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS builder

# Set working
WORKDIR /source

# Copy csproj and restore as distinct layers
COPY *.csproj ./

# Dwonload .NET dependencies
RUN dotnet restore

# Copy the source code into the workdir
COPY . .

# Publish the app, we have 2 flags here --runtime(https://docs.microsoft.com/en-us/dotnet/core/rid-catalog) and slef-contained (https://docs.microsoft.com/en-us/dotnet/core/deploying/)
RUN dotnet publish \
    --runtime linux-x64 \
    --self-contained true \
    -c Release \
    -o out

FROM aspnet:6.0-distroless

COPY --from=builder --chown=nonroot:nonroot /source/out /app/

# Change working dir
WORKDIR /app

ENTRYPOINT ["./hello-world"]

```