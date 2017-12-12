Installing the oc Command-Line Tool
===================================

On Red Hat Enterprise Linux (RHEL) systems with valid subscriptions, the tool is available directly as an RPM and installable using the yum install command.

```
[user@host ~]$ sudo yum install -y atomic-openshift-clients
```


Security
========
It is possible to log in as the OCP cluster administrator from any master node without a password by using the system:admin argument for the -u option. This gives you full privileges over all the operations and resources in the OCP instance and should be used with care.

```
oc login -u system:admin
```

Override security policy to allow commands to be run as root.

```
oc adm policy add-scc-to-user anyuid -z default
oc adm policy add-scc-to-user anyuid -n default
```

Creating Applications Using oc new-app
======================================

The oc new-app command can be used, with the option -o json or -o yaml, to create a skeleton resource definition file in JSON or YAML format, respectively. This file can be customized and used to create an application using the oc create -f <filename> command, or merged with other resource definition files to create a composite application.

The oc new-app command can create application pods to run on OpenShift in many different ways. It can create pods from existing docker images, from Dockerfiles, and from raw source code using the Source-to-Image (S2I) process.


To create an application based on an image from a private registry:

```
oc new-app --docker-image=myregistry.com/mycompany/myapp --name=myapp
```

To create an application based on source code stored in a Git repository:

```
oc new-app https://github.com/openshift/ruby-hello-world --name=ruby-hello-world
```

Useful commands to manage Openshift resources
=============================================

There are several essential commands used to manage OpenShift resources. The following list describes these commands.


Typically, as an administrator, the tool you will most likely to use is oc get command. This allows a user to get information about resources in the cluster. Generally, this command outputs only the most important characteristics of the resources and omits more detailed information.


If the RESOURCE_NAME parameter is omitted, then all resources of the specified RESOURCE_TYPE are summarized. The following output is a sample of an execution of oc get pods:

```
[user@host ~]$ oc get pods

NAME               READY   STATUS      RESTARTS  AGE
nginx-1-5r583      1/1     Running     0         1h
myapp-1-l44m7      1/1     Running     0         1h


```


oc get all



If the administrator wants a summary of all the most important components of a cluster, the oc get all command can be executed. This command iterates through the major resource types and prints out a summary of their information.  For example:

```
[user@host ~]$ oc get all


NAME       DOCKER REPO                              TAGS      UPDATED
is/nginx   172.30.1.1:5000/basic-kubernetes/nginx   latest    About an hour ago

NAME       REVISION   DESIRED   CURRENT   TRIGGERED BY
dc/nginx   1          1         1         config,image(nginx:latest)

NAME         DESIRED   CURRENT   READY     AGE
rc/nginx-1   1         1         1         1h

NAME        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
svc/nginx   172.30.72.75   <none>        80/TCP,443/TCP   1h

NAME               READY     STATUS    RESTARTS   AGE
po/nginx-1-ypp8t   1/1       Running   0          1h


```



oc describe RESOURCE RESOURCE_NAME

If the summaries provided by oc get are insufficient, additional information about the resource can be retrieved by using the oc describe command. Unlike the oc get command, there is no way to simply iterate through all the different resources by type. Although most major resources can be described, this functionality is not available across all resources. The following is an example output from describing a pod resource:

```
[user@host ~]$ oc describe pod docker-registry-4-ku34r

Name:              docker-registry-4-ku34r
Namespace:         default
Security Policy:   restricted
Node:              node.lab.example.com/172.25.250.11
Start Time:        Mon, 23 Jan 2017 12:17:28 -0500
Labels:            deployment=docker-registry-4
                   deploymentconfig=docker-registry
                   docker-registry=default
Status:            Running
...
No events

```


oc export

This command can be used to export a definition of a resource. Typical use cases include creating a backup, or to aid in modification of a definition. By default, the export command prints out the object representation in YAML format, but this can be changed by providing a -o option.


oc create

This command allows the user to create resources from a resource definition. Typically, this is paired with the oc export command for editing definitions.


oc edit

This command allows the user to edit resources of a resource definition. By default, this command opens up a vi buffer for editing the resource definition.


oc delete RESOURCE_TYPE name

The oc delete command allows the user to remove a resource from an OpenShift cluster. Note that a fundamental understanding of the OpenShift architecture is needed here, because deletion of managed resources like pods result in newer instances of those resources being automatically recreated. When a project is deleted, it deletes all of the resources and applications contained within it.


oc exec POD_NAME <options> <command>

The oc exec command allows the user to execute commands inside a container. You can use this command to run interactive as well as non-interactive batch commands as part of a script.


oc rsh POD_NAME <options>

The oc rsh command opens an interactive shell session to execute commands inside a container. It is a shorthand for oc exec -it POD_NAME bash.


### Gives detailed output of currently active pods including node location in a continously watching state.
oc get -o wide pods -w --all-namespaces=true


To trigger the deployment of a pod issue the following command:

```
oc rollout latest dc/hello -n s2i
```

Scaled the pods back down to zero:

```
oc scale dc/hello --replicas=0
```