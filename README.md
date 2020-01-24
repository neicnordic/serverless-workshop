# Introduction to serverless functions with OpenFaaS

In this page we introduce a serverless computing framework called OpenFaaS, which promises to help developers to deploy event-driven functions and microservices to Kubernetes without repetitive, boiler-plate coding.

## Requirements
Vagrant and VirtualBox are the only dependencies that need to be available in your host.

In Windows:

- Install Chocolate: https://chocolatey.org/install
- Install Vagrant: https://www.vagrantup.com/downloads.html
- Install VirtualBox v6.0: https://www.virtualbox.org/wiki/Downloads

- Open PowerShell and run: 

```bash
> choco install vagrant
```

In Mac & Linux: 

You should be able to download the installation files for your operating system:

- Install Vagrant: https://www.vagrantup.com/downloads.html
- Install VirtualBox v6.0: https://www.virtualbox.org/wiki/Downloads

> **Note:** It is also possible to use libvirt, but we encourage you to use virtualbox for this tutorial.

## Set up the Vagrant box
After cloning this repository, open your favourite shell and navigate to the folder containing the `Vagrantfile`. Then you can run the following command to spin up the playground:

```bash
> vagrant up --provision --provider [virtualbox | libvirt]
```

> **Note:** Vagrant will request to allocate 3 cpus and 4GB of memory for the machine.

Once it is done, verify that you can ssh into the machine:

```bash
> vagrant ssh
```

Make sure that the OpenFaaS dashboard and Grafana are working by opening a web browser and navigating to http://127.0.0.1:31112 and http://127.0.0.1:31113 respectively.

After you have finished the tutorial, you may destroy the virtual machine using:
```bash
> vagrant destroy
```
