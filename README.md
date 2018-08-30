# Getting Started with Functions

## Installation

One platform-independent way to install the platform is by downloading the installer via the command-line tool `curl`:
```
curl -LSs https://raw.githubusercontent.com/fnproject/cli/master/install | sh
```

As part of the installation, the Fn command-line interface (CLI) is now available. There are a number of ways to integrate with the Fn platform, and the Fn CLI is currently the easiest one.

If you want a list of functions that are supported by the CLI, simply enter the command `fn` at the command prompt and press enter.

## Demo Function


The Fn platform allows you to execute functions in any language. This article focuses on Java functions. Apart from the Java code for your function, some minor metainformation is needed in a configuration file named func.yaml. The Fn CLI tool allows you to easily generate a default function and a default configuration file, as follows:
```bash
fn init --runtime java
```

The fn init command causes the creation of a new function. This basically means a template is applied in the current directory. If you specify the --runtime java parameter, a Java template is used and a Maven project file (pom.xml) file is created, together with a very simple Java file. Also, a func.yaml file is created that contains the metainformation.

The generated func.yaml file looks as follows:
```yaml
version: 0.0.1
runtime: java
cmd: com.example.fn.HelloFunction::handleRequest
build_image: fnproject/fn-java-fdk-build:jdk9-1.0.55
run_image: fnproject/fn-java-fdk:jdk9-1.0.55
format: http
```

The information contained in this file is needed for the Fn platform to create a Docker image that holds your function.

At this point, the most important entry in that file is the following:
```txt
cmd: com.example.fn.HelloFunction::handleRequest
```

This entry states that when the function is called, the ```method handleRequest``` on the ```com.example.fn.HelloFunction``` class will be invoked. The fn init command created this class already, as part of the default template. It looks as follows:
```java
package com.example.fn;

public class HelloFunction {

    public String handleRequest(String input) {
        String name = (input == null || input.isEmpty()) ? "world"  : input;
        return "Hello, " + name + "!";
    }

}
```

This is a very simple Java class and the behavior of the ```handleRequest``` function is self-explanatory. You can, of course, change this code and make it more suitable for your real-world applications. Chances are that you will need more than just a class with a one-line function. You can add more classes and move them to different packages.

The ```fn init``` command also generated a Maven ```pom.xml``` file that contains the build and packaging instructions, and you can modify that file as well, for example, to introduce dependencies.

If you don't want to use Maven, you can use Gradle or any other build system, and I will show later how you can do this with slightly modified build instructions.

###Running a Function

At this point, let's execute the function that was auto-generated. There are a number of options:

* Use `fn run` to build a Docker image and run the function in a standalone environment.
* Use `fn deploy` and other commands to deploy your function in the Fn server.

Clearly, if you really want to take advantage of an FaaS platform, you need the second option. You want to package your function and deploy it somewhere to a service that will then manage it for you.

But for testing purposes, it is often handy to simply run the functions locally. The fact that you don't have to modify code if you want to switch between local deployment and deployment in a cloud environment is a huge advantage.

Running the function locally can be done using the `fn run` command.

The output of the command is the following:
```
Building image javatwo:0.0.1 .........................................................................
Hello, world!
```
The first time this command is executed, it might take some time. If you run the command a second time, it will be much faster.

Although as a developer you don't need to worry about the implementation details, it might be interesting to understand how the Fn project is doing this.

This command performs two operations: it builds the function and it runs the function. How this is done is abstracted away from the developer.

The Fn CLI tool manages building and locally running functions for you by using a multistage Docker build process. Building the function and running the function each happen in a separate Docker stage.

In our simple case, building the function means executing the right Maven command in the right environment. Because Fn knows that you want to use a Java environment, it will execute the build commands in a Docker image that has all the required tools (for example, the Java 9 SDK).

The second step runs the function. This starts a new Docker container with the correct environment, and it launches the function specified by the cmd argument in the `func.yaml` configuration file.

The key point for developers is that you should focus on your function, not on how to run that function on a specific configuration or infrastructure.

The `fn run` command executes your function inside a specific Docker container, but it doesn't integrate it into an FaaS environment yet. The Fn platform, which provides an FaaS environment, adds functionality such as routing, asynchronous execution, versioning, logging, and so on.

If we want to deploy our function to the Fn server, we first need to start the Fn server. This can be done using the following command, which is part of the CLI tool:
```
fn start
```
Starting the server gives the following output:
```
fn start
Unable to find image 'fnproject/fnserver:latest' locally
latest: Pulling from fnproject/fnserver
ff3a5c916c92: Pull complete 
1a649ea86bca: Pull complete 
ce35f4d5f86a: Pull complete 
b6206661264b: Pull complete 
b8b71dba24d3: Pull complete 
3873004a68ee: Pull complete 
f4205b132661: Pull complete 
91a85eeeb257: Pull complete 
93c96d032b32: Pull complete 
bb761748d6e1: Pull complete 
81f6c51c4ac2: Pull complete 
2ba715696dba: Pull complete 
f46c2b56aaf3: Pull complete 
0e2053c1d08d: Pull complete 
06e7bebac2bd: Pull complete 
Digest: sha256:5f9c2f7e173ffb4fbf5228ff8965cbb14fda1b37a3c039c6fc9691569063d034
Status: Downloaded newer image for fnproject/fnserver:latest
time="2018-08-29T15:53:59Z" level=info msg="Registering container driver 'docker'"
time="2018-08-29T15:53:59Z" level=info msg="Registering log provider 's3'"
time="2018-08-29T15:53:59Z" level=info msg="Registering data store provider 'sql'"
time="2018-08-29T15:53:59Z" level=info msg="Registering log provider 'sql'"
time="2018-08-29T15:53:59Z" level=info msg="Registering sql helper 'mysql'"
time="2018-08-29T15:53:59Z" level=info msg="Registering sql helper 'postgres'"
time="2018-08-29T15:53:59Z" level=info msg="Registering sql helper 'sqlite'"
time="2018-08-29T15:53:59Z" level=info msg="Setting log level to" level=info
time="2018-08-29T15:53:59Z" level=info msg="mysql does not support sqlite3"
time="2018-08-29T15:53:59Z" level=info msg="postgres does not support sqlite3"
time="2018-08-29T15:53:59Z" level=info msg="mysql does not support sqlite3"
time="2018-08-29T15:53:59Z" level=info msg="postgres does not support sqlite3"
time="2018-08-29T15:53:59Z" level=info msg="Connecting to DB" url=/app/data/fn.db
time="2018-08-29T15:53:59Z" level=info msg="datastore dialed" datastore=sqlite3 max_idle_connections=256 url="sqlite3:///app/data/fn.db"
time="2018-08-29T15:53:59Z" level=info msg="agent starting cfg={MinDockerVersion:17.10.0-ce DockerNetworks: DockerLoadFile: FreezeIdle:50ms EjectIdle:1s HotPoll:200ms HotLauncherTimeout:1h0m0s AsyncChewPoll:1m0s CallEndTimeout:10m0s MaxCallEndStacking:8192 MaxResponseSize:0 MaxLogSize:1048576 MaxTotalCPU:0 MaxTotalMemory:0 MaxFsSize:0 PreForkPoolSize:0 PreForkImage:busybox PreForkCmd:tail -f /dev/null PreForkUseOnce:0 PreForkNetworks: EnableNBResourceTracker:false MaxTmpFsInodes:0 DisableReadOnlyRootFs:false DisableDebugUserLogs:false}"
time="2018-08-29T15:53:59Z" level=info msg="no docker auths from config files found (this is fine)" error="open /root/.dockercfg: no such file or directory"
time="2018-08-29T15:53:59Z" level=info msg="available memory" cgroupLimit=9223372036854771712 headRoom=268435456 totalMemory=968294400
time="2018-08-29T15:53:59Z" level=info msg="sync and async ram reservations" availMemory=699858944 ramAsync=559887156 ramAsyncHWMark=447909724 ramSync=139971788
time="2018-08-29T15:53:59Z" level=info msg="available cpu" availCPU=4000 totalCPU=4000
time="2018-08-29T15:53:59Z" level=info msg="sync and async cpu reservations" cpuAsync=3200 cpuAsyncHWMark=2560 cpuSync=800

        ______
       / ____/___
      / /_  / __ \
     / __/ / / / /
    /_/   /_/ /_/
        v0.3.544

time="2018-08-29T15:53:59Z" level=info msg="Fn serving on `:8080`" type=full
```

The Fn server, which is a Docker image itself, will take care of the bookkeeping required for dealing with functions and applications.

In order to deploy our function to the Fn server, we use the CLI command `fn deploy`:
```
fn deploy --app demo --local
```
We use the `--local` argument to indicate that we want to send the generated Docker image to our local environment as opposed to registering it on Docker Hub. I will talk about Docker Hub later, but for now, we want to do all development in our local development environment.

The `--app demo` argument indicates that we want to deploy our function as part of the "demo" application. If that application doesn't exist yet, it will be created.

The function can now be identified by the combination of the application name (demo) and the function name (by default, the name of the directory).

The result of the `fn deploy` command is the following:
```
Deploying javatwo to app: demo at path: /HelloFunction
Bumped to version 0.0.2
Building image javatwo:0.0.2 ..
Updating route /HelloFunction using image javatwo:0.0.3...
```
Note that each time a function is deployed to the Fn server, the version of that function is incremented by one. You can verify that by inspecting the func.yaml file, which now has this entry:
```
version: 0.0.2
```
 

The function is now deployed on the Fn server. In order to run it via the Fn server (as opposed to directly running the Docker image using fn run, as explained before), we have a few options:
* Use the Fn CLI.
* Use an HTTP trigger.

Invoking a function via the Fn CLI tool can be done using this command:
```
fn call <app> <function>
```
In our case, this boils down to the following:
```
fn call demo HelloFunction
```
 

The result of this execution is simply the following:
```
Hello, world!
```
 

The second approach for invoking a function is using an HTTP trigger. When a function is registered to the Fn server, a route is created that can be accessed over HTTP. The following syntax should be followed:
```
curl http://<host>:8080/r/<app>/<function>
```
In our simple demo, this leads to the following REST call, which gives the same output as above:
```
curl http://localhost:8080/r/demo/HelloFunction
```
In both these options, the function wasn't directly invoked by the CLI tool. Rather, the CLI tool or the REST call triggered the Fn server to invoke the specific function that is part of a specific application.

In a real production environment, the Fn server will also be the entity that triggers the function invocations. But as you could see, the result of invoking a function directly (via fn run) or via the Fn server produces the same output, and that makes it very convenient to develop these functions.

##References
[An Introduction to the Fn Project](https://developer.oracle.com/java/fn-project-introduction)

##Credits
All credits goes to Johan Vos, author of original blog article from which this README has been extracted and fixed.