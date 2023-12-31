# Flux ARM

> 💪️ An experiment to build Flux with BIG MUSCLES I mean, with ARM.


## Usage

These images are _incredibly slow_ to build on GitHub actions, so we have several building strategies:

### Multi-container Strategy

This strategy builds components separately (and sequentially):

 1. [build-deploy-base](.github/workflows/build-deploy-base.yaml) uses [Dockerfile.base](Dockerfile.base) to build up openpmix and prrte. (takes appoximately 1 hour 25-37 minutes)
 2. [build-deploy-flux-security](.github/workflows/build-deploy-flux-security.yaml) uses [Dockerfile.flux-security](Dockerfile.flux-security) to add Flux Security to the base image (1) (only takes 5 minutes)
 3. [build-deploy-flux-core](.github/workflows/build-deploy-flux-core.yaml) uses [Dockerfile.flux-core](Dockerfile.flux-core) to add Flux Core to (2). (takes approximately 1 hour 30-40 minutes)
 4. [build-deploy-flux-sched](.github/workflows/build-deploy-flux-sched.yaml) adds Flux Sched on top of Flux core via the [Dockerfile.flux-sched](Dockerfile.flux-sched) (takes approximately 52 to 1 hour 22 minutes)

### Spack

We also wanted to do a test build with [spack](spack). This didn't work on GitHub actions - but worked great on AWS! 🙌️ 
For any of the above images, if you need to test an update, do a test build uncommenting the pull request logic to trigger:

```diff
- #  pull_request: []
+   pull_request: []
```

And if you just want to rebuild as is, you can go to Actions -> (Choose workflow on left) -> Run Workflow (on right) to 
trigger a build, selected the named workflow.

### Single Container Strategy

And if you want a "all in one" build, although we don't use GitHub actions (it takes a LONG time) you can use:

 - [build-deploy.yaml](.github/workflows/build-deploy.yaml) uses the vanilla [Dockerfile](Dockerfile) to build everything (takes approximately 4 hours 25 minutes on linux/arm64)

Since this was slow on GHA, I decided to also try it on an AWS instance. You can use the setup instructions from [spack](spack) and once you
are in the instance (from the root here):

```bash
$ docker buildx build --platform linux/arm64 --tag ghcr.io/rse-ops/flux-arm:arm64 .
```

At this point you can tag for the date too, and push both (you'll need to add credentials to the instance)

```bash
docker tag ghcr.io/rse-ops/flux-arm:arm64 ghcr.io/rse-ops/flux-arm:june-2023-arm64
docker push ghcr.io/rse-ops/flux-arm --all-tags
```
