Chapter 5. Creating Custom Container Images

- [Designing Custom Container Images](https://github.com/mnakib/rhls/blob/main/do180/ch-5-creating-custom-container-images/section-5-3-building-custom-container-images-with-containerfile.md#designing-custom-container-images)
- [Building Custom Container Images with Containerfiles](https://github.com/mnakib/rhls/blob/main/do180/ch-5-creating-custom-container-images/section-5-3-building-custom-container-images-with-containerfile.md#building-custom-container-images-with-containerfiles)
- [Guided Exercise: Creating a Basic Apache Container Image](https://github.com/mnakib/rhls/blob/main/do180/ch-5-creating-custom-container-images/section-5-3-building-custom-container-images-with-containerfile.md#guided-exercise-creating-a-basic-apache-container-image)

# Designing Custom Container Images

One way to creating container images is to commit a running container. However, this method doesn't follow best practices, like maintainability, automation of building, and repeatability.



# Building Custom Container Images with Containerfiles

## Building Base Containers

A `Containerfile` is a mechanism to automate the building of container images. Building an image from a Containerfile is a three-step process.

1. Creating a working directory
2. Writing the Containerfile
3. Building the image with Podman


### Creating a working directory

The working directory is the directory containing all files needed to build the image.

Creating an empty working directory is good practice to avoid incorporating unnecessary files into the image.


### Writing the Containerfile

The file name must be `Containerfile` or `Dockerfile` and contains (case-sensitive) instructions to build the custom image.

Instructions starting with a hash (#) are comments.

```
# This is a comment line
FROM ubi8/ubi:8.3 
LABEL description="This is a custom httpd container image" 
MAINTAINER Mourad NAKIB <mnakib@redhat.com> 
RUN yum install -y httpd less iputils telnet && \
    yum clean all
EXPOSE 80 
ENV LogLevel "info" 
ADD http://linkToPDFFile.pdf /var/www/html 
COPY ./src/ /var/www/html/ 
USER apache 
ENTRYPOINT ["/usr/sbin/httpd"] 
CMD ["-D", "FOREGROUND"]  
```

> An example of a link to a pdf file is 
> https://www.vmware.com/content/dam/digitalmarketing/vmware/en/pdf/techpaper/performance/pvrdma-hpc-setup-perf.pdf


#### CMD and ENTRYPOINT

The Containerfile should contain at most one `ENTRYPOINT` and one `CMD` instruction. If both are present, the **last** instruction takes **precedence**.

`ENTRYPOINT` **cannot** be overwritten. `CMD` **can** be overwritten.

Examples below:

`ENTRYPOINT ["/bin/date", "+%H:%M"]`

`ENTRYPOINT ["/bin/date"]`  
`CMD ["+%H:%M"]`

In both cases, when a container starts without providing a parameter, the current time is displayed:

`$ sudo podman run -it ubuntu`
```
11:41
```

In below case, the `CMD` can be overwritten.

`ENTRYPOINT ["/bin/date"]`  
`CMD ["+%H:%M"]`

`$ sudo podman run -it ubuntu +%A`
```
Tuesday
```



#### ADD and COPY

`ADD` can copy local & remote files, and can also untar archives.

`COPY` can copy only local files.



### Building the image with Podman

The `podman build` command processes the Containerfile and builds a new image based on the instructions it contains

`$ podman build -t NAME:TAG DIR`

> **DIR** is the path to the working directory


`$ podman build -t httpd-custom:v0.1 .`




# Guided Exercise: Creating a Basic Apache Container Image

0. Prepare the lab

`$ lab dockerfile-create start`
  


1. Create the Apache Containerfile

`$ vim ~/DO180/labs/dockerfile-create/Containerfile`

```
FROM ubi8/ubi:8.5

MAINTAINER Your Name <youremail>

LABEL description="A custom Apache container based on UBI 8"

RUN yum install -y httpd && \
    yum clean all

RUN echo "Hello from Containerfile" > /var/www/html/index.html

EXPOSE 80

CMD ["httpd", "-D", "FOREGROUND"]
```


2. Build and verify the Apache container image

`$ cd ~/DO180/labs/dockerfile-create`

`$ podman build --layers=false -t do180/apache`

> The `--layers=false` is instruct Podman to delete the anonymous intermediate layers.

`$ podman images`

```
REPOSITORY                           TAG     IMAGE ID      CREATED         SIZE
localhost/do180/apache               latest  16c8493a19ad  45 seconds ago  257 MB
...
```


3. Run the Apache container.

`$ podman run --name lab-apache -d -p 10080:80 do180/apache`

`$ podman ps`

`$ curl -s 127.0.0.1:10080`

















