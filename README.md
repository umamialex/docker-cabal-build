# docker-cabal-build
This is a base image for building [Haskell's][haskell] [Cabal][cabal] (and 
pretty much the rest of the [Haskell Platform][haskell-platform].

This image is heavily influenced by [nubs/npm-build][nubs].

## Purpose
This docker image builds on top of Arch Linux's base/archlinux image for the
purpose of building projects using [Cabal][cabal].  It provides several key features:

* A non-root user (`build`) for executing the image build.  This is important
  for security purposes and to ensure that the package doesn't require root
  permissions to be built.
* Access to the build location will be in the volume located at `/code`.  This
  directory will be the default working directory.
* The [Cabal][cabal] bin directory is automatically included in `PATH` using the
  `/home/build/.cabal/bin` directory.

## Usage
This library is useful with simple `.cabal`s from the command line.
For example:

```bash
docker run --interactive --tty --rm --volume /tmp/my-code:/code suitupalex/cabal-build

# Using short-options:
docker run -i -t --rm -v /tmp/my-code:/code suitupalex/cabal-build
```

This will execute the default command (`cabal install -j`).

Other commands can also be executed.  For example, to update dependencies:

```bash
docker run -i -t --rm -v /tmp/my-code:/code suitupalex/cabal-build cabal update
```

## Permissions
This image uses a build user to run [Cabal][cabal]. This means that your file permissions
must allow this user to write to certain folders inside your project directory. The
easiest way to do this is to create a group and give that group write access to
the necessary folders.

```bash
# To give permissions to the entire project directory, do:
groupadd --gid 11235 cabal-build
chmod -R g+w .
chgrp -R cabal-build .
```

You may also want to give your user access to files created by the build user.

```bash
usermod -a -G 11235 "$(whoami)"
```

### Dockerfile build
Alternatively, you can create your own `Dockerfile` that builds on top of this
image.  This allows you to modify the environment by installing additional
software needed, altering the commands to run, etc.

A simple one that just installs another package but leaves the rest of the
process alone could look like this:

```dockerfile
FROM suitupalex/cabal-build

USER root

RUN pacman --sync --noconfirm --noprogressbar --quiet somepackage

USER build
```

You can then build this docker image and run it against your `.cabal`
volume like normal (this example assumes the `.cabal` and `Dockerfile` are
in your current directory):

```bash
docker build --tag my-code .
docker run -i -t --rm -v "$(pwd):/code" my-code
docker run -i -t --rm -v "$(pwd):/code" my-code cabal update
```

[haskell]: https://haskell.org
[cabal]: https://haskell.org/haskellwiki/Cabal
[haskell-platform]: https://haskell.org/platform
[nubs]: https://registry.hub.docker.com/u/nubs/npm-build/
