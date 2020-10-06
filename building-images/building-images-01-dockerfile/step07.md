# Basic security considerations

Objective:

Learn basic security strategies, including limiting runtime user privileges and minimizing the container surface area.

Step objective here.

In this step, you will:
- Set the runtime user to a non-root user
- Reducing the size and surface area of image

## Basic security considerations

There are many online resources that cover security for images. In this section we will just scratch the surface with two basic security concerns: the runtime user, and image size.

## Runtime user

Start a container using `hello-img` and check who the runtime user is. You should see that it is root, which has maximum access within the container.

```
docker run --rm --entrypoint /bin/sh hello-img -c "id"
```{{execute}}

Being root in the container does not imply having root privileges on the host VM, but it’s still a concern. 
If an attacker gains access to the container, being root makes it easier to explore and identify other services over the network. 
A savvy hacker might even be able to exploit a vulnerability in the OS and gain access to the host. 

`hello-img` is inheriting the user configuration from the `golang` base image. It makes sense for a base image to use root as the default runtime to enable, for example, OS package installation, but generally - and ideally - applications do not need root access. From a security perspective, we should follow the principle of least privilege and configure the runtime user to have only the access required.

Review the next Dockerfile. Notice the new USER instruction. It sets a non-root user using an arbitrary number as the ID. This is sufficient to change the user and privileges.

```
bat Dockerfile-5
```{{execute}}

Rebuild the image and verify the runtime user.

```
docker build . -t hello-img -f Dockerfile-5
docker run --rm --entrypoint /bin/sh hello-img -c "id"
```{{execute}}

By default, this new user will have read access in the container and write permissions to the “/tmp” directory. In order to grant the user additional permissions or ownership, that needs to be done explicitly through instructions in the Dockerfile. 

## Image size

Image size may seem like an efficiency concern, and to some degree, it is, since image size impacts the first upload or download time of an image. After that, however, only layers that have changed are transferred over the network, so the size of individual layers that change frequently is more important for efficiency than the total size of an image. 

In actuality, the biggest motivation for reducing the size of an image is security. By ensuring that the runtime image contains only the packages and components that are needed for runtime, you can minimize the surface area of attack that a malicious person or program could potentially abuse.

Take a look at the sizes of the golang base image and `hello-img`. You can see that the bulk of `hello-img` comes from the golang base, which is needed to build the application, but not to run the application.

```
docker images | egrep -i 'golang|hello-img'
```{{execute}}

Review the next Dockerfile, which uses an approach called a multi-stage build. In the first stage, it uses the golang base image to build the application. In the second stage, it uses a slimmer base image that has the minimum requirements for runtime and copies the binary built in the first stage into the slimmer base. Notice that the COPY command in the second stage uses a “--from” flag to refer to the first stage, using an alias that is assigned above.

```
bat Dockerfile-6
```{{execute}}

What you need at build time and at runtime depends of course on your application. 
A Java application, for example, needs a JDK to build, but only a JRE to run. 
In this case, the `hello` Go application needed Go tooling for the build, but it can be packaged as a statically-linked (standalone) executable that has no runtime dependencies, so we can reduce the application image size and surface area significantly.
_Scratch_ - the base image name in the second build stage - is a special keyword that enables the initiation of a build stage without adding anything to the image (it does not add any layers to the image). 

It's worth noting that there is a trade-off between image size and functionality. Scratch, for example, doesn't add a container operating system and doesn't have utilities like shell, so it's not adequate for all use cases and it makes it hard to debug. There are a variety of minimal images to choose from with different levels of default functionality. 

Rebuild the image and note that it is significantly smaller.

```
docker build . -t hello-img -f Dockerfile-6
docker images | grep hello-img
```{{execute}}

You can also validate that it still runs. 

```
docker run --rm hello-img "dockerfile ninjas"
```{{execute}}
