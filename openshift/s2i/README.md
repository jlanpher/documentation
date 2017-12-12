The Source-to-Image (S2I) Process
=================================

Source-to-Image (S2I) is a facility that makes it easy to build a container image from application source code. This facility takes an application's source code from a Git server, injects the source code into a base container based on the language and framework desired, and produces a new container image that runs the assembled application.

The following figure shows the resources created by the oc new-app command when the argument is an application source code repository. Notice that a Deployment Configuration and all its dependent resources are also created.


S2I is the major strategy used for building applications in OpenShift Container Platform. The main reasons for using source builds are:

User efficiency: Developers do not need to understand Dockerfiles and operating system commands such as yum install. They work using their standard programming language tools.

Patching: S2I allows for rebuilding all the applications consistently if a base image needs a patch due to a security issue. For example, if a security issue is found in a PHP base image, then updating this image with security patches updates all applications that use this image as a base.

Speed: With S2I, the assembly process can perform a large number of complex operations without creating a new layer at each step, resulting in faster builds.

Ecosystem: S2I encourages a shared ecosystem of images where base images and scripts can be customized and reused across multiple types of applications.


Building an Application with S2I and the CLI
============================================

Building an application with S2I can be accomplished using the OCP CLI.

An application can be created using the S2I process with the oc new-app command from the CLI.

```
oc new-app php~http://infrastructure.lab.example.com/app --name=myapp
```

1. The image stream used in the process appears to the left of the tilde(~).

2. The url after the tilde indicates the location of the source code's Git repository.

3. Sets the application name.

The oc new-app command allows for creating applications using source code from a local or remote Git repository. If only a source repository is specified, oc new-app tries to identify the correct image stream to use for building the application. In addition to application code, S2I can also identify and process Dockerfiles to create a new image.

The following example creates an application using the Git repository at the current directory.
``
oc new-app .
``

##Warning

When using a local Git repository, the repository must have a remote origin that points to a URL accessible by the OpenShift instance.


It is also possible to create an application using a remote Git repository and a context subdirectory:

```
oc new-app https://github.com/openshift/ruby-hello-world.git#beta4
```

If an image stream is not specified in the command, new-app attempts to determine which language builder to use based on the presence of certain files in the root of the repository:

| Language	 | Files                         |
| ---------- | ----------------------------- |
| ruby	     | Rakefile, Gemfile, config.ru  |
| Java EE	 | pom.xml                       |
| nodejs	 | app.json, package.json        |
| php	     | index.php, composer.json      |
| python	 | requirements.txt, config.py   |
| perl	     | index.pl, cpanfile            |

After a language is detected, new-app searches for image stream tags that have support for the detected language, or an image stream that matches the name of the detected language.


A JSON resource definition file can be created using the -o json parameter and output redirection:

```
oc -o json new-app php~http://infrastructure.lab.example.com/app --name=myapp > s2i.json
```


The build configuration (bc) is responsible for defining input parameters and triggers that are executed to transform the source code into a runnable image. The BuildConfig (BC) is the second resource and the following example provides an overview of the parameters that are used by OpenShift to create a runnable image.


```
...
{
            "kind": "BuildConfig", *1*
            "apiVersion": "v1",
            "metadata": {
                "name": "myapp", *2*
                "creationTimestamp": null,
                "labels": {
                    "app": "myapp"
                },
                "annotations": {
                    "openshift.io/generated-by": "OpenShiftNewApp"
                }
            },
            "spec": {
                "triggers": [
                    {
                        "type": "GitHub",
                        "github": {
                            "secret": "S5_4BZpPabM6KrIuPBvI"
                        }
                    },
                    {
                        "type": "Generic",
                        "generic": {
                            "secret": "3q8K8JNDoRzhjoz1KgMz"
                        }
                    },
                    {
                        "type": "ConfigChange"
                    },
                    {
                        "type": "ImageChange",
                        "imageChange": {}
                    }
                ],
                "source": {
                    "type": "Git",
                    "git": {
                        "uri": "http://infrastructure.lab.example.com/app" *3*
                    }
                },
                "strategy": {
                    "type": "Source", *4*
                    "sourceStrategy": {
                        "from": {
                            "kind": "ImageStreamTag",
                            "namespace": "openshift",
                            "name": "php:5.5" *5*
                        }
                    }
                },
                "output": {
                    "to": {
                        "kind": "ImageStreamTag",
                        "name": "myapp:latest" *6*
                    }
                },
                "resources": {},
                "postCommit": {},
                "nodeSelector": null
            },
            "status": {
                "lastVersion": 0
            }
        },

```

1. Define a resource type of *BuildConfig*.

2. Name the *BuildConfig* as myapp.

3. Define the address to the source code Git repository.

4. Define the strategy to use S2I.

5. Define the builder image as the php:latest image stream.

6. Name the output image stream as myapp:latest.


The deployment configuration is responsible for customizing the deployment process into OpenShift. They may include parameters and triggers that are necessary to create new container instances, and are translated into a replication controller from Kubernetes. Some of the features provided by DeploymentConfigs are:

User customizable strategies to transition from the existing deployments to new deployments.

Rollbacks to a previous deployments.

Manual replication scaling.


```
...
{
            "kind": "DeploymentConfig", *1*
            "apiVersion": "v1",
            "metadata": {
                "name": "myapp", *2*
                "creationTimestamp": null,
                "labels": {
                    "app": "myapp"
                },
                "annotations": {
                    "openshift.io/generated-by": "OpenShiftNewApp"
                }
            },
            "spec": {
                "strategy": {
                    "resources": {}
                },
                "triggers": [
                    {
                        "type": "ConfigChange" *3*
                    },
                    {
                        "type": "ImageChange", *4*
                        "imageChangeParams": {
                            "automatic": true,
                            "containerNames": [
                                "myapp"
                            ],
                            "from": {
                                "kind": "ImageStreamTag",
                                "name": "myapp:latest"
                            }
                        }
                    }
                ],
                "replicas": 1,
                "test": false,
                "selector": {
                    "app": "myapp",
                    "deploymentconfig": "myapp"
                },
                "template": {
                    "metadata": {
                        "creationTimestamp": null,
                        "labels": {
                            "app": "myapp",
                            "deploymentconfig": "myapp"
                        },
                        "annotations": {
                            "openshift.io/container.myapp.image.entrypoint": 
"[\"container-entrypoint\",\"/bin/sh\",\"-c\",\"$STI_SCRIPTS_PATH/usage\"]",
                            "openshift.io/generated-by": "OpenShiftNewApp"
                        }
                    },
                    "spec": {
                        "containers": [
                            {
                                "name": "myapp",
                                "image": "myapp:latest", *5*
                                "ports": [ *6*
                                    {
                                        "containerPort": 8080,
                                        "protocol": "TCP"
                                    }
                                ],
                                "resources": {}
                            }
                        ]
                    }
                }
            },
            "status": {}
        },
...

```

1. Define a resource type of *DeploymentConfig*.
2. Name the *DeploymentConfig* as myapp.
3. A configuration change trigger causes a new deployment to be created any time the replication controller template changes.
4. An image change trigger causes a new deployment to be created each time a new version of the *myapp:latest* image is available in the repository.
5. Define that the *library/myapp:latest* container image should be deployed.
6. Specifies the container ports.


Build Process
=============

After creating a new application, the build process starts. See a list of application builds with oc get builds:

```
$ oc get builds
NAME             TYPE      FROM          STATUS     STARTED      DURATION
myapp-1          Source    Git@59d3066   Complete   3 days ago   2m13s
```

OpenShift allows you to viewing the build logs. The following command shows the last few lines of the build log:

```
oc logs build/myapp-1
```

Trigger a new builder with the oc start-build build_config_name command:

```
$ oc start-build myapp
build "myapp-2" started
```


Relationship between BuildConfig and DeploymentConfig
=====================================================

The BuildConfig pod is responsible for creating the images in OpenShift and pushing them to the internal Docker registry. Any source code or content update normally requires a new build to guarantee the image is updated.

The DeploymentConfig pod is responsible for deploying pods into OpenShift. The outcome from a DeploymentConfig pod execution is the creation of pods with the images deployed in the internal docker registry. Any existing running pod may be destroyed, depending on how the DeploymentConfig is set.

The BuildConfig and DeploymentConfig resources do not interact directly. The BuildConfig creates or updates a container image. The DeploymentConfig reacts to this new image or updated image event and creates pods from the container image.

