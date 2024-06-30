+++
title = 'Containerized development environment'
date = 2022-08-21T14:49:25Z
draft = false
lang = 'en-US'
tags = ['development environment', 'containerized environment', 'docker']
+++

A few days ago I was asked by a team member about which Go version to install to work on the project. Then it hit me that I no longer cared about setting up my development environments, almost. Most of my development environments are replicable, disposable, and recreate-able within seconds.

---

A few years back, when I was working on various projects, I realized that dealing with various versions of various tech stacks was a real pain. And after embracing Docker for containerized deployment, I wondered if we can have something similar for our development environments.

Imagine something like this: we can create the definition of development environment for every project. Through this specification, we can specify the tools and versions required by that project. And when I open a project, something will set up its environment isolated from other projects and the host system.

If we can have something like that, it will be very helpful when moving between computers, and for sharing the environments with other developers working on the project. Gone are the hassles of setting up environments.

---

Fortunately, that technology is already existed from a few years back, in form of something called VSCode Dev Container. It’s working exactly as described above.

## How it works

[VSCode Dev Container](https://code.visualstudio.com/docs/devcontainers/containers) utilizes Docker to create a containerized / isolated environment for every project. It uses a JSON file (`devcontainer.json`) to define the environment for each project.

We start by defining the base image (a Docker image), usually the base tech of the stack. We can pick one of various base techs available. Popular techs are officially provided including C++, Go, Python, NodeJS, Rust and more.

![image](images/01-pick-base.png#center)

We can then add additional features to be included in the environment, including combining multiple techs, e.g., Go, NodeJS, and MongoDB. In fact, we can add everything if it's needed by the project.

![image](images/02-select-features.png#center)

If we require something that can't be provided through JSON, we can use our own Dockerfile to specify our custom image.

The extension will prepare a remote environment based on the specification and connect the IDE with it. What we edit in VSCode are files located in the host OS, not in the container.

For the actual guide, please visit the [official documentation](https://code.visualstudio.com/docs/devcontainers/tutorial).

---

## Advantages

- Perfectly replicable development environments (almost)
  - including the tools, other techs, and extensions
  - making them easy to share
- Isolated from host and from other projects’ environment
  - no need to dirty the host system
  - no need to update host's environment variables
  - no need to hunt for files and environment variables when done with an environment
- Automatic
  - no need to follow some tedious steps to install tools or extensions
  - if a tool is needed, just add it into the definition and it will automatically be installed when the environment is rebuilt

## Disadvantages

- Requires Docker, VSCode and [Dev Container extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers). It might be supported by other IDEs.
- It takes more disk space as there will be copies of compilers, runtimes and other tools across environments. But this can be offseted by destroying unused environments (easy and quick!).
- Larger memory usage
- Sharing resources with the host machine or other virtualized / simulated environments usually require extra steps. But the upside, we will notice it well when a resource sharing is required

---

## How I use Dev Container

Dev Container have tremendously helped me on working with various projects with different stacks.

All my projects, except mobile app projects, are dev-containerized. I haven’t installed Go tools, Python interpreter, NodeJS runtime, Rust tooling, etc. directly on my computer's OS. My computer's OS is relatively clean of dev tools. Well, except Dart / Flutter because I needed to install it in a non-containerized manner as it’s required for mobile dev.

I’ve used dev environments to learn and experiment with various techs. When I want to learn a new tech, I just need to create a folder and then create a Dev Container for it. When I am done, cleaning it up is a breeze.

Even when I am checking out other's project, I’d open it in a dev container even when the project is not originially dev-containerized.
