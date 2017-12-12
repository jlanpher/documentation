Working with Routes
===================

While services allow for network access between pods inside an OCP instance, routes allow for network access to pods from users and applications outside the OCP instance.

A route connects a public-facing IP address and DNS host name to an internal-facing service IP. At least, this is the concept. In practice, to improve performance and reduce latency, the OCP router connects directly to the pods using the internal pod software-defined network (SDN), using the service only to find the end points, that is, the pods exposed by the service.

OCP routes are implemented by a shared router service, which runs as pods inside the OCP instance and can be scaled and replicated like any other regular pod. This router service is based on the open source software HAProxy.


An important consideration for the OCP administrator is that the public DNS host names configured for routes need to point to the public-facing IP addresses of the nodes running the router. Router pods, unlike regular application pods, bind to their nodes' public IP addresses, instead of to the internal pod SDN.

The following listings shows a minimal route defined using JSON syntax:

```
{
    "apiVersion": "v1",
    "kind": "Route",
    "metadata": {
        "name": "quoteapp"
    },
    "spec": {
        "host": "quoteapp.cloudapps.example.com",
        "to": {
            "kind": "Service",
            "name": "quoteapp"
        }
    }
}

```

Starting the route resource, there are the standard, apiVersion, kind, and metadata attributes. The Route value for kind shows that this is a resource attribute, and the metadata.name attributes gives this particular route the identifier quoteapp.

As with pods and services, the interesting part is the "spec" attribute, which is an object containing the following attributes:

*host* is a string containing the FQDN name associated with the route. It has to be preconfigured to resolve to the OCP router IP address.

*to* is an object stating the "kind" of resource this route points to, which in this case is an OCP Service and the "name" of that resource.

*Important*

Unlike services, which uses selectors to link to pod resources containing specific labels, a route links directly to the service resource name.



Creating Routes
===============
To create a route use the oc expose command, passing a service resource name as the input. The --name option can be used to control the name of the route resource. For example:

```
$ oc expose service quotedb --name quote
```

Routes created from templates or from oc expose generate DNS names of the form:

route-name-project-name.default-domain

Where:

route-name is the name explicitly assigned to the route, or the name of the originating resource (template for oc new-app and service for oc expose or from the --name option ).

project-name is the name of the project containing the resource.

default-domain is configured on the OpenShift master and corresponds to the wildcard DNS domain listed as prerequisite for installing OCP.

For example, creating route quote in project test from an OCP instance where the wildcard domain is cloudapps.example.com results in the FQDN quote-test.cloudapps.example.com.


Finding the Default Domain
==========================

The subdomain or, default domain, is defined in the OpenShift configuration file master-config.yaml in the routingConfig section with the keyword subdomain. For example:

```
routingConfig:
  subdomain: 172.25.250.254.xip.io
```

When using the oc cluster up command to run the OpenShift cluster as we do in this course, this configuration file can be found at /var/lib/origin/openshift.local.config/master/master-config.yaml.