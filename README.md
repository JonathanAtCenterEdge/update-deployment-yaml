## `JonathanAtCenterEdge/update-deployment-yaml@main`
GitHub action that utilizes the yq yaml processor to modify the image and environment variables for a specified container for a single k8s deployment yaml resource

This takes a `.yaml` file as an input that contains a `Kind: Deployment` resource inside of it, and modifies the `image` and `env` fields in `.spec.template.spec.containers[]` matching a `container[]` entry by name using the [yq yaml processor](https://github.com/mikefarah/yq) that's shipped in the `ubuntu-20.04` and up GitHub action runner

#### That means this action only supports `ubuntu-20.04` and `ubuntu-22.04` runners


## Usage
```yaml
- name: Update
    uses: JonathanAtCenterEdge/update-deployment-yaml@main
    with:
      image: traefik/whoami:v1.9 # new image version
      container: whoami-container # the container in the deployment matching via name property
      deployment: ./deployment.yaml # the deployment file, usually this has already been baked by kustomize
      env: TEXT_RESPONSE=test # a list of environment variables to replace, comma seperated and optionally multieline
      
      indention: 1 # optional configureation of the yq indention switch
      diff-lines: 3 # optionally configure the diff command context lines
```
## Multiple environment variables
You can pass multiple environment variables seperated by a comma, optionally multi-line is supported
```yaml
with:
  env: | # mult-line
      TEXT_RESPONSE=test,
      TEXT_RESPONSE_NEW=test,
      TEXT_RESPONSE_LEGACY=test,
```

## Highlighted modifications
Whenever the action finishes it will print a diff of the modifications, for example
```diff
*** 31,38 ****
              - name: PORT
                value: "8080"
              - name: TEXT_RESPONSE
!               value: Hello, World!
!           image: containous/whoami
            name: whoami-container
            ports:
              - containerPort: 80
--- 31,38 ----
              - name: PORT
                value: "8080"
              - name: TEXT_RESPONSE
!               value: 00000-00000000-0000000
!           image: test
            name: whoami-container
            ports:
              - containerPort: 80
```

## Example
```yaml
- name: Update
    uses: JonathanAtCenterEdge/update-deployment-yaml@main
    with:
      image: traefik/whoami:v1.9 # new image version
      container: whoami-container # the container in the deployment matching via name property
      deployment: ./deployment.yaml # the deployment file, usually this has already been baked by kustomize
      env: TEXT_RESPONSE=test # a list of environment variables to replace, comma seperated and optionally multieline
      
      indention: 1 # optional configureation of the yq indention switch
      diff-lines: 3 # optionally configure the diff command context lines
```

Will update my baked `./deployment.yaml` from this
```yaml
apiVersion: v1
kind: Service
metadata:
  name: whoami-service
spec:
  ports:
  - nodePort: 30080
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: whoami
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
      - env:
        - name: PORT
          value: "8080"
        - name: TEXT_RESPONSE
          value: Hello, World!
        image: containous/whoami
        name: whoami-container
        ports:
        - containerPort: 80

```
into this
```yaml
apiVersion: v1
kind: Service
metadata:
  name: whoami-service
spec:
  ports:
    - nodePort: 30080
      port: 80
      protocol: TCP
      targetPort: 80
  selector:
    app: whoami
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - env:
            - name: PORT
              value: "8080"
            - name: TEXT_RESPONSE
              value: "test"
          image: traefik/whoami:v1.9
          name: whoami-container
          ports:
            - containerPort: 80
```