Openshift Container Platform Networking
=======================================

Openshift uses an software defined network (SDN) based on OpenVSwitch technology.

Each container gets it's own virtual interface which is a one to one mapping to an OpenVSwitch virtual interface.  Container eth0 is mapped to a virtual eth, veth on the host.

Is OpenVSwitch virtual interface is added to a OpenVSwitch bridge (i.e. br0) in order to share resources.

If traffic needs to get somewhere else on the OpenShift Cluster traffic is routed though an encrypted vxlan tunnel interface.  Every pod/container can use one or it can be isolated at the project level.

Then if we need to get out to the Internet it is routed through a tun0 (tunnel) interface.  Which is attached to the bridge.  And this allows traffic to get out through the default gateway.


Example commands:

To view the OpenVSwitch bridge interface:

```
[user@host ~]$ ovs-vsctl list-br
```

Shows all of the interfaces associated with the bridge.
```
[user@host ~]$ ovs-vsctl list-ifaces br0
```

Note: there is no direct/easy mapping of the OpenVSwitch virtual Ethernet Interfaces hashed names to OpenShift virtual interfaces.

To determine the mapping do the following:

Get a reference to the pod in question:

```
[user@host ~]$ oc get pods


myapp-1-sov3k

```

Execute the following command in the pod:

```
[user@host ~]$ oc exec myqpp-1-sov3k cat /sys/class/net/eth0/iflink

23
```

Login to the host where the container is running and then execute the following command:

```
ip a | grep veth

```

The output of the command above should display a veth entry which is proceeded with the number 23.


This is needed in order to tcpdump a container.



How to map a pid running on a host to a container:
==================================================

In order to get the pid used by the container execute the following command:

### Shows me all the containers running:
```
[user@host ~]$ docker ps | grep myqpp

cad75363b582
```

### Use docker inspect and extract the host pid.
```
[user@host ~]$ docker inspect -f '{{.State.Pid}}' cad75363b582

4218
```

### Show the process which the host pid has spawned.

```
[user@host ~]$ pstree -p 4218
```




