Saving and Loading Images
=========================

An existing image from the Docker cache can be saved to a tar file using the docker save command.  The generated file is not just a regular tar file; it contains image metadata and preserves the original image layers, so the original image can be later re-created exactly as it was.

The general syntax of the docker command save verb is:

```
docker save [-o FILE_NAME] IMAGE_NAME[:TAG]
```

if the -o option is not used the generated image is ent to the standard output as binary data.

In the following example, the MySQL container image from RedHat Software Collections is aved to the file mysql.tar:

```
docker save -o mysql.tar registry.access.redhat.com/rhscl/mysql-56-rhel7
```

A tar file generated using the save verb can be used for backup purposes.  To restore the image, use the docker load command.  The general syntax of the command ia as follows:

```
docker load [-i FILE_NAME]
```

If the tar file given as an argument is not a container image with metadata, the docker load command will fail.

Following the previous docker save example, the image may be restored to the Docker cache using the following command:

```
docker load -i mysql.tar
```

Publishing an Image to a Registry
=================================
To push an image to the registry, the image must be stored in docker's cache, and the image should be tagged for identification purposes.  To tag an image, use the tag sub-command:

```
docker tag IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
```

For example, to tag the Nginx image with the latest tag, you would execute the following command:

```
docker tag nginx nginx
```

To push the image to the registry, you would execute the following command:

```
docker push nginx
```

Deleting Images
===============

Any image downloaded to the local Docker cache is kept there even if no containers are using it.  However, images can be come outdated, and should be subsequently replaced.

To delete an image from the local Docker cache, use the docker rmi command.  The syntax for this command is as follows:

```
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

The major option available for the rmi sub-command is --force=true to force the removal of an image.  An image can be referenced using its name or its ID for removal purposes.

The same image can be shared among multiple tags, and to use the rmi sub-command with the image ID will fail.  To avoid a tag-by-tag removal for an image, the simplest approach is to use the --force=true option.

Any container using the image will block any attempt to delete an image.  All the containers using that image must be stopped and removed before the image can be deleted.

#### Deleting all Images ####

To delete all images that are not used by any container, use the following command:

```
docker rmi $(docker images -q)
```

This returns all image IDs available in the local Docker cache and passes them as a parameter to the docker rmi command for removal.  Images that are in use will not be deleted, but this does not prevent any unused images from being removed.


Modifying Images
================

Ideally, all container images should be built using a Dockerfile to create a clean and slim set of image layers, without log files, temporary files, or other artifacts created by the container customization.  Despite these recommendations, some container images may be provided as they are, without any Dockerfile available.  As an alternative approach to creating new images, a running container can be changed in place and its layers saved to create a new container image.  This feature is provided by the docker commit command.

The syntax for the docker commit command is as follows:

```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

The following shows the important options available for the docker commit command:

| Option       | Description                                                         |
| ------------ | ------------------------------------------------------------------- |
| --author=""  | Identifies the author responsible for the container image creation. |
| --message="" | Includes a commit message to the registry.                          |

To identify a running container in docker, execute the docker ps command:

```
docker ps
```

To identify which files were changed, created, or deleted since the container was started, you can issue the docker command docker diff.  The diff sub-command only requires the container name or container ID:

```
docker diff mysql-basic
```

Any added/new file(s) are marked with an "A" and any changed file(s) are marked with a "C".

To commit the changes to another image, execute the following command:

```
docker commit mysql-basic mysql-custom
```

Tagging Images
==============

Tags are used by container developers to distinguish between multiple versions of the same software, such as the one observed for MySQL container image.

Multiple tags are provided to easily identify a release.  On the offical MySQL container image website, the version is used as the tag's name (e.g. 5.5.16).  In addition the same image has a second tag with the minor version (e.g. 5.5) to minimize the need to get the latest release for a certain version.

To tag an image, use the docker tag command:

```
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
```

The "IMAGE" argument is the image name with an optional tag that was locally stored to the docker daemon.  The following argument refers to alternative names for the image that are stored locally.  If no tag is provided, the "latest" tag will be used by default.  For example, to tag an image, the following command may be used:

```
docker tag mysql-custom devops/mysql
```

The mysql-custom option is the image name that is stored in the docker daemon cache.

To use a different tag name, use the following command instead:

```
docker tag mysql-custom devops/mysql:snapshot
```

#### Removing Tags from the Image ####

Tags can be removed using the docker rmi command mentioned previously.  Therefore, to delete a specific tag from the docker daemon, execute the following command:

```
docker rmi devops/mysql:snapshot
```

Because multiple tags can point to teh same image, to remove an image referred to by multiple tags, each tag should be individually removed first.  Alternatively, use the docker rmi --force command.


Tagging Practices
=================

Normally, the "latest" tag is automatically added by docker if nothing is provided, because it is considered to be teh image's latest build.  However, this may not be true depending on how the tags are used.  For example, most open source projects consider "latest" as the most recent release not the latest build.

Moreover, multiple tags are provided to minimize the need to recall the latest release of a certain version of a project.  Thus, if there is a project version release (e.g. 2.1.10), another tag called 2.1 can be created and pointed to the same image from the 2.1.10 release to simplify how the image is pulled from the registry.



General
=======

To access an already running container execute the following command:

```
docker exec -it CONTAINER_NAME bash
```

LABEL is responsible for adding generic metadata to an image. A LABEL is a simple key/value pair.


MAINTAINER is responsible for setting the Author field of the generated container image. You can use the docker inspect command to view image metadata.


RUN executes commands in a new layer on top of the current image, then commits the results. The committed result is used in the next step in the Dockerfile. The shell that is used to execute commands is /bin/sh.


EXPOSE indicates that the container listens on the specified network ports at runtime. The Docker containerized environment uses this information to interconnect containers using the linked containers feature. The EXPOSE instruction is only metadata; it does not make ports accessible from the host. The -p option in the docker run command exposes a port from the host and the port does not need to be listed in an EXPOSE instruction.

ENV is responsible for defining environment variables that will be available to the container. You can declare multiple ENV instructions within the Dockerfile. You can use the env command inside the container to view each of the environment variables.


ADD copies new files, directories, or remote URLs and adds them to the container file system.


COPY also copies new files and directories and adds them to the container file system. However, it is not possible to use a URL.


USER specifies the username or the UID to use when running the container image for the RUN, CMD, and ENTRYPOINT instructions in the Dockerfile. It is a good practice to define a different user other than root for security reasons.

ENTRYPOINT specifies the default command to execute when the container is created. By default, the command that is executed is /bin/sh -c unless an ENTRYPOINT is specified.

CMD provides the default arguments for the ENTRYPOINT instruction.


The ENTRYPOINT and CMD instructions have two formats:

* Using a JSON Array:

```
ENTRYPOINT ["command", "param1", "param2"]


CMD ["param1", "param2"]
```

This is the preferred form.


* Using a shell form:
```
ENTRYPOINT param1 param2

CMD param1 param2
```

The Dockerfile should contain at most one ENTRYPOINT and one CMD instruction. If more than one of each is written, then only the last instruction takes effect. Because the default ENTRYPOINT is /bin/sh -c, a CMD can be passed in without specifying an ENTRYPOINT.

The CMD instruction can be overridden when starting a container. For example, the following instruction causes any container that is run to ping the local host:

```
ENTRYPOINT ["/bin/ping", "localhost"]
```

The following example provides the same functionality, with the added benefit of being overwritable when a container is started:

```
ENTRYPOINT ["/bin/ping"]

CMD ["localhost"]
```
When a container is started without providing a parameter, localhost is pinged:

```
docker run -it test/my-test
```

if a parameter is provided after the image name in the docker run command, however, it overwrites the CMD instruction. For example, the following command will ping google.com instead of localhost:

```
docker run -it test/my-test google.com
```

As previously mentioned, because the default ENTRYPOINT is /bin/sh -c, the following instruction also pings localhost, without the added benefit of being able to override the parameter at run time.

```
CMD ["/bin/ping", "localhost"]
```


The ADD and COPY instructions have two forms:

* Using Shell form:

```
ADD <source>... <destination>

COPY <source>... <destination>
```

* Using JSON array:

```
ADD ["<source>...", ["<destination>"]

COPY ["<source>...", ["<destination>"]
```

The source path must be inside the same folder as the Dockerfile. The reason for this is that the first step of a docker build command is to send all files from the Dockerfile folder to the docker daemon, and the docker daemon cannot see folders or files that are in another folder.

Image Layering

Each instruction in a Dockerfile creates a new layer. Having too many instructions in a Dockerfile causes too many layers, resulting in large images.

```
RUN yum --disablerepo=* --enablerepo="rhel-7-server-rpms" && \
yum update && \
yum install -y httpd
```

The example creates only one layer and the readability is not compromised. This layering concept also applies to instructions such as ENV and LABEL. To specify multiple LABEL or ENV instructions, Red Hat recommends that you use only one instruction per line, and separate each key-value pair with an equals sign (=):

```
LABEL version="2.0" \
description="This is an example container image" \
creationDate="01-09-2017"


ENV MYSQL_ROOT_PASSWORD="my_password" \
MYSQL_DATABASE "my_database"

```

Building Images with the docker command
=======================================

The docker build command processes the Dockerfile and builds a new image based on the instructions it contains.  The syntax for this command is as follows:

```
docker build -t NAME:TAG DIR
```

DIR is the path to the working directory.  It can be the current directory as designated by a period(.) if the working directory is the current directory of the shell.  NAME:TAG is a name with a tag that is assigned to the new image.  It is specified with the -t option.  If the TAG is not specified, then the image is automatically tagged as latest.
