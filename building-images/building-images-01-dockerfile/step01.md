# Prepare environment

Objective:

Prepare your local environment.

In this step, you will:
- Validate that the environment is initialized
- Explore the sample app

## Validate environment initialization

Please wait until `Environment ready!` appears in the terminal window.

## Explore the sample app

In this scenario, you will be working with a sample application written in Go.
Take a moment first to explore the sample app. 

Go to the directory containing the sample app source code.

```
cd /workspace/hello-go/src
ls
```{{execute}}

Run the app to see that, by default, it returns usage instructions. Since it is the first time you run the app, it will also need to download a string utility package that the app uses.

```
go run hello.go
```{{execute}}

Run it again, this time providing a string. Notice that it returns "Hello", plus the string you provided.

```
go run hello.go world
```{{execute}}

In the next steps, you will building this app into a Docker image using a series of sample Dockerfiles. Each Dockerfile will illustrate a new concept or improvement.
