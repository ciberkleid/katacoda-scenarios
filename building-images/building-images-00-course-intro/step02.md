# Step Title Here

Objective:

Explore the behavior of the sample app.

Step objective here.

In this step, you will:
- Do this
- And that

## Section heading here
Explain section


AGENDA SLIDE
In this course we’ll learn a few different ways to turn applications into container images. We’ll start with a brief introduction to containerization and then we'll cover Dockerfile, Jib, and the Cloud Native Buildpacks project using Paketo Buildpacks.

INTRO - SECTION SLIDE
Let's start with a brief introduction to containerization.

INTRO SLIDE 1 - CONTEXT
This flow probably looks familiar. You start with source code, package your application, and deploy it to a virtual machine that’s been prepared for the application.

For example, in the context of Java, this means that you build your war file or your jar file and deploy it to a machine that has been prepared with an operating system, a JRE, potentially an app server, maybe even some shared libraries and configuration. 

More generally, the application package is the deployable artifact, and the virtual machine is the infrastructure abstraction layer that we work with. The gap in between - all of the dependencies the app needs to function - is usually the responsibility of an operations team. 

CLICK - ANIMATION
With container platforms such as Kubernetes, this paradigm changes. Rather than install these dependencies directly onto a virtual machine, CLICK - ANIMATION we bundle them with our application into a package called an image, and we deploy the image to the container platform. These images are our new deployment artifacts.

INTRO SLIDE 2 - CONTAINERIZATION
Why is this shift happening? CLICK - ANIMATION We want to be able to deploy and run software in a repeatable and reliable way across computers. CLICK - ANIMATION The way to accomplish that is to package all dependencies with the application using a standard packaging format. 

CLICK - ANIMATION
So our task is to build this new deployable artifact called an image, using a standard format, namely Docker or OCI - the Open Container Initiative specification. We can then deploy our images to a container platform such as Kubernetes. Containers are simply the running instances of our images.

INTRO SLIDE 3 - ECOSYSTEM
There are a variety of tools available to help us turn our applications into images. In this course, we’ll cover Dockerfile, which enables you to write your own script, and some higher level tools: Jib, a Google plugin for Java applications, and the Cloud Native Buildpacks project using Paketo Buildpacks.

CLICK - ANIMATION
As we go through these, keep in mind the factors for which we want to optimize. We want to make sure that our builds are consistent across applications and across teams and that our builds are quick. We also want to consider security concerns, and provide transparency and governance over the contents of our images, especially over time and at scale.

