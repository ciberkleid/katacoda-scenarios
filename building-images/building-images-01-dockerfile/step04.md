# Tagging & Build Context

Objective:

Learn about the build context.

In this step, you will:
- Learn why it's important to control the build context
- Learn how to control the build context

## Why control the build context

As we saw earlier the docker build command requires us to specify a context, and the context needs to contain all of the files that the Dockerfile needs in order to build the image.

When you execute a `docker build` command, the `docker` CLI sends the Dockerfile and the contents of the build context to the Docker daemon, and the daemon builds the image.

Scroll up in the terminal window to the last `docker build` command that you executed. Notice the first line in the log output. It should indicate that it is `Sending` data to the daemon.

You can also re-run the build and grep for the "Sending" log info:
```
docker build . -t hello-img | grep Sending
```{{execute}}

**To speed up your builds, it is advantageous to reduce the amount of data that needs to be sent to the daemon.**

Once the context is sent to the daemon, any of the files in the context can be copied to the image.
Run the following command to validate that the `COPY` statement in our Dockerfile copied all the files in the context to the image. This command uses the `--entrypoint` flag to override the ENTRYPOINT defined by the Dockerfile and simply run a shell. The `-it` options makes it interactive with the Terminal:

```
docker run --rm -it --entrypoint /bin/sh hello-img
```{{execute}}

Once inside the container, run the following commands. You can type them into the terminal or copy/paste them:

Run `pwd`{{copy}} to validate that the default directory is “workspace”, as configured by the last WORKDIR instruction in the Dockerfile. Run `ls -l`{{copy}} to list the files in this directory and note that is is all of the files from the build context. Some of these files are not needed for runtime, in particular, the Dockerfile and README. These are adding unnecessary bulk and we’ve created a loophole for potentially confidential or malicious files to make it into our runtime environment.

`Send Ctrl+C`{{execute interrupt T1}} to quit the `docker run` command.

## How to control the build context

A `.dockerignore` file - similar to a `.gitignore` file for git - can serve to filter unwanted files from the docker build context. Create a new file called .dockerignore and configure it to excluding everything by default, and then allow only the application files. 

```
cat << EOF > .dockerignorefile
# Exclude everything by default
*

# Include what is needed to build
!hello.go
!go.mod
!go.sum
EOF

bat .dockerignore
```{{execute}}

Rebuild the image and compare the data sent to the daemon to the last build. You should see that less data is being uploaded now that the `.dockerignore` file is present.

```
docker build . -t hello-img | grep Sending
```{{execute}}

Check the files that were copied to the image this time. You can do so by starting a container and running the sahell again. This time, rather than using the interactive (`--it`) mode, add a command to list the files in-line. you should see that only the necessary files have been copied to the image.

```
docker run --rm --entrypoint /bin/sh hello-img -c "ls -l"
```{{execute}}
