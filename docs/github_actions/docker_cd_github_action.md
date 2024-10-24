# Github Action to Deploy Dockerfile to Remote Registry

Name the following file `docker_<remote_registry_name>_pkg.yaml` and add it to the `workflows` directory in your `.github` directory.

This file is currently configured to build and deploy an image to github packages. Since we are running our deployment as a Github Action we also benefit with having the github package that is created from the action associated with the repository that is running the action.

If you would like to change the remote registry update the domain value specified at `env.REGISTRY`. If you would like to update how this action is triggered update the `on` value.

Currently this action is triggered from a github release.

```yaml
# .github/workflows/docker_docker_github_pkg.yaml
name: Create and publish a Docker image

# Configures this workflow to run every time a release is created.
on:
  release:
    types: ["published"]

# Defines two custom environment variables for the workflow. These are used for the Container registry domain, and a name for the Docker image that this workflow builds.
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${%raw%}{{ github.repository }}{%endraw%}

# There is a single job in this workflow. It's configured to run on the latest available version of Ubuntu.
jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    # Sets the permissions granted to the `GITHUB_TOKEN` for the actions in this job.
    permissions:
      contents: read
      packages: write
      #
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      # Uses the `docker/login-action` action to log in to the Container registry registry using the account and password that will publish the packages. Once published, the packages are scoped to the account defined here.
      - name: Log in to the Container registry
        uses: docker/login-action@65b78e6e13532edd9afa3aa52ac7964289d1a9c1
        with:
          registry: ${%raw%}{{ env.REGISTRY }}{%endraw%}
          username: ${%raw%}{{ github.actor }}{%endraw%}
          password: ${%raw%}{{ secrets.GITHUB_TOKEN }}{%endraw%}
      # This step uses [docker/metadata-action](https://github.com/docker/metadata-action#about) to extract tags and labels that will be applied to the specified image. The `id` "meta" allows the output of this step to be referenced in a subsequent step. The `images` value provides the base name for the tags and labels.
      - name: Extract metadata (tags, labels) for Docker
        id: meta
        uses: docker/metadata-action@9ec57ed1fcdbf14dcef7dfbe97b2010124a938b7
        with:
          images: ${%raw%}{{ env.REGISTRY }}{%endraw%}/${%raw%}{{ env.IMAGE_NAME }}{%endraw%}
          tags: |
            type=semver,pattern={%raw%}{{version}}{%endraw%} # use git tag
      # This step uses the `docker/build-push-action` action to build the image, based on your repository's `Dockerfile`. If the build succeeds, it pushes the image to GitHub Packages.
      # It uses the `context` parameter to define the build's context as the set of files located in the specified path. For more information, see "[Usage](https://github.com/docker/build-push-action#usage)" in the README of the `docker/build-push-action` repository.
      # It uses the `tags` and `labels` parameters to tag and label the image with the output from the "meta" step.
      - name: Build and push Docker image
        uses: docker/build-push-action@f2a1d5e99d037542a71f64918e516c093c6f3fc4
        with:
          context: .
          push: true
          tags: ${%raw%}{{ steps.meta.outputs.tags }}{%endraw%}
          labels: ${%raw%}{{ steps.meta.outputs.labels }}{%endraw%}
```

Accompanying documentation to added to a project's readMe.md

```md
This project utilizes [Github Actions](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions#publishing-a-package-using-an-action) for deploying a production ready docker container to the github container registry. For more information see [working with the container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry).

To push a new container image to github packages create a `release` from the `main` branch with a specified [git tag](https://git-scm.com/book/en/v2/Git-Basics-Tagging). The git tag should be labelled with a version number such as [1.2.3](https://github.com/docker/metadata-action?tab=readme-ov-file#tags-input).
```
