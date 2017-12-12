Openshift Container Area of Responsibility
==========================================

## OpenShift Container Platform is responsible for:

* Container Image
* Image Stream
* Build Configuration
* Deployment Configuration
* Adding/Updating Haproxy configuration
* Route/Routing

## Kubernetes is responsible for:

* Replication Controller
* Service


## Docker is responsible for:

* Mount namespace with the OS/Kernel.  Takes your custom container image and puts it in its own mount namespace to isolate the application and its files.  This is what is responsible for file system isolation.


* Network namespace with the OS/Kernel.  Creates a newtwork interface and isolates it with a unique network namepace.


* Inter-Process Communication(s) (i.e. IPC) namespace with the OS/Kernel.  Isolates your application's memory resources.


* Unix Time Share (i.e. UTS) namespace with the OS/Kernel.  Gives your container a unique hostname and domain name.  Example of working with UTS outside of containers: 

``` [user@host ~]$ uname -a ```


* Process Identifier (i.e. PID) namespace with the OS/Kernel.  Gives your container its own PID counter set.  Process running in container is traditionally given pid one where in the host operating system pid one is used to run "init".  



## RedHat Enterprise Linux (i.e. Operating System)

* SELinux context(s) for the container(s).  This allow you to establish fine grained controls which can be used to restrict the manipulation of resources by establishing fixed labels.


* Control groups for the container(s).  Creates a control groups to limit the resource your container can consume.  Allow you to enforce quotas.  Allows you to get rid of noisy neighbor syndrome.
