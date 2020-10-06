# Image layers & build cache

Objective:

Learn about image layers and build cache.

Step objective here.

In this step, you will:
- Explore the layered nature of Docker images
- Understand the ability of layers to function as cache
- Review strategies for maximizing caching opportunities during image builds

## Image Layers

Images are composed of _layers_. 
Each Dockerfile instruction creates, or _commits_, a new layer.
The committed layers are similar to git commits. 
For example, if one Dockerfile instruction adds a file, and the next Dockerfile instruction deletes the file, the file is "removed" from the image, but the image does not actually get smaller in size, just as a git repo does not get smaller when a file is deleted, since the deleted file is still in the history in the previous commit (aka layer).

You can see the layers that comprise an image using the `docker history` command, including the Dockerfile instruction that created each layer. On the bottom you can see the layers inherited from the golang base, and on the top are the layers added by our Dockerfile.

```
docker history hello-img
```{{execute}}

Notice that the layers have different ages. This shows that for every Dockerfile instruction, if nothing has changed, Docker will reuse an existing layer rather than create a duplicate. To determine if something has changed, Docker will compare the checksum of any files in ADD or COPY instructions to the checksum of the file in cached layers. For other instructions, it compares the instruction itself to the instruction used to generate cached layers. If there is a change, then Docker will create a new layer, and it will continue to create new layers for all remaining instructions in the Dockerfile.

## Maximize caching

Now that you understand layers and caching, you can appreciate that the grouping and ordering of commands in a Dockerfile will impact how efficiently it can reuse cached layers.

In general, there are three main considerations when maximizing the utilization of cache for Dockerfile builds: the order of Dockerfile instructions, the grouping of commands within a Dockerfile instruction, and the contents that each command will commit to the layer.

Take a moment to reivew the last Dockerfile. This Dockerfile has some flaws. Try to identify as many as you can.

```
bat Dockerfile-3
```{{execute}}

The COPY instruction, for example, copies all files from the context at once and it does so early on in the Dockerfile. This means that any change to the source code will essentially all layers to be recreated, which means `tini` will and the Go package will be downloaded and installed during each build. 

In addition, the installation of tini is spread across two RUN instructions. If the second one is modified - for example, if another package is added to the list of packages to install - you might end up running the install against a potentially outdated “apt-get update” layer. 

There are some more subtle improvements that can be made. Take a look at an improved Dockerfile. 

## Review an improved Dockerfile

```
bat Dockerfile-4
```{{execute}}

This improved Dockerfile makes the reasonable assumption that OS packages will probably not change often, Go packages may occasionally change, and source code will change most frequently. It therefore organizes the instructions so that the addition of each to layers happens separately and in that order. Notice that the COPY statement that previously copied files from the build context to the image is now split across two COPY statments that appear later in the file. Now if you change only the source code, you'll reuse the layer with the OS package and the layer with the Go package and simply rebuild the executable.

The Dockerfile also groups all of the commands to install tini into a single instruction and it removes unnecessary bulk from the layer before committing it to the image. You can see that it includes a flag to avoid installing unnecessary files such as documentation, and it removes the package cache. This Dockerfile also includes a version for tini, which provides control over the version that’s installed as well as a way to invalidate the cache at this instruction should you want to change the version. 


Validate the improvements building an image without using cache. You can use the `time` utility to see how long the build takes.

```
time docker build . -t hello-img -f Dockerfile-4 --no-cache
```{{execute}}

Make a change to the source code and rebuild.

```
sed -i '' 's/Hello/Greetings/g' hello.go
time docker build . -t hello-img -f Dockerfile-4
```{{execute}}

Read through the log to validate that most layers are being pulled from cash, up until the COPY command for the source code. 

If the Dockerfile had not been improved, the second build would have taken just as long as the first, but you can easily compare the times of the two builds to see that the second was much faster than the first.
