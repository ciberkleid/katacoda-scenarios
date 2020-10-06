**Course Overview**

This scenario is part of a course on [Building Images](https://www.katacoda.com/ciberkleid/courses/building-images).

In recent years, container technologies, mainly Docker and Kubernetes, have gained an enormous amount of traction. 
Containerization has myriad benefits, including greater predictability and portability of applications, since the unit of deployment - aka, the image - includes not only the application, but all of its dependencies as well. 

Development and DevOps teams benefit because they can better control all of the components that are deployed with the application.
Operations teams benefit because they no longer need to provision and maintain custom environments for each application or application type. 
Instead, they can focus on providing a more generic container runtime system, like Kubernetes, where a number of images can be deployed and run. 
Containers, after all, are merely running instances of images.

In this course, we cover three approaches for packaging applications and their dependencies as images: Dockerfile, Jib, and Cloud Native Buildpacks.

**Scenario Overview**

Jib is a tool for building images from Java applications without using the Docker daemon or Dockerfiles. 
In this scenario, you will learn how to use Jib's Maven and Gradle build plugins to configure and build images.
