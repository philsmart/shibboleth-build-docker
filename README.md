# shibboleth-build-docker

A collection of stable environments for use in building and testing
[Shibboleth][] products. Each environment is represented by a
container image so that it can be embedded within a normal desktop
environment using [Docker][].

## Pre-built Environments

If your use case is the building of Shibboleth products or their Maven sites,
you can use one of the pre-built environments based on Amazon Corretto 11
(for Java 11 platform products) or Amazon Corretto 17 (for the Maven
sites of Java 11 platform products, or for Java 17 platform products).

I also provide a pre-built environment based on Amazon Corretto 8, although
this version is no longer used in Shibboleth builds.

These are multi-architecture images supporting both both `linux/amd64`
and `linux/arm64` hosts;the latter is used on Apple Silicon
based Macs.

To use these images, you make use of the various `runx` scripts included in this
repository; the corresponding pre-built environment for your host architecture
will be downloaded from Docker Hub.

Each of the `runx` scripts will start a new container based on the appropriate
image and provide a shell running within it. The `user` directory _in the
directory in which you invoke `runx`_ will be created if necessary and then
mounted into the container as the user's home directory. This will persist
across sessions.

It is normal to invoke the `runx` scripts from the root directory of this
repository; this means that the home directory will persist even across
sessions using _different_ container variants.

`runx` scripts exist in the `amazoncorretto-8`, `amazoncorretto-11` and `amazoncorretto-17`
directories and can be invoked as such:

```bash
amazoncorretto-11/runx
```

In addition, a `runx` alias is provided in the root directory for the most
common case of invoking the Corretto 11 environment.

You can pass arguments to a `runx` script and they will be passed on to the
`docker run` command as Docker options. For example, to run the container
without network access:

```bash
$ ./runx --network none
...
user@C11: ~ $ curl -I https://iay.org.uk
curl: (6) Could not resolve host: iay.org.uk
user@C11: ~ $
```

To pull the most up-to-date version of the container image before execution:

```bash
$ ./runx --pull always
amazon11: Pulling from ianayoung/shibboleth-build-docker
...
Status: Downloaded newer image for ianayoung/shibboleth-build-docker:amazon11
...
user@C11: ~ $
```

Running the `./copy-dotfiles` script will populate the shared home directory
with some useful files from your host environment:

* The Maven configuration from your `~/.m2/settings.xml` file. Other files
  (such as the `~/.m2/repository` directory) are not copied: the Maven
  repository inside the container is isolated from the repository on the host.

* Your `~/.gitconfig` file, if there is one.

* Your `~/.ssh/` directory, primarily so that you can `git clone` things from
  non-public repositories.

* Selected files from your GPG configuration in `~/.gnupg` are copied primarily
  so that you can sign `git tag` operations and Maven artifacts:
  * `secring.gpg` and `pubring.gpg` are copied if you have them. If you don't,
    see [GPG Keyring Format](#gpg-keyring-format) below.
  * `gpg.conf` for your main configuration.
  * Other files are _not_ copied, as things like agent configurations and
    agent sockets need to be different inside the container.

## Build Your Own Environments

If you are unable to use the pre-built environments for whatever reason,
you will need to perform a local build of the container image or images
you want to use. Note that images built in this way will be specific to
your Docker host's architecture.

Each directory in the repository represents a different build environment.
Each directory contains a `build` script to build the container image, and a
`run` script to start a new container based on that image and providing a shell
running within it.

You can pass arguments to the `build` script and they will be passed on to the
`docker build` command. For example, adding `--pull` would mean to build
against the latest base image rather than whatever is cached locally.

If you want to build _all_ of the variants at once, you can use the `build-all`
script in the top directory. If you pass arguments to this script they will
be forwarded to each variant's `build` script in turn.

The `run` script mounts the `user` directory from the repository as `/home/user`
within the container; from the container's prompt, this appears as the user's
home directory and persists across sessions. It is also common across sessions
using _different_ container variants.

You can pass arguments to the `run` script and they will be passed on to the
`docker run` command as Docker options. For example:

```bash
$ ./run --network none
...
user@C11: ~ $ curl -I https://iay.org.uk
curl: (6) Could not resolve host: iay.org.uk
user@C11: ~ $
```

Running the `./copy-dotfiles` script will populate the shared home directory
with some useful files from your host environment:

* The Maven configuration from your `~/.m2/settings.xml` file. Other files
  (such as the `~/.m2/repository` directory) are not copied: the Maven
  repository inside the container is isolated from the repository on the host.

* Your `~/.gitconfig` file, if there is one.

* Your `~/.ssh/` directory, primarily so that you can `git clone` things from
  non-public repositories.

* Selected files from your GPG configuration in `~/.gnupg` are copied primarily
  so that you can sign `git tag` operations and Maven artifacts:
  * `secring.gpg` and `pubring.gpg` are copied if you have them. If you don't,
    see [GPG Keyring Format](#gpg-keyring-format) below.
  * `gpg.conf` for your main configuration.
  * Other files are _not_ copied, as things like agent configurations and
    agent sockets need to be different inside the container.

## amazoncorretto-11

The `amazoncorretto-11` directory builds what should be regarded as the default
environment. It is based on the official Docker image for Amazon's Corretto 11
distribution of OpenJDK 11, which in turn is based on Amazon Linux 2, an
`rpm`-based distribution related to RHEL 7 and Centos 7.

Corretto 11 is the Java distribution specified as the "primary distribution" for
the Shibboleth Project's Java 11 platform, and the intention is to allow formal
production builds of any of the products based on that platform.

The image includes the following tools:

* maven (3.8.4)
* Git
* rpmbuild
* sudo
* wget
* which

To build the Docker image for this environment, do this:

```bash
(cd amazoncorretto-11; ./build)
```

The image will be tagged as `shibboleth-build-docker:amazon11`.

To execute the environment, type `amazoncorretto-11/run`, or just `./run`. This
will give you a `bash` prompt running under the home directory of user `user`
within a container. This home directory will also exist _outside_ the container
as a `user/` directory under the build location, and will retain state between
runs. Terminate the container and return to your original environment using
Ctrl+D.

If you want to start fresh, just exit the environment and then remove the
`user/` directory. It will be recreated on the next run.

If you're running on macOS, and there's an `ssh-agent` running in the host
session, it will be forwarded into the container so that you can use
things like `git clone` without further ado. Otherwise, a local `ssh-agent`
will be started inside the container, to which you can add identities from the
container's local `.ssh/` directory.

## amazoncorretto-17

This is the same as `amazoncorretto-11` but providing Corretto 17 instead of
Corretto 11. The expectation is that the next generation of Shibboleth Java
products will be built on this platform.

It is also used to perform Maven "site" builds for some projects for the Java
11 platform. These builds don't currently work under Java 11 distributions.

For large projects, you may find that building the site takes a very long time
inside a container. Refer to the [Performance](#performance) section below for
an explanation and some tuning suggestions.

## openjdk-7-centos-7

The `openjdk-7-centos-7` directory builds an environment intended to allow
formal production builds of products based on the Shibboleth Project's Java 7
platform.

The image is based on the [CentOS][] 7 userspace, with the following tools
available:

* maven (3.3.9)
* Subversion
* Git
* OpenJDK version 7
* rpmbuild
* wget
* which

The OpenJDK provided here is not quite the Java distribution specified as the
"primary distribution" for the Shibboleth Project's Java 7 platform; that's
Oracle's distribution.

The intention nevertheless is to allow close to formal
production builds of any of the products based on that platform.

To build the Docker image for this environment, do this:

```bash
(cd openjdk-7-centos-7; ./build)
```

The image will be tagged as `shibboleth-build-docker:ojdk7-c7`.

To execute the environment, type `openjdk-7-centos-7/run`. This will give you a `bash` prompt
running under the home directory of user `user` within a container. This home
directory will also exist _outside_ the container as a `user/` directory under
the build location, and will retain state between runs. Terminate the
container and return to your original environment using Ctrl+D.

If you want to start fresh, just exit the environment and then remove the
`user/` directory. It will be recreated on the next run.

If you have customisations in your outer environment, execute `./copy-dotfiles`
to copy them into the user directory. At the moment this handles:

* `~/.m2/settings.xml`
* `~/.gitconfig`
* `~/.ssh`

Copying `~/.ssh` is mostly intended to allow the use of `git` to a remote
repository, but note that you will need to type the passphrase each time as
no `ssh` agent is set up. This is not intended to be a development environment.

If you're running on macOS, and there's an `ssh-agent` running in the host
session, it will be forwarded into the container so that you can use
things like `git clone` without further ado. Otherwise, a local `ssh-agent`
will be started inside the container, to which you can add identities from the
container's local `.ssh/` directory.

## openjdk-8-centos-7

The `openjdk-8-centos-7` directory builds an image which will be tagged
as `shibboleth-build-docker:ojdk8-c7`. This is the same as the `ojdk7-c7` tag
except that OpenJDK 8 is provided instead of OpenJDK 7.

To build the Docker image for this environment, do this:

```bash
(cd openjdk-8-centos-7; ./build)
```

Run the environment using `openjdk-8-centos-7/run`.

This image is best used for testing under OpenJDK 8. It should not be used for
either product builds or site builds.

One large issue with the OpenJDK 8 environment is that the Cobertura coverage
tool does not work there.

## openjdk-\*-debian

These three directories build images from the official Docker library `java`
images, which are based on OpenJDK and Debian. These can be worth testing against
because OpenJDK in the Debian environment ships with an alternative elliptic
curve cryptographic provider.

To build these images:

```bash
(cd openjdk-7-debian; ./build)
(cd openjdk-8-debian; ./build)
```

The tags for these will be `ojdk7-deb` and `ojdk8-deb`.

## Troubleshooting

### GPG Keyring Format

The version of GPG used in most of the image variants built here is quite old,
because the containers use the version of GPG that is current for the underlying
operating system. In most cases, this is something like version 2.0.

Outside the container, you may well be using something from the 2.2 series: this
uses a new keychain format which is not supported by version 2.0. If you have
used GPG for a long time, you will have in your host's `~/.gnupg` both the old
format files (`secring.gpg` and `pubring.gpg`) and the new format ones. In this
case, the `./copy-dotfiles` script will copy in the `secring.gpg` and
`pubring.gpg` files and all will be well.

If you are a more recent user of GPG, or if you erased the old format files once
they had been converted, you will not be able to perform signature operations
inside the containers until you import your secret keys. To so this, perform the
following _outside_ the container:

```bash
gpg -a --export-secret-keys --output user/secret.asc
```

Now, perform the following _inside_ the container:

```bash
gpg --import secret.asc
```

### GPG Agent

If the location of this repository _on the host_ is too far from the root (if
the length of the repository's path is too long), you may run into issues
starting a `gpg-agent` process:

```text
gpg: can't connect to the agent: IPC connect call failed
```

You can try starting the agent manually to get more detail:

```bash
$ eval $(gpg-agent --daemon)
gpg-agent[29]: error binding socket to '/home/user/.gnupg/S.gpg-agent': File name too long
```

If you run into this issue, the solution seems to be to move the
`shibboleth-build-docker` repository itself closer to the root _on the host_ and
try again.

It's not clear exactly how long a path becomes a problem, but we have a couple
of data points.

* A value of `pwd | wc -c` of 50 or below seems to be acceptable.
* A value of `pwd | wc -c` of 90 or above seems to be problematic.

### Performance

If you are running these containers under Docker on a Linux system, you should
find that operations inside a container perform essentially indistinguishably to
running them on the host.

On the other hand, if you are running these containers using a technology such
as [Docker Desktop for Mac][], some operations will be significantly slower
because the containers are really running not on your host machine but on a
Linux virtual machine that is in turn running on your host machine. For
CPU-bound operations, the difference will likely not be noticeable in practice.

Operations that perform heavy I/O to the host filesystem (the `/home/user`
directory) may, however, be significantly slower because the host file system is
essentially remote to the container VM. We have found that the
`maven-javadoc-plugin` is particularly affected by this.

There are ways to [tune the performance of volume
mounts](https://docs.docker.com/docker-for-mac/osxfs-caching/) which we haven't
investigated yet. One fairly simple short-term solution should this problem
arise is to move the data from `/home/user` to the container's `/tmp` directory
temporarily. Because `/tmp` is local to the container, it exists within the
container VM rather than on the host file system and performance is much higher.
Of course one downside of this approach is that the `/tmp` directory does not
persist if you exit the shell and thus terminate the container; you should
ensure that the results of your operation are preserved either in `/home/user`
or otherwise before exiting.

[CentOS]: https://www.centos.org
[Docker]: https://www.docker.com
[Docker Desktop for Mac]: https://hub.docker.com/editions/community/docker-ce-desktop-mac
[Shibboleth]: https://shibboleth.net
