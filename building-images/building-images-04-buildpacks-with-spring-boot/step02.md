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

# Clone git repos (sample apps and sample buildpack)
cd ${DEMO_TEMP}
if [ ! -d java-app ]; then git clone git@github.com:ciberkleid/hello-java java-app; fi
cd java-app
rm -rf demo
rm -rf Dockerfiles
rm -rf kpack
rm -rf bump
rm -rf manifest.yml
rm HELP.md
./mvnw dependency:go-offline
docker pull gcr.io/paketo-buildpacks/builder:base-platform-api-0.3

# Start demo
clear
@_ECHO_ON
# Spring Boot
ls
./mvnw spring-boot:build-image -DskipTests
@_SKIPdocker images | grep "builder\|java"
@_SKIPpack inspect-image hello-java:1.0.0 --bom | jq '.local[] | select(.name == "jre")'
pack inspect-image hello-java:1.0.0 --bom | jq '.local[].name' -r
pack inspect-image hello-java:1.0.0 --bom | jq '.local[] | select(.name == "dependencies")'

@_SKIPmvn spring-boot:build-image -Dspring-boot.build-image.imageName=ciberkleid/hello-java:1.0.0
@_SKIPmvn spring-boot:build-image -Dspring-boot.build-image.builder=heroku/buildpacks:18 -Dspring-boot.build-image.imageName=hello-java:1.0.0:heroku

# Configuration (Paketo Java BP)
@_SKIP./mvnw spring-boot:build-image -DskipTests
@_SKIP-e BP_DEBUG_ENABLED=true

clear
clear
