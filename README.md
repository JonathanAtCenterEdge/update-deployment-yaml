# update-deployment-yaml
GitHub action that utilizes the yq yaml processor to modify the image and environment variables for a specific container in a k8s deployment yaml resource

- `udy_add_<env-var-to-add>` adds the environment variable to the container spec
- `udy_replace_<env-var-to-replace>` replaces the environment variable if matched in the container spec