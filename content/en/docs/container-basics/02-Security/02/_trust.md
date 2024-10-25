---
title: "Docker Trust"
weight: 22
sectionnumber: 2.2
---

This lab focuses on understanding and securing image distribution. We'll start with a simple `docker pull` and build up to using Docker Content Trust (DCT).

Let us start with a common pull

 ```bash
docker pull alpine:edge
```

This command will pull the Alpine image tagged as `edge`. The corresponding image can be found [here on Docker Store](https://store.docker.com/images/alpine).

If no tag is specified, Docker will pull the image with the `latest` tag.

```
$ docker pull alpine:edge

edge: Pulling from library/alpine
e587fa4f6e1f: Pull complete
Digest: sha256:e5ab6f0941eb01c41595d35856f16215021a941e9893501d632ed4c0ee4e53a6
Status: Downloaded newer image for alpine:edge
```

Now run a new container from the image and ping songlaa.com

```bash
docker run --rm -it alpine:edge ping songlaa.com
```

Pulling by tag is easy and convenient. However, tags are mutable, and the same tag can refer to different images over time. For example, you can add updates to an image and push the updated image using the same tag as a previous version of the image. This scenario where a single tag points to multiple versions of an image can lead to bugs and vulnerabilities in your production environments.

This is why pulling by digest is such a powerful operation. Thanks to the content-addressable storage model used by Docker images, we can target pulls to specific image contents by pulling by digest. In this step you'll see how to pull by digest.

Pull the Alpine image with the `sha256:b7233dafbed64e3738630b69382a8b231726aa1014ccaabc1947c5308a8910a7` digest.

```bash
docker pull alpine@sha256:b7233dafbed64e3738630b69382a8b231726aa1014ccaabc1947c5308a8910a7
```

It's not easy to find the digest of a particular image tag. This is because it is computed from the hash of the image contents and stored in the image manifest. The image manifest is then stored in the Registry. This is why we needed a `docker pull` by tag to find digests previously. It would also be desirable to have additional security guarantees such as image freshness.

Enter Docker Content Trust: a system currently in the Docker Engine that verifies the publisher of images without sacrificing usability. Docker Content Trust implements [The Update Framework](https://theupdateframework.github.io/) (TUF), an NSF-funded research project succeeding Thandy of the Tor project. TUF uses a key hierarchy to ensure recoverable key compromise and robust freshness guarantees.

Under the hood, Docker Content Trust handles name resolution from IMAGE tags to IMAGE digests by signing its own metadata -- when Content Trust is enabled, docker will verify the signatures and expiration dates in the metadata before rewriting a pull by tag command to a pull by digest.

In this step you will enable Docker Content Trust and pull signed and unsigned images.

Enable Docker Content Trust by setting the DOCKER_CONTENT_TRUST environment variable.

```bash
export DOCKER_CONTENT_TRUST=1
```

All Docker commands remain the same. Docker Content Trust will work silently in the background.
Pull the `alpine` signed image.

```bash
docker pull alpine
```

That works because the image is trusted.

Now try to pull an untrusted image

```bash
docker pull grafgabriel/alpine
```

It will fail with a similiar error (also please never use this image, it is really old)

```
Error: remote trust data does not exist for docker.io/grafgabriel/alpine: notary.docker.io does not have trust data for docker.io/grafgabriel/alpine
```

If you have Content Trust enabled you can only download trusted images. If you try to push images with content trust enabled Docker will ask you to create a key to sign your images and it will then sign your image before uploading it.

Docker Content Trust is powered by [Notary](https://github.com/docker/notary), an open-source TUF-client and server that can operate over arbitrary trusted collections of data. Notary has its own CLI with robust features such as the ability to rotate keys and remove trust data.

Disable Content Trust again as we will need to download unsigned images in our lab:

```bash
export DOCKER_CONTENT_TRUST=0
```

For more information about Docker Content Trust, see [the documentation](https://docs.docker.com/engine/security/trust/).

## Official images

All images in Docker Hub under the `library` organization (currently viewable at: <https://hub.docker.com/explore/>)
are deemed "Official Images."  These images undergo a rigorous, [open-source](https://github.com/docker-library/official-images/)
review process to ensure they follow best practices. These best practices include signing, being lean, and having clearly written Dockerfiles. For these reasons, it is strongly recommended that you use official images whenever possible.

Official images can be pulled with just their name and tag. You do not have to precede the image name with `library/` or any other repository name.
