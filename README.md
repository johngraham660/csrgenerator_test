# Create the csrgenerator app service in Openshift
In order to successfully host the csrgenerator application service in Openshift there are a couple of things to do.

First we need to create a new project (namespace) to host the service. The NGINX web service in the base image runs as root and by default Openshift will not allow this to run. So having one namespace allocated for root apps makes sense as the permissions to allow privileged containers  is performed at a namespace level, you cannot do this per application service (as yet).

In order to allow privileged containers we need to modify the SCC to allow user default to run with the anyuid privilege.

### Login to Openshift via the CLI
`oc login -u <user> https://admin.ocp.10.137.0.166.xip.io:8443`

### Create the project for root apps
`oc new-project root-apps`

### Update the scc for user default
`oc adm policy add-scc-to-user anyuid -z default`

### Create the application
`oc new-app https://gitlab.com/virtua-galaxy/openshift/csrgenerator.git

### Disable SSL verification of the Gitlab Repo
The Gitlab certs are self signed certificates and Openshift will not verify them as being secure, this will cause the build to fail. In order to work around this issue for the time being I am disabling the Git SSL verification by defining an environment variable in the BuildConfig.
`oc env bc/csrgenerator GIT_SSL_NO_VERIFY=true`

### Issue a new build (First one will fail)
`oc start-build bc/csrgenerator`
### 
### Expose the route
`oc expose service csrgenerator --hostname=csrgenerator.apps.ocp.bskyb.vm --port=80`

If there is no wildcard DNS for your Openshift cluster you will need to create a new A record for the application service in DNS. 

If you are just testing, add an entry to your /etc/hosts file pointing to the load balancer FE interface. The name must match the exposed route so that Openshift can direct the traffic via the appropriate service.

`10.137.0.166 csrgenerator.apps.ocp.bskyb.vm`