# Image tags and tagging practices

Objective:

Learn about image tags.

In this step, you will:
- Understand when and how to use tags when creating and using images

## Assigning tags when creating images

A tag is an alias to an image ID. Notice that both the base and app images on your local Docker daemon are tagged as “latest,” which is the default value assigned if no other tag is specified. “Latest” implies that this is the most recent edition of a given image.

```
docker images | grep -E '(golang|hello-img)'
```{{execute}}

It is useful - not to mention "best practice" - to assign tags to convey more specific information about the version or variant. 

Assign two additional tags to `hello-img` to specify a specific version as well as a more general version group.

```
docker tag hello-img hello-img:1.6
docker tag hello-img hello-img:1
docker images | grep hello-img
```{{execute}}

Notice that all three tags are pointing to the same image ID. Users of this image can specify any of these tags, and they will get the same image as a result.

Make a change and rebuild in order to create an image with a new ID. Any change to the app or the Dockerfile will produce a different ID, so you can simply change the default CMD in the Dockerfile for the purposes of this example. Tag the new image as 1.7 and assign it the more general version 1 and latest tags as well.

```
sed -i '' 's/world/sunshine/g' Dockerfile
docker build . -t hello-img:1.7 -t hello-img:1 -t hello-img
docker images | grep hello-img
```{{execute}}

Notice that the more generic tags have been moved to the new image ID. Image tags, unlike image LABELs, are mutable, meaning they can change over time. If a user were to pull `hello-img:latest` now, they would get a different image than they would have a few minutes ago. This may be acceptable or even desirable for early development. However, for a production deployment, the most specific tag should be used to guarantee that a particular image is used. As creators of images, it is important to assign and manage tags in a way that is informative and useful to the users of our images. 

## Specifying tags when pulling images

It is also important to consider tagging images in the FROM commands in our Dockerfiles. For example, our Dockerfile simply states `FROM golang`, which defaults to `golang:latest`. We can safely assume that this tag will be moved to a different image ID as soon as a new golang image is available. This might cause us to inadvertently incorporate base image changes into `hello-img` when our Dockerfile is executed on a different machine or at another point in time.

Run the following command to see an example of the Dockerfile with the base image tag specified.

```
bat /workspace/hello-go/Dockerfiles/Dockerfile-1
```{{execute}}