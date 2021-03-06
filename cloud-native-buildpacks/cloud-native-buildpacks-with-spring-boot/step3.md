# Build an image with Spring Boot

Let's begin by confirming there are no images yet for our sample app. The following command should return an empty result: 
```
docker images | grep spring-sample-app
```{{execute}}

Build the project using the new `spring-boot:build-image` Maven plugin:
```
cd spring-sample-app
./mvnw spring-boot:build-image
```{{execute}}

You should see evidence of the image build in the log. A few observations:
- When the build-image plugin is invoked, you should recognize the lifecycle phases that we observed with `pack`. 
- Recall that with pack, the Maven build was part of the buildpack build phase. With Spring Boot, the Maven build is done before the buildpack lifecycle is invoked, hence the buildpack lifecycle builds the image using the jar that Maven created, rather than the source code.
- Spring Boot's choice of default builder is pinned to a specific version (0.0.53-bionic). This is [configurable](https://docs.spring.io/spring-boot/docs/2.3.0.M2/maven-plugin/html/#build-image-example-custom-image-builder). As long as we use the same builder across platforms, we can be confident that the image is being built in the same way.
```
[INFO] Building image 'docker.io/library/spring-sample-app:0.0.1-SNAPSHOT'
[INFO]
[INFO]  > Pulling builder image 'docker.io/cloudfoundry/cnb:0.0.53-bionic' 100%
...
[INFO]  > Pulling run image 'docker.io/cloudfoundry/run:base-cnb' 100%
...
[INFO]  > Executing lifecycle version v0.6.1
[INFO]  > Using build cache volume 'pack-cache-5cbe5692dbc4.build'
[INFO]
[INFO]  > Running detector
[INFO]     [detector]    6 of 13 buildpacks participating
...
[INFO]  > Running analyzer
...
[INFO]  > Running restorer
[INFO]
[INFO]  > Running builder
...
[INFO]  > Running exporter
...
[INFO]     [exporter]    Adding 6/6 app layer(s)
...
[INFO] Successfully built image 'docker.io/library/spring-sample-app:0.0.1-SNAPSHOT'
```

Check to see that an image has been created:
```
docker images | grep spring-sample-app
```{{execute}}

Notice that, in contrast with `pack`, the name of the published image is automatically inferred from the application name, and the tag is inferred from the version.

Start your application using `docker run`:
```
docker run -it -p8080:8080 spring-sample-app:1.0.0
```{{execute}}

Send a request to the app. Clicking on the following command will send the request in a new terminal window:
```
curl localhost:8080; echo
```{{execute T2}}

`Send Ctrl+C`{{execute interrupt T1}} to stop the app before proceeding to the next step.

As we saw with `pack` in the previous scenario in this course, subsequent builds will be faster as they will re-use local build and run images, local Maven repository, and cached image layers.