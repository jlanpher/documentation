Image Streams
=============

OpenShift deploys new versions of user applications into pods quickly. To create a new application, in addition to the application source code, a base image (the S2I builder image) is required. If either of these two components gets updated, a new container image is created. Pods created using the older container image are replaced by pods using the new image.

While it is obvious that the container image needs to be updated when application code changes, it may not be obvious that the deployed pods also need to be updated should the builder image change.

The image stream resource is a configuration that names specific container images associated with image stream tags, an alias for these container images. An application is built against an image stream. The OpenShift installer populates several image streams by default during installation. To check available image streams, use the oc get command, as follows:

```
$ oc get is -n openshift
NAME                                DOCKER REPO                               TAGS                                                      
...
jenkins                             172.30.0.103:5000/openshift/jenkins       2,1                                                                
mariadb                             172.30.0.103:5000/openshift/mariadb       10.1                                                                    
mongodb                             172.30.0.103:5000/openshift/mongodb       3.2,2.6,2.4                                                           
mysql                               172.30.0.103:5000/openshift/mysql         5.5,5.6                                             
nodejs                              172.30.0.103:5000/openshift/nodejs        0.10,4                                             
perl                                172.30.0.103:5000/openshift/perl          5.20,5.16                                                               
php                                 172.30.0.103:5000/openshift/php           5.5,5.6                                             
postgresql                          172.30.0.103:5000/openshift/postgresql    9.5,9.4,9.2                                                            
python                              172.30.0.103:5000/openshift/python        3.5,3.4,3.3 + 1 more...                                                                                                 
ruby                                172.30.0.103:5000/openshift/ruby          2.3,2.2,2.0  
```

OpenShift has the ability to detect when an image stream changes and to take action based on that change. If a security issue is found in the nodejs-010-rhel7 image, it can be updated in the image repository and OpenShift can automatically trigger a new build of the application code.