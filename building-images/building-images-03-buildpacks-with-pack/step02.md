# Step Title Here

Objective:


Step objective here.

In this step, you will:
- Do this
- And that

## Section heading here
Explain section

@_ECHO_OFF
dclean

# build caches in pack are docker volumes
# TODO: update to be more selective and just rm pack volumes
docker volume prune -f

# launch caches are images
# docker image rm... OR pack build ... --clear-cache

if [ ! -d go-app ]; then git clone git@github.com:ciberkleid/hello-go.git temp-hello-go; mkdir -p go-app; mv temp-hello-go/src/* ./go-app/; rm -rf temp-hello-go; fi
unset HELLO_ARG

@_SKIPif [ ! -d node-app ]; then git clone git@github.com:paketo-buildpacks/samples.git paketo-samples; mkdir -p node-app; mv paketo-samples/demo-apps/app-source/* ./node-app/; rm -rf paketo-samples; fi

if [ ! -d sample-buildpack ]; then git clone git@github.com:buildpacks/samples.git cnb-samples; echo -e "\n[[stacks]]\nid = \"io.buildpacks.stacks.bionic\"" >> cnb-samples/buildpacks/hello-world/buildpack.toml; mkdir -p sample-buildpack; mv cnb-samples/buildpacks/hello-world/* ./sample-buildpack/; rm -rf cnb-samples; fi

# Pull images
docker pull gcr.io/paketo-buildpacks/builder:base
docker pull gcr.io/paketo-buildpacks/run:base-cnb
@_SKIPdocker pull gcr.io/paketo-buildpacks/run:0.0.17-base-cnb

pack build go-img -p go-app
@_SKIPpack rebase go-img --run-image gcr.io/paketo-buildpacks/run:base-cnb

# Start demo
clear
cd go-app
@_ECHO_ON
# pack build
ls
pack set-default-builder gcr.io/paketo-buildpacks/builder:base
docker images
pack build go-img
@_SKIPpack build go-img -e BP_IMAGE_LABELS=maintainer=me@example.com -e BP_OCI_AUTHORS=me@example.com

@_SKIP# metadata analyzer gets from image:
@_SKIPdocker inspect go-img | jq '.[].Config.Labels."io.buildpacks.lifecycle.metadata"' | jq 'fromjson'

# inspect
pack inspect-image go-img
@_SKIPpack inspect-image go-img --bom | jq

# user
docker run --rm --entrypoint /bin/sh go-img -c "id"

# inspect-builder
pack inspect-builder

# rebase (patch OS)
@_SKIPdocker pull gcr.io/paketo-buildpacks/run:0.0.17-base-cnb
@_SKIPdocker tag go-img go-img:bad
pack rebase go-img --run-image gcr.io/paketo-buildpacks/run:0.0.17-base-cnb
@_SKIPdocker images | grep go-img
docker images
catd <(docker inspect 92045f666fcd) <(docker inspect go-img) | tail -n20

@_SKIP# ALTERNATE WORKFLOW
@_SKIP# pack rebase go-img --run-image ...
@_SKIPdocker tag gcr.io/paketo-buildpacks/run:base-cnb gcr.io/paketo-buildpacks/run:bad
@_SKIPdocker tag go-img go-img:bad
@_SKIPdocker tag gcr.io/paketo-buildpacks/run:0.0.17-base-cnb gcr.io/paketo-buildpacks/run:base-cnb
@_SKIPdocker images
@_SKIPpack rebase go-img --no-pull
@_SKIPdocker images | grep go-img
@_SKIPcatd <(docker inspect go-img:bad) <(docker inspect go-img) | tail -n20

@_SKIP# pack build ... --publish   (rebase too)

# custom buildpack
ls -l ../sample-buildpack/*
pack build go-img --buildpack from=builder --buildpack ../sample-buildpack

clear
clear
