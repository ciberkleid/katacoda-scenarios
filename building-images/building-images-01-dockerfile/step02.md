# Dockerfile basics

Objective:

Learn the syntax of Dockerfiles, focusing on a handful of basic Dockerfile commands and focusing on controlling the application launch behavior. 
Learn how to execute your Dockerfiles to build images. 
Understand some of the benefits of packaging an application as an image.

In this step, you will:
- Review the basic syntax of a sample Dockerfile
- Build an image from the app source code
- Explore the difference between ENTRYPOINT and CMD instructions
- Understand some of the benefits of using an image as the deployment artifact

## Review Dockerfile syntax

A Dockerfile is a script with the commands needed to assemble an image. 
It consists of a series of commands, which are executed in order from top to bottom.

Copy the first sample Dockerfile to the application source code directory and review the contents.

```
cp /workspace/hello-go/Dockerfiles/Dockerfile .
bat Dockerfile
```

Notice that each command in the script starts with a valid Dockerfile instruction. 
By convention, the instructions are capitalized for readability. 

Instructions are grouped into blocks called _build stages_. 
Each build stage is initialized using a _FROM_ instruction. 
This Dockerfile has a single _build stage_.
 
 The FROM instruction sets the base image for the rest of the instructions in the build stage. 
 For this particular example we need a base image in which we can build and run a Go application. 
 The command 'FROM golang' references a base image called `golang` that is publicly available on Docker Hub. 
 This image has a Linux OS and Go tools installed, so it will provide everything we need.
 
 Since the `golang` image name does not include the image registry and namespace, the default of 'docker.io/library/' will be used. This location is home to a set of curated repositories known as  [Docker Official Images](https://docs.docker.com/docker-hub/official_images). This is one of many sources of base images - make sure any base images you choose to use come from a trusted source.
 
Notice the remaining commands in the Dockerfile:
- LABEL adds metadata that might be useful in the future in inspecting or filtering images
- WORKDIR creates a directory and cd’s into it
- COPY copies the application files to the image
- RUN is used, in this case, to run the command `go install`. This command will automatically create an executable called `hello` and place it in the PATH
- ENTRYPOINT and CMD are used to configure the app launch behavior (more on this shortly)

## Build the image

Execute the Dockerfile by running the `docker build` command. 
Provide the path to the _build context_ (the directory containing the app source code), and a name, or `tag`, for the image.

```
docker build . -t hello-img
```{{execute}}

Look through the output of the build command. 
Notice that the golang base image is downloaded from the library namespace on Docker Hub. 
A new image is created using `golang` as the base, and that the rest of the Dockerfile instructions are carried out in the new image.

Run the following command to list the images on the local Docker daemon. 
The golang base image is now available locally, so any future builds that need it will be faster. 
The new `hello-img` you built is also there. 
You can see from the size of the new image that it includes everything in the base, plus the app.

```
docker images | grep -E '(golang|hello-img)'
```{{execute}}

## Application launch behavior: ENTRYPOINT vs CMD

Run the image using the `docker run` command.

```
docker run hello-img
```{{execute}}

Note that the default behavior of the app using the image is different from the default behavior that you observed in the previous step ("Helo, world!" vs usage instructions). 
This is because the CMD instruction in the Dockerfile is supplying a default argument to the ENTRYPOINT, so the full command executed is `hello world`.

Run the image again, providing an argument after the image name.

```
docker run hello-img sunshine
```{{execute}}

You should get "Hello, sunshine!" - the ENTRYPOINT executes the `hello` program, but the command-line value overrides the default value of the CMD instruction.

More generally, you can specify an executable and arguments in one of three ways:
- Using ENTRYPOINT
- Using CMD
- Using the command-line (`docker run <image> [COMMAND] [ARG...]`)

If you use ENTRYPOINT in combination with CMD, then CMD supplies default arguments for ENTRYPOINT.

In this case, we're using ENTRYPOINT to launch the app and CMD to change the default behavior and return “Hello world” instead of usage instructions.

## Image as artifact

Using an image as an artifact enables the delivery of an app including the full stack that the application needs. In this case, the image includes Linux, the application executable, and the extra Go package that it uses. The image is also properly configured - the PATH environment variable includes the app binary, and the launch command is configured to behave as we desire. This provides much more control, consistency and predictability as compared to delivering only the application binary. We are able to essentially ship the environment with the application. 
