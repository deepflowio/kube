# integration tests

This is a working example of a kubernetes application `dapp`, that is deployed on CI during circle's `kind_k8s` and `kind_compile` job.

## Behavior
The app currently only does what the `job_api` example does, but from within the cluster, so it needs the rbac permissions to `create` a `job` in `batch`.

## CircleCI
It's a slightly complicated process to do this on CI due to cache boundaries and switching between machine and docker executors on circleci, so this explains the process:

### kind_compile
The first job compiles the app with `cargo build -p tests --release` using [muslrust](https://github.com/clux/muslrust) for the musl cross compile.

The static binary is then persisted to circleci's workspace.

### kind_k8s
This resumes with a machine excutor (which can run `docker` commands).

It carries on building the tiny image with the mounted binary from the workspace into a [distroless:static](https://github.com/GoogleContainerTools/distroless) image.

This is then pushed to [dockerhub/clux/kube-dapp](https://hub.docker.com/repository/docker/clux/kube-dapp/tags), kind of pointlessly, but it allows debugging a build locally from an artifact from CI.

At this point, we install [kind](https://kind.sigs.k8s.io/) using the direct binary install from their [github releases](https://github.com/kubernetes-sigs/kind/releases), and use the [circleci kubernetes orb](https://circleci.com/orbs/registry/orb/circleci/kubernetes) to apply our [test yaml](./deployment.yaml).

It's successful if the app exits successfully, without encountering errors.

## Locally
Start a cluster first. Say, via `make minikube-create && make minikube` or `make kind-create && make kind`.

NB: Mileage may vary. At time of writing `kind` was best for CI (but had troubles loading images), whereas `minikube` seems more battle tested for local development at the moment.

### Building Yourself
Run `make integration-test` to cross compile `dapp` with `muslrust` locally using the same docker image, and then deploy it to the current active cluster.

### Using CI built image
Switch to the same git sha used on CI, and use `make integration-pull`.