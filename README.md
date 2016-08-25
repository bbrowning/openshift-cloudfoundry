# Cloud Foundry Applications (and buildpacks) on OpenShift

This repository contains the source for the
`bbrowning/openshift-cloudfoundry` Docker image which lets you run
applications that use Cloud Foundry buildpacks on top of OpenShift 3.

If you're interested in running Cloud Foundry applications on
OpenShift then you should also check out
https://github.com/bbrowning/ocf. That's a command-line tool that
emulates `cf push` but deploys the applications to OpenShift instead
of Cloud Foundry.

**Note:** As of right now, this only bundles the Java, Node.js, and
Ruby buildpacks. Custom buildpacks can be provided by setting the
`BUILDPACK_URL` environment variable in the build and deployment
configurations.

Please report any bugs you find via GitHub issues.

## Deploying a locally built Java application

This emulates the `cf push target/some-app.jar` command in
Cloud Foundry. Replace target/some-app.jar with any executable jar or
deployable war file. Read
https://github.com/cloudfoundry/java-buildpack/blob/master/README.md
for more information on supported application types.

**Note:** The `-docker19` suffix here is temporary until OpenShift
  and/or the Red Hat CDK update to a newer Docker version that can
  consume automated builds from Docker Hub again.

    oc new-build bbrowning/openshift-cloudfoundry-docker19 --binary=true --name=cf-test
    oc start-build cf-test --from-file=target/some-app.jar --follow

Once the initial build is done, create a new application from this
image and expose it to the outside world.

    oc new-app cf-test
    oc expose svc/cf-test

After the deploy finishes (check via `oc status`), use `oc get routes`
to lookup the hostname and copy/paste that into your browser to test.

After making changes to the app locally, just run the `oc start-build`
command and the application will redeploy after the new build
finishes.


## Deploying the Cloud Foundry sample Node application

    oc new-app bbrowning/openshift-cloudfoundry-docker19~https://github.com/cloudfoundry-samples/cf-sample-app-nodejs.git --follow
    oc expose svc/cf-sample-app-nodejs


## Testing changes to this Docker image on OpenShift

First, clone this repository and `cd` into the newly cloned repo.

    oc new-build --binary=true --name=cloudfoundryish
    oc start-build cloudfoundryish --from-dir=. --follow

Then, use `cloudfoundryish` instead of
`bbrowning/openshift-cloudfoundry` to subsequent `oc new-build`
commands to use the locally changed image.


## Releasing new versions of this image to Docker Hub

Automated builds are performed on every commit of this repo to the
`bbrowning/openshift-cloudfoundry` Docker repository.

However, there's an additional repository that needs to be manually
updated for Docker 1.9 users of
`bbrowning/openshift-cloudfoundry-docker19`. To do that from inside
the Red Hat CDK:

    git clone git clone https://github.com/bbrowning/openshift-cloudfoundry
    cd openshift-cloudfoundry
    docker build -t bbrowning/openshift-cloudfoundry-docker19 .
    docker tag bbrowning/openshift-cloudfoundry-docker19 registry-1.docker.io/bbrowning/openshift-cloudfoundry-docker19
    docker login registry-1.docker.io
    docker push registry-1.docker.io/bbrowning/openshift-cloudfoundry-docker19
