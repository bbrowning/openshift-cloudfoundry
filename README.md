# CloudFoundry Buildpacks on OpenShift

This repository contains the source for the
`bbrowning/openshift-cloudfoundry` Docker image which lets you run
applications that use CloudFoundry buildpacks on top of OpenShift 3.

**Note:** As of right now, this only supports the Java and Node.js
buildpacks. Support for the other CloudFoundry-provided buildpacks as
well as custom buildpacks will be added in the future.

Please report any bugs you find via GitHub issues.

## Deploying a locally built Java application

This emulates the `cf push target/some-app.jar` command in
CloudFoundry. Replace target/some-app.jar with any executable jar or
deployable war file. Read
https://github.com/cloudfoundry/java-buildpack/blob/master/README.md
for more information on supported application types.

    oc new-build bbrowning/openshift-cloudfoundry --binary=true --name=cf-test
    oc start-build cf-test --from-file=target/some-app.jar
    oc logs -f bc/cf-test

Once the initial build is done, create a new application from this
image and expose it to the outside world.

    oc new-app cf-test
    oc expose svc/cf-test

After the deploy finishes (check via `oc status`), use `oc get routes`
to lookup the hostname and copy/paste that into your browser to test.

After making changes to the app locally, just run the `oc start-build`
command and the application will redeploy after the new build
finishes.


## Deploying the CloudFoundry sample Node application

    oc new-app bbrowning/openshift-cloudfoundry~https://github.com/cloudfoundry-samples/cf-sample-app-nodejs.git
    oc logs -f bc/cf-sample-app-nodejs
    oc expose svc/cf-sample-app-nodejs


## Testing changes to this Docker image on OpenShift

First, clone this repository and `cd` into the newly cloned repo.

    oc new-build --binary=true --name=cloudfoundryish
    oc start-build cloudfoundryish --from-file=Dockerfile
    oc logs -f bc/cloudfoundryish

Then, use `cloudfoundryish` instead of
`bbrowning/openshift-cloudfoundry` to subsequent `oc new-build`
commands to use the locally changed image.
