# Provisioning Containerized Services

## Fetching Container Images with Podman

Applications can run inside containers, providing an isolated and controlled execution environment.

Container images are available from image registries that allow users to search and retrieve container images.

`$ podman search rhel`
```
INDEX      NAME                            DESCRIPTION  STARS OFFICIAL AUTOMATED
redhat.com registry.access.redhat.com/rhel This plat... 0
...
```


After the image is found, you may **pull** it.

`$ podman pull rhel`
```
Trying to pull registry.access.redhat.com/rhel...
Getting image source signatures
Copying blob sha256:
...
```


Container image syntax are named based on below syntax:

> **registry-name/user-name/image-name:tag**



To display **locally stored images**

`$ podman images`
```
REPOSITORY                        TAG      IMAGE ID       CREATED       SIZE
registry.access.redhat.com/rhel   latest   699d44bc6ea2   4 days ago    214MB
...
```

## Running Containers

The `podman run` runs a container locally based on an image. At a **minimum**, the command requires the **name of the image** to execute in the container.




