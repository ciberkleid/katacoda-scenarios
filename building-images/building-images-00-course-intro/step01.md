# Prepare environment

Objective:

Prepare your local environment.

In this step, you will:
- Validate that the environment is initialized
- Explore the sample app

## Validate environment initialization

Please wait until `Environment ready!` appears in the terminal window.

## Explore the sample app

Throughout this course, you will be working with a sample application written in Go.
Go to the directory containing the sample app source code.

```
cd /workspace/hello-go/src
ls
```{{execute}}

Run the app to see that, by default, it returns usage instructions.

```
go run hello.go
```{{execute}}

Run it again, this time providing a string. Notice that it returns "Hello", plus the string you provided.

```
go run hello.go world
```{{execute}}

You will be working with this application throughout the rest of this course.