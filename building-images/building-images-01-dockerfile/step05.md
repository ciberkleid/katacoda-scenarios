# Step Title Here

Objective:

Learn how Dockerfile configuration can impact application shutdown behavior.

In this step, you will:
- Understand shutdown behavior in containers
- Test app shutdown when the app is the parent process
- Test app shutdown when the shell is the parent process
- Explore two solutions for ensuring the app receives system shutdown signals

## Graceful shutdowns

Applications often include logic for a graceful shutdown. 
When the application receives a shutdown signal, it executes this logic in order to shut down in a deliberate, controlled manner.

When working directly with an application running on a VM, an application owner or system admin can send a shutdown signal to the application process directly.
On container runtime systems, however, the app owner or system admin (or sometimes an admin process such as a cluster auto-scaler or a pod eviction policy) instructs the container runtime system to send a shutdown signal to the container. The shutdown signal is passed to the parent process in the container, which is the process with Process ID 1, or `pid 1`. If your app was the first process to start in the container, it will be PID 1. If, however, another process was launched first (e.g. a shell process), and that process launched your app, you are relying on the parent process to pass the shutdown signal to your app.

If the shutdown signal is not passed to your app, or if your app does not shut down within a certain amount of time after receving the shutdown signal, the container runtime system will send a kill signal and force the container to stop. In this case, your app will not be given an opportunity to shut down gracefully.

Therefore, it is good practice to validate that your app receives and responds to the shutdown signal sent to the container.

## Test app shutdown with app as parent process

The process you launch through your ENTRYPOINT or your CMD is launched as PID 1. Recall the ENTRYPOINT from our sample app.

```
bat Dockerfile | grep ENTRY
```{{execute}}

This ENTRYPOINT syntax with brackets means that hello will be launched as PID 1 (without the brackets, a shell is launched as PID 1, and the shell starts the ENTRYPOINT process).
This means that the shutdown signal to the container will go directly to the app process. 

You can confirm that the app receives and responds to the shutdown signal by starting a container and using Ctrl+C to stop it.
The HELLO_SLEEP environment variable will cause the application to stay up, giving you an opportunity to carry out this test.

```
docker run --rm -e HELLO_SLEEP=1 hello-img
```{{execute}}

`Send Ctrl+C`{{execute interrupt T1}} to stop the container.
The app should stop right away.

This simple test confirms that the Go tooling alone builds the app in a way that properly traps the shutdown signal. 
If we had more sophisticated shutdown logic coded into the app, it would be given chance to execute.

## Test app shutdown with shell as parent process

Sometimes, we need to use more complex logic to start an application, in which case one option is to configure the ENTRYPOINT or CMD to launch a shell, or even a shell script, to start our application.
Review the following example - it uses a shell to combine multiple user arguments into a single string to pass to `hello`. 
In this case, the shell process will be PID 1, and `hello` will be a child of the shell.

```
cp ../Dockerfiles/Dockerfile-2 .
bat Dockerfile-2 | grep ENTRY
```{{execute}}

Build and run a container using this new Dockerfile.

```
docker build . -t hello-img -f Dockerfile-2
docker run --rm --name hi -e HELLO_SLEEP=1 hello-img rising stars
```{{execute}}

`Send Ctrl+C`{{execute interrupt T1}} to stop the container.
The app will continue to run, ignoring the shutdown signal.
This shows that either shell (PID 1) is not trapping the shutdown signal, or it is trapping it but not forwarding it to the `hello` process. In this case, the latter is true.

In a new terminal, check the processes running inside the container. You can use the nickname of `hi` that you asssigned in the `docker run` command above to easily access the running container. Notice that shell is PID 1 and hello is a child of the shell.

```
docker exec hi ps -ef
```{{execute T2}}


Now, stop the container using `docker stop`. After about 10 seconds, the container should stop. This is because `docker stop` first sends a shutdown signal, as you did manually using `Ctrl+C`. After 10 seconds, docker sends a kill signal. This forces the container to shut down, without waiting for the app and shell processes to stop first.

```
docker stop hi
```{{execute T2}}

## Solution 1: Elevate the app process to PID 1

You have seen that the app as PID 1 properly traps the shutdown signal, but with shell as PID 1, the app never receives the signal. In this case, you can easily deduce that elevating the app process to PID 1 after the shell launches the app would be sufficient to solve the problem.

This can be achieved by simply adding the keyword `exec` to the application launch command. Review a sample Dockerfile that implements this solution.

```
bat ../Dockerfiles/Dockerfile-3-exec | grep ENTRY
```{{execute}}

In general, you should try to keep your containers to a single process rather than a complex process tree, so even in cases where you might need to use shell to launch your process, elevating the app process using exec is usually sufficient.

## Solution 2: Use a purpose-build parent process to handle shutdown signals

In some exceptions, you may run into a situation where elevating a process is not sufficient. Modern languages and frameworks generally handle trapping the shutdown signal weel, but it's possible to run across an exception. Or you may encountar a case where you cannot keep the container to a single process. In these cases, you can uses a special initiailization process as PID 1 that is purpose-built to trap system signals and forward them to all processes in the process tree. One example is [tini](https://github.com/krallin/tini).

**Using tini:**

`tini` is built into docker, so you can enable it when you create a new container from an existing image. Use `docker run` with the `--init` flag to re-run the last container you built.

```
docker run --init -e TINI_KILL_PROCESS_GROUP=1 --rm --name hi -e HELLO_SLEEP=1 hello-img rising stars
```{{execute}}

`Send Ctrl+C`{{execute interrupt T1}} to stop the container.
The app should stop right away. This is the same container that did not previously respond to `Ctrl+C`.

If you want the container to launch `tini` as PID 1 by default, without requiring someone to remember to include it at runtime, then you can include `tini` directly in your ENTRYPOINT OR CMD. In this case, you also need to install it in the golang base image. Look through the next sample Dockerfile for an example.

```
cp ../Dockerfiles/Dockerfile-3 .
bat Dockerfile-3
```{{execute}}

This Dockerfile will create an image that will launch tini as PID 1. The shell will be a child of tini, and the app will be a chalid of the shell, grandchild of tini. The “-g” flag in the ENTRYPOINT is equivalent to the `TINI_KILL_PROCESS_GROUP` environment variable in the `docker run` command: it ensures tini sends the signal to the group - both shell and hello - not just to its direct child.

Validate that `tini` properly forwards the signal to `hello`.

```
docker build . -t hello-img -f Dockerfile-3
docker run --rm -e HELLO_SLEEP=1 hello-img
```{{execute}}

In a separate terminal window, check the process tree. Verify that `tini` is PID 1, parent to the shell, and grandparent to `hello`.

```
docker exec hi ps -ef
```{{execute T2}}

`Send Ctrl+C`{{execute interrupt T1}} to stop the container.
The app should stop right away.
