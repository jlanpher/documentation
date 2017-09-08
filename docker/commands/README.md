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
