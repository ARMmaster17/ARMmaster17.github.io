---
layout: post
title:  "Introduction to GitOps with Flux v2"
date:   2020-05-13 12:00:00 -0500
categories: gitops devops cd ci flux github kubernetes docker
---

## Overview
When I first learned about GitOps, I really wanted to try it out on my homelab. I initially went with [Argo CD](https://argoproj.github.io/cd/), simply because that's what everyone else was using. Although Argo CD is cool and all, there was one major deal-breaker for me. Argo CD requires nearly 8GB of RAM for normal operation and a lot of CPU power, which was more than double the amount of RAM in my test k3s cluster. As a broke college student I figured there had to be a better (and cheaper) way. The solution I found was [Flux CD](https://github.com/fluxcd/flux). It's very lightweight, and all of it's configuration is stored in your source control, so no need to mess with any fancy PV setups in k8s.

Flux works on top of any existing k3s or k8s cluster. I'll be using k3s for this tutorial. You'll need about 256MB of RAM and a few milli-CPU available in your cluster. One important thing that I found is that the Flux documentation is a little confusing at times, and I often had to delete the Flux deployment and re-deploy to implement the change. This guide will attempt to get it right the first time (assuming your needs are the same as mine).

## Setup
This tutorial assumes that you already have a kubernetes cluster of some kind already up and running. If you do not have a cluster, there are several options available to you. [Minikube](https://minikube.sigs.k8s.io/docs/) is an excellent choice if you only have one (somewhat-powerful) machine. [k3sup](https://github.com/alexellis/k3sup) is a good tool if you have a few spare computers and want to quickly turn them into Kubernetes nodes. And finally there are always hosted options such as [EKS](https://aws.amazon.com/eks/) and [GKE](https://cloud.google.com/kubernetes-engine/), but be aware these can get pricy very quickly. (If you're a masochist, there's also [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)).

All the commands below should be run using a computer with Kubeconfig file that can connect to your cluster. In my case I have a dedicated LXC container running in Proxmox that is inside an isolated network that I have created for my k3s cluster that I set up using k3sup. Start by installing the Flux CLI tool:
```bash
# Linux
curl -s https://fluxcd.io/install.sh | sudo bash
# MacOS
brew install fluxcd/tap/flux
# Windows (requires Chocolatey)
choco install flux
```
Next go to GitHub and create a [Personal Access Token](https://github.com/settings/tokens). Make sure that you check all the boxes under `repo` and `read:packages`. Export this variable and your GitHub username and launch the bootstrap command!

```bash
# On Windows, replace this with 'set'.
export GITHUB_TOKEN=<gh_token>
export GITHUB_USER=<gh_username>
export GITHUB_REPO=<gh_repository>
flux bootstrap github --components-extra=image-reflector-controller,image-automation-controller --owner=$GITHUB_USER --repository=$GITHUB_REPO --branch=main --path=./clusters/prod --personal --read-write-key
```

Now what does this command do? First we tell Flux that we want to bootstrap a new cluster, and we want to store our Infrastucture-as-Code (IaC) on GitHub. We then list some extra components we want to install. These will be important later when we set up automatic image update scanning. We then pass some information about GitHub to Flux. Flux will automatically create `$GITHUB_REPO` and watch the `main` branch for updates. At the very end we note that the key we attached is a `--read-write-key`. This extra argument is critical for Flux to be able to write back to GitHub when image updates are found. Once you run this command give Flux a few minutes to deploy everything and fully initialize. In the meantime, you should see a new private repository on GitHub that has some Kubernetes manifest files in it. Congratulations, you now have a functioning GitOps setup!

## Deploy An Application
Of course, there is no point in setting up GitOps without deploying some apps through it. We're going to use GitHub and GitHub Actions to build and deploy an example application.

First go on GitHub and create a new public repository. Name it whatever you want, and create the following files (all at the root of the repository unless otherwise specified):
#### www/index.html
```html
<html>
    <body>
        <p>Hello from Kubernetes!</p>
    </body>
</html>
```
#### Dockerfile
```Dockerfile
FROM httpd:2.4-alpine
RUN rm /usr/local/apache2/htdocs/index.html
COPY ./www/ /usr/local/apache2/htdocs/
```
#### .github/workflows/image-build.yaml
```
{% raw %}
name: Create and publish a Docker image

on:
  push:
    branches: ['main']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Log in to the Container registry
        uses: docker/login-action@7f47463f5646678eb7ccf8f0f2e2d0896916a10a
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@548e2346a9987b56d8a4104fe776321ff8e23440
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          
      - name: Get current date
        id: date
        run: echo "::set-output name=date::$(date +%s)"

      - name: Build and push Docker image
        uses: docker/build-push-action@5e11b373bfed0d8024ef33d1586c675819690e95
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}-${{ github.sha }}-${{ steps.date.outputs.date }}
          labels: ${{ steps.meta.outputs.labels }}
{% endraw %}
```
After pushing that last file, GitHub should create an Action and automatically run it.

## Wrap-up
(Conclusion)
