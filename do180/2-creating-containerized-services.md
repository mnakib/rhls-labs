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

