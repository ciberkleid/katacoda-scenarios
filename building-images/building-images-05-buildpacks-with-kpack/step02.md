Commit and push the changes you made throughout this scenario to your new fork.

```
git add -A
git commit -m 'Changes from the Intro Scenario'
git push origin master
```{{execute}}

# Step Title Here

Objective:


Step objective here.

In this step, you will:
- Do this
- And that

## Section heading here
Explain section

@_ECHO_OFF
setBatLang YAML

echo "### Installing kpack and kpack logs CLI"
KPACK_VERSION=0.0.8
kubectl apply -f https://github.com/pivotal/kpack/releases/download/v${KPACK_VERSION}/release-${KPACK_VERSION}.yaml
curl -L https://github.com/pivotal/kpack/releases/download/v${KPACK_VERSION}/logs-v${KPACK_VERSION}-macos.tgz | tar zx && chmod +x logs && mv logs ~/opt/logs
echo "### Finished installing kpack and kpack logs CLI"

kubectl create ns kpack-builds
kubectl config set-context --current --namespace=kpack-builds

if [ ! -d hello-go ]; then git clone git@github.com:ciberkleid/hello-go.git; fi

cd hello-go

# Set credentials, then create secret:
#kubectl apply -f kpack/secret.yaml
kubectl apply -f kpack/service-account.yaml
kubectl apply -f kpack/builder.yaml
kubectl apply -f kpack/image.yaml

# Start demo
clear
@_ECHO_ON
# kpack resources
kubectl api-resources --api-group build.pivotal.io

# config
bat kpack/builder.yaml
bat kpack/image.yaml

# trigger build
printf "#" >> bump; git add bump; git commit -m "bump commit id"; git push

open https://hub.docker.com/repository/docker/ciberkleid/hello-go

kubectl describe image hello-go | more

# reason for build
LAST_BUILD=$(kubectl describe image hello-go | grep "Latest Build Ref" | sed 's/.* //')
kubectl get build $LAST_BUILD -o json | jq .metadata.annotations

# build log
logs -namespace kpack-builds -image hello-go

#kubectl get builds
#kubectl get image hello-go


