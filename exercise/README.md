## Exploring the portals

Start by exploring the OpenFaaS UI by opening a web browser and navigating to http://127.0.0.1:31112.

Deploy some sample functions and try to understand how the API works. You can try the Have I Been Pwnd function, which will return the number of hits of a given password. Type a sequence of characters into the *Request* field. Then click *Invoke* and see the response appear in the bottom half of the screen.

Now take a look at the Grafana dashboard by navigating to http://127.0.0.1:31112. OpenFaaS keeps track of metrics about your functions automatically using Prometheus, which can be turned into a useful dashboard.

Four different panels are displayed, containing information about:
* The per-second rate of invocation as measured over the previous X seconds.
* The total function replicas.
* The total number of successful function invocations.
* The average function execution time, as measure over the previous X seconds.

## Using the OpenFaaS API

faas-netes is an OpenFaaS provider which enables Kubernetes for OpenFaaS. In this tutorial, we will use the CLI in order to communicate with the provider.

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

Make sure you pull the template you want from the store:

```sh
faas template pull [mytemplate]
```

Here we will demonstrate how to create a Python function, but you can decide to try something else.

* Scaffold the function

```sh
$ faas-cli new --lang python3 myfunction
```
This will create three files and a directory:

```sh
./myfunction.yml
./myfunction
./myfunction/handler.py
./myfunction/requirements.txt
```

The YAML (.yml) file is used to configure the CLI for building, pushing and deploying your function. In this case, we will only build the work with the images locally, as network resources might be limited.

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
    environment:
      read_timeout: 20s
      write_timeout: 20s
    limits:
      cpu: 100m
    requests:
      cpu: 100m
    fprocess: "python index.py"
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

Here is the contents of a minimal `handler.py` file:

```python
def handle(req):
    """handle a request to the function
    Args:
        req (str): request body
    """

    return req
```

This function will just return the input, so it's indeed an `echo` function. Any values returned to stdout will subsequently be returned to the calling program.
