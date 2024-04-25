# Demo Tekton

## Setup

### Create kind cluster

1. Install kind with [brew install kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-with-a-package-manager)
2. Roughly following the [kind quickstart](https://kind.sigs.k8s.io/docs/user/quick-start/)
3. Create cluster `kind create cluster`
4. List clusters `kind get clusters`
5. Install kubectl if you don't have it with [brew install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/)
6. See cluster info with `kubectl cluster-info --context kind-kind`
7. Install tekton pipelines on the cluster [here](https://tekton.dev/docs/installation/pipelines/)
8. Run `kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml`
9. Watch tekton get setup `kubectl get pods --namespace tekton-pipelines --watch`
10. Install `tkn` cli with [brew install tektoncd-cli](https://tekton.dev/docs/cli/#installation)
11. Follow the guide at [](https://archive.docs.dagger.io/0.9/213240/tekton)
12. In `dagger-task.yaml`, make sure the engine is latest version on line 20: `image: registry.dagger.io/engine:v0.11.1`
13. In `dagger-task.yaml`, update the script starting on line 44 to:
```sh
      #!/usr/bin/env sh
      apk add curl
      curl https://dl.dagger.io/dagger/install.sh | BIN_DIR=/usr/local/bin sh
      dagger call test --dir .
```
14. In `git-pipeline-run.yaml`, use `https://github.com/kpenfound/greetings-api` for the repo url

### Add Dagger Cloud token for visualizing and debugging pipeline runs
To use k8s secrets for dagger cloud:
1. Run `kubectl create secret generic dagger-cloud --from-literal=token='YOUR_TOKEN'`
2. See the secret information: `kubectl describe secrets dagger-cloud`

```Name:         dagger-cloud
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
token:  36 bytes
```

1. Change dagger-task.yaml to pull from a secret instead of string in the env section:

   ```- name: DAGGER_CLOUD_TOKEN
        valueFrom:
          secretKeyRef:
            name: $(params.dagger-cloud-token)
            key: "token"
   ```

1. Change git-pipeline-run.yaml param to reference the secret name instead of the token itself:

   ```- name: dagger-cloud-token
    value: "dagger-cloud"
   ```

### Run the pipeline and visualize the output in Dagger Cloud
1. Start the pipeline with `kubectl create -f git-pipeline-run.yaml`. This writes out the pipeline run with its ID in the terminal. You'll use the ID to watch the output (next step).
2. Watch the output with `tkn pipelinerun logs clone-read-run-<id> -f`
3. Notice output difference with `[dagger : read]` vs `[dagger : sidecar-dagger-engine]`. The latter is the Dagger engine logs
4. Go to https://dagger.cloud to view the pipeline running in realtime (streaming logs)
