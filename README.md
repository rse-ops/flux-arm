# Flux ARM

> 💪️ An experiment to build Flux with BIG MUSCLES I mean, with ARM.


## Usage

These images are _incredibly slow_ to build on GitHub actions, so we have a multi-container
building strategy:


 1. [build-deploy-base](.github/workflows/build-deploy-base.yaml) uses [Dockerfile.base](Dockerfile.base) to build up openpmix and prrte. (takes appoximately 1 hour 25-37 minutes)
 2. [build-deploy-flux-security](.github/workflows/build-deploy-flux-security.yaml) uses [Dockerfile.flux-security](Dockerfile.flux-security) to add Flux Security to the base image (1)
 3. **build-deploy-flux-sched** (TBA)
 4. **build-deploy-flux-core** (TBA)
 
And if you want a "all in one" build, although we don't use GitHub actions (it takes a LONG time) you can use:

 - [build-deploy.yaml](.github/workflows/build-deploy.yaml) uses the vanilla [Dockerfile](Dockerfile) to build everything (takes approximately 4 hours 25 minutes on linux/arm64)
 
For any of the above images, if you need to test an update, do a test build uncommenting the pull request logic to trigger:

```diff
- #  pull_request: []
+   pull_request: []
```

And if you just want to rebuild as is, you can go to Actions -> (Choose workflow on left) -> Run Workflow (on right) to 
trigger a build, selected the named workflow.

*under development*
