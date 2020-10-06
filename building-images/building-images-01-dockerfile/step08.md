# Inspecting Images

Objective:

Learn about inspecting images.

Step objective here.

In this step, you will:
- Inspect an image using `docker inspect`
- Inspect an image using `dive`

## Inspect images with `docker inspect`

The `docker inspect` command provides low-level metadata about images or containers.
Use it to inspect `hello-img`. The output is in JSON format, so piping the output to the `jq` utility makes it easier to read.
Scroll through the output to familiarize yourself with the kind of information it provides. 
For example, you can see the full ID, which is a SHA256 digest that uniquely identifies the image, as well as the image name and tag, and more. 
The Container and ContainerConfig sections refer to a temporary container that is created when the image is built, but there’s Config section lower down that refers to the image.

```
docker inspect hello-img | jq
```{{execute}}

To filter the output to return specific pieces of information that you might be interested in, you can use `jq` filtering capabilities and/or formatting/templating capabilities of ‘docker inspect’ itself. 
There are many resources online that cover the syntax of both formatting syntaxis.

For example, you can get all of the top-level keys.

```
docker inspect hello-img -f '{{json .}}' | jq keys
```{{execute}}

Or, you can get just the Config section.

```
docker inspect hello-img -f '{{json .Config}}' | jq
```{{execute}}

## Inspect images with `dive`

To examine the contents of individual layers, you can use a tool called `dive`. 
Use the following command to start `dive`. 
You should see the image layers listed on the left and the layer contents on the right. 

```
dive hello-img
```{{execute}}

Click the `tab` key to toggle between the two. 
This scratch-based image only has one layer, which contains the app executable. Exit `dive` with `Ctrl+C`{{execute interrupt T1}}.

Take a look at a more complex image. 
For example, you can re-build the image using an earlier single-stage Dockerfile.

```
docker build . -t hello-img -f Dockerfile-5
dive hello-img
```{{execute}}

At the top left of the screen, you can see that the first layers are from the golang base, followed by layers created by the Dockerfile. 
Use the downward arrow to highlight the layer that creates the workspace directory.

Click on the `tab` key to toggle to the layer contents on the right of the screen.

Click `CTRL+U` to filter the layer contents so that they show only what changed when this layer was added to the filesystem. 
You can see that the only change was the creation of the workspace directory.

Leave `Ctrl+U` enabled to see only changes and click on `tab` again to toggle back to the list of layers. 
Use the downward arrow to explore the next layers and try to correlate them with the instructions in the Dockerfile. 
When you are finished, exit `dive` with `Ctrl+C`{{execute interrupt T1}}.
