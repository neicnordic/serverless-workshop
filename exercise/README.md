## Exploring the portals

Start by exploring the OpenFaaS UI by opening a web browser and navigating to http://127.0.0.1:31112.

Deploy some sample functions and try to understand how the API works. You can try the Have I Been Pwnd function, which will return the number of hits of a given password. Type a sequence of characters into the *Request* field. Then click *Invoke* and see the response appear in the bottom half of the screen.

Now take a look at the Grafana dashboard by navigating to http://127.0.0.1:31112. OpenFaaS keeps track of metrics about your functions automatically using Prometheus, which can be turned into a useful dashboard.

Four different panels are displayed, containing information about:
* The per-second rate of invocation as measured over the previous X seconds.
* The total function replicas.
* The total number of successful function invocations.
* The average function execution time, as measured over the previous X seconds.

## Using the OpenFaaS API

faas-netes is an OpenFaaS provider which enables Kubernetes for OpenFaaS. Take a look below at a conceptual diagram of the components involved in a typical workflow using OpenFaaS.

![alt text](https://raw.githubusercontent.com/neicnordic/serverless-workshop/master/exercise/of-workflow.png)

In this tutorial, we will use the CLI in order to communicate with the provider. It can also be accessed through its REST API, or through the UI you tested before. Prometheus collects metrics which are available via the faas-netes API for auto-scaling purposes. By changing the URL for a function from /function/NAME to /async-function/NAME an invocation can be run in a queue using NATS Streaming.

### CLI basics

To get a feeling of the different functionalities available, execute the following command:

```sh
faas help
```

With the next command you can list all functions, together with how many replicas you have, the invocation count and the docker image:

```sh
faas ls -v
```

### Creating a function

Before starting, create a new folder for your function:

```sh
mkdir -p myfunction && \
cd myfunction
```

Take a look at all available templates from https://github.com/openfaas/templates.git:

```sh
faas template store ls
```

Pull a template of your choice from the store:

```sh
faas template pull [mytemplate]
```

Here we will demonstrate how to create a Python function, but you can decide to try something else.

```sh
faas new --lang python3 myfunction
```
This will create three files and a directory:

```sh
./myfunction.yml
./myfunction
./myfunction/handler.py
./myfunction/requirements.txt
```

The YAML (.yml) file is used to configure the CLI for building, pushing and deploying your function. In this case, we will work with the images locally, as network resources might be limited.

You can learn more about the function configuration here: https://github.com/openfaas/faas-cli/blob/30b7cec9634c708679cf5b4d2884cf597b431401/stack/schema.go#L14 and https://docs.openfaas.com/reference/yaml/.

Here you can find the contents of a typical config file:

```yaml
provider:
  name: openfaas
  gateway: http://127.0.0.1:31112

functions:
  myfunction:
    lang: python
    handler: ./myfunction
    image: "myfunction:latest"
    labels:
      com.openfaas.scale.min: "2"
      com.openfaas.scale.max: "10"
      com.openfaas.scale.factor: "2"
      com.openfaas.scale.zero: true
      com.openfaas.health.http.path: "/healthz"
      com.openfaas.health.http.initialDelay: "30s"
    environment:
      fprocess: "python index.py"
      read_timeout: 20s
      write_timeout: 20s
      exec_timeout: 40s
      write_debug: false
    limits:
      cpu: 100m
    requests:
      cpu: 100m
    environment_file:
      - env.yml
    secrets:
      - top-secret
    readonly_root_filesystem: true
        
```

* The name of the function is represented by the key under `functions` i.e. `myfunction`.
* The language is represented by the `lang` field.
* The folder used to build from is called `handler`, this must be a folder not a file.
* The Docker image name to be used is under the field `image`.
* Function annotations for autoscaling, environmental variables, the function's process and secrets can also be configured.

Here are the contents of a minimal `handler.py` file:

```python
def handle(req):
    """handle a request to the function
    Args:
        req (str): request body
    """

    return "Hello Geilo!"
```

This function will just return a sequence of characters, so it's indeed an `echo` function. Any values returned to stdout will subsequently be returned to the calling program. Should you want to add more packages, update the `requirements.txt` file. On Kubernetes it is possible to run any container image as an OpenFaaS function as long as your application exposes port 8080 and has a HTTP health check endpoint.

### Building a function and understanding the watchdog

In order to build the function, you might run:

```sh
faas build -f ./myfunction.yml [--shrinkwrap]
```

It is possible to understand how faas builds a function by exploring the "build" folder if you employ the dry run `shrinkwrap` flag. This will not build the function, but instead put in the "build" folder all building blocks of your function, including the watchdog definition. The classic Watchdog forks one process per request giving the highest level of portability, but the newer version enables a http mode where that same process can be re-used repeatedly to offset the latency of forking. Below you can find a conceptual diagram of an invocation of the Watchdog:

![alt text](https://raw.githubusercontent.com/neicnordic/serverless-workshop/master/exercise/watchdog.jpg)

Read more about the Watchdog here: https://docs.openfaas.com/architecture/watchdog/

### Deploying a function

To deploy a function to Kubernetes you might run:

```sh
faas deploy -f ./myfunction.yml
```

Make sure everything goes fine. By default all functions will be located at the openfaas-fn namespace.

```sh
watch -n 5 kubectl get pods -n openfaas-fn
```

This is the standard way for interacting with functions. The function URL follows:

```
[https://gateway_URL:port/function/function_name]
```

## Invoking functions asynchronously or synchronously?

When you call a function synchronously a connection is made to the  OpenFaaS gateway and is held open for the whole execution time. Synchronous calls are *blocking* so your shell will pause and become inactive until the function has finished. 

* The gateway uses a route of: `/function/<function_name>`
* You have to wait until it has finished
* You get the result after the call
* You know if it passed or failed

Asynchronous tasks are slightly different: 

* The gateway uses a different route: `/async-function/<function_name>`
* The client gets an immediate response of *202 Accepted* from the gateway
* The function is invoked later using a queue-worker
* By default the result is discarded

Asynchronous function calls are preferrable for tasks where you can defer the execution until a later time, or you don't need the result on the client.

Let's create a function called `long-task` with its`fprocess` (function process) to `sleep 10`. Setting this variable changes the binary you want to run for each request. You can also accomplish the same behaviour by simply making your function handler sleep. Be careful when choosing the `write_timeout`. This parameter defines the HTTP timeout for writing a response body from your function. This is importsnt, as you might see it terminate prematurely and thus unsuccessfully.

After deploying it successfully, invoke it 5 times synchronously by running:

```
echo -n "" | faas invoke long-task
echo -n "" | faas invoke long-task
echo -n "" | faas invoke long-task
echo -n "" | faas invoke long-task
echo -n "" | faas invoke long-task
```

Then invoke the function 5 times asynchronously:

```
echo -n "" | faas invoke long-task --async
echo -n "" | faas invoke long-task --async
echo -n "" | faas invoke long-task --async
echo -n "" | faas invoke long-task --async
echo -n "" | faas invoke long-task --async
```

What did you observe? The first example should have taken ~50 seconds whereas the second example would have returned to your prompt straightaway. The work will still take similar time to complete, but that is now going to be placed on a queue for deferred execution. The default stack for OpenFaaS uses NATS Streaming for queueing and deferred execution. It is possible to view the logs with the following command:

```
kubectl logs deployment/queue-worker -n openfaas`
```

## Autoscaling function-based workloads

Auto-scaling in OpenFaaS allows a function to scale up or down depending on demand represented by different metrics. Read a bit about how autoscaling works in the following page: https://docs.openfaas.com/architecture/autoscaling/.

OpenFaaS will automatically scale based upon the request per seconds metric as measured through Prometheus. This measure is captured as traffic passes through the API Gateway. If the defined threshold for request per seconds is exceeded an alert will be fired. You can control the minimum amount of replicas for function by setting com.openfaas.scale.min, the default value is currently 1. You can control the maximum amount of replicas that can spawn for a function by setting com.openfaas.scale.max, the default value is currently 20.

Here we will show you how to trigger the autoscaling of functions by using the `hey`
 load testing tool. Run the following command against your function and observe the number of replicas of your function in the Grafana dashboard. What happened?
 
 ```bash
 hey -z=40s -q 7 -c 2 -m POST -d=asdasd http://127.0.0.1:8080/function/myfunction
 ```
 The above simulates two active users `-c` at 5 requests per second `-q` over a duration `-z` of 40 seconds.
 
 You may also want to try to scale your function from 0. If you scale down your function to 0 replicas, you can still invoke it later. Note that this will trigger a cold start, instead of a warm start, which is slower.
 
 It is important to note that there is a difference between applying a scientific method and tooling to a controlled environment and running a Denial Of Service attack on your own laptop. This is not representative of a production deployment.
