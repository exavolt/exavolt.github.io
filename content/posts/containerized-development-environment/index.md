+++
title = 'Containerized development environment'
date = 2022-08-21T14:49:25Z
draft = false
lang = 'en-US'
+++

A few days ago I was asked by a team member about which Go version to install to work on the project. Then it came to my mind that I no longer care about setting up my development environments. My development environments are disposable and recreate-able within seconds, or minutes, well, depending on the internet connection.

---

A few years back, when I was working on various projects, I felt that maintaining various versions of various tech stacks is a real pain. And after embracing Docker for containerized deployment, I wondered if we can have something similar for our development environments.

Imagine something like this: we can create the definition of development environment for every project. Through this specification, we can specify the tools and versions required by that project, which might be different from one to another. And when I open a project, something will set up its environment isolated from other projects and the host system.

If we can have something like that, it will be very helpful when moving between computers, and for sharing the environments with other developers working on the project. Gone are the hassles of setting up environments.

Sure, environment managers provided by various languages, like `pyenv` `nvm` `gvm`, do help. But they missed one or more of the following requirements: 

- Language agnostic with possibility of combining various languages
- Complete environment rather than just language-specific tools
- Machine-readable environment specification so that we don't need to specify it in the README

---

Fortunately, that technology already existed from a few years back, in form of something called VSCode Dev Container. It’s working exactly as described above.

[VSCode Dev Container](https://code.visualstudio.com/docs/devcontainers/containers) utilizes Docker to create a containerized / isolated environment for every project. We define the environment we want as a Dockerfile and through a ~~YAML~~ JSON file.

---

## Advantages

- Perfectly replicable development environment (almost) — making them easy to share
- Isolated from host and from other projects’ environment — not affecting each-other by default
- Self-contained including tools and VSCode extensions
    - Tools’ version are pinned
    - Extensions are nicely set up by default on official base containers
- Destroying an environment is easy, switching between runtime versions, including to experimental versions, is easy. Recreating is easy.
- Great for experimenting with development with various techs
- Reduce cognitive loads and setting up hassles — the definition is the documentation

## Disadvantages

- Requires Docker, VSCode and [Dev Container extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers). It might be supported by other IDEs.
- It takes more disk usage as there will be copies of compilers, runtimes and other tools across environments. But this can be offseted by destroying unused environments.
- Larger memory usage
- Sharing resources with the host machine or other virtualized / simulated environments usually require extra steps. But the upside, we will notice it well when a resource sharing is required

---

Dev Container have tremendously helped me on working with various projects with different stacks. When I need to work on a new project, or experiment on new techs, I just need to create a folder and then create a container for it based on common base images.
