## Sonatype Platform Demo (JAVA) 
###Exploit Demo for CVE-2017-5638

Based on https://github.com/piesecurity/apache-struts2-CVE-2017-5638

### Demo-able Features:
* Java Application
* Struts2 2.5.10 as vulnerable

* Ahab: Configuration of Ahab in the `Dockerfile`
* IDE Integration
  * Breaking Changes
  * Transitive Solver
* IaC Pack: `aws.large.tfplan` included
* Azure DevOps: `azure-pipelines.yml`
  * Sonatype Lift step
  * NeuVector scanner step
  * Push built container to Nexus Repo
* Jenkins: `Jenkinsfile`
* Muse: Includes custom tool config for more results `.muse.toml`
* IntelliJ run configurations for common tasks `.run/`
* Nx Container: Sample CRD files for struts app `.container-crds/`



### The Exploit:
Pre-requisites: have docker, and a jre installed

1. fork this repo
1. Build and package the app
   > `./mvnw clean package`
1. Build the image 
   > `docker build --no-cache --build-arg IQ_USER=admin --build-arg IQ_TOKEN=Nexus!23 --build-arg IQ_STAGE=build --build-arg IQ_SERVER=http://host.docker.internal:8070 -t struts-hack .`
1. Start the container 
   > `docker run -d -p 9080:8080 struts-hack`
1. once container comes online - verify by running in browser 
   >`http://localhost:9080`

To begin testing RCE - run the exploit.py file.
> `python exploit.py http://localhost:9080/orders/3 "CMD"` 

Try with different CMDs like
* `pwd` - where are we?
* `whomai` - what user are we running this?
* `ls -la` - what's in my directory?
* `ls /` - what's my machine
* `ls /etc` - what else we can find?

## How to Fix!
Use the Nexus Lifecycle Component Information Panel to identify a non-vulnerable version of struts2-core. 
Update the POM to that version and rebuild.You can also rebuild the docker image and run it to retry the attack.
