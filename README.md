# OpenShift 3 STI for WAS Liberty Profile on RHEL7  

**Table of contents:**

* [Overview](#overview)
* [Required Downloads](#requirememts)
* [Usage](#usage)
* [Final Notes](#final)

## Overview

The following repo contains 3 directories:

* **wlp-dockerfile**: A base Docker image that runs WAS Liberty Profile 8.5.5.5 on top of RHEL7.1 + IBM's JDK 1.7
* **wlp-sti-dockerfile**: An STI Docker image that takes already packaged ears, wars, jars and a single server.xml file and deploys them in the wlp base image to build a final wlp-sample-app image. 
* **wlp-sample-app**: Sample application files + WLP config file to build the final docker image 

## Required Downloads

You will need the following to make this work.

### RHEL 7.1 Docker ImagesYou will need the following to make this work. 

To the best of my knowledge, rhel7 base images are not openly accessible from Red Hat. These can be obtained via download from access.redhat.com or directly via docker pull from registry.access.redhat.com

    docker pull registry.access.redhat.com/rhel7.1

### IBM JDK and WAS Liberty Profile Binaries

Due to the size of the ibm-java-x86_64-sdk-7.1-2.10.bin and wlp-runtime-8.5.5.5.jar binary files they have not been added to the repo. You will have to download them from www.ibm.com and copy them into the wlp-dockerfile directory before build the wlp base image.

## Usage

The first step is to build the **wlp** and **wlp-sti** images. 

    cp ~/Downloads/wlp-runtime-8.5.5.5.jar wlp-dockerfile/
    cp ~/Downloads/ibm-java-x86_64-sdk-7.1-2.10.bin wlp-dockerfile/
    docker build -t wlp wlp-dockerfile/
    docker build -t wlp-sti wlp-sti-dockerfile/

Once these images are ready in your local machine you have the option of either performing a local build of the final **wlp-sti-sample** image via STI CLI or by configuring the Openshift 3 application with a BuildConfig element that will take care of the build when commanded via **osc start-build** or when invoked via webhook.  

### Command Line Interface STI

This method has the prerequisite of having the sti command line installed. 

Get it here: https://github.com/openshift/source-to-image/releases  

Once that is ready, provision the application resources (route, service, deploymentconfig) via json file.

    OSE3_USER=joe
    OSE3_PROJECT=wlp-sti-project
    osadm new-project $OSE3_PROJECT --display-name="WLP STI Sample" \ 
      --description="WAS Liberty Profile Sample application built through STI" --admin=$OSE3_USER
    osc create -f wlp-sample-app/cli-sti-sample-app.json -n $OSE3_PROJECT

Then proceed to build the final image and push it into OSE3's internal Docker registry.

    DOCKER_REGISTRY="$(osc describe service docker-registry -n default | grep IP | awk '{print $2}'):5000"
    IMAGE_NAME="$DOCKER_REGISTRY/$OSE3_PROJECT/wlp-sample-app"
    sti build --forcePull=false wlp-sample-app/ wlp-sti $IMAGE_NAME
    docker push $IMAGE_NAME

When OSE3 detects the new image, the deployment controller that is expecting it should spool up one pod of the wlp-sti-sample. The results can be checked via browser under the domain wlp-sample-app.cloudapps.example.com 

### PaaS STI

Pending!

NOTE: As far as I know currently Openshift only supports git repos as source for STI strategy in a BuildConfig, so it does not makes that much sense to use PaaS STI for pushing binaries to the PaaS. 

## Final Notes 

DISCLAIMER: I am a true believer of the Copy&Paste religion. As such most of the stuff here has been originated somewhere else. Praise or blame them, not me. :-p  

COLLABORATION: Just do it! :D How about the PaaS STI section? or instead of the Json config a template? Feel free to add you own ideas.  

IMPORTANT: And finally, when you get it working, remember to scream like I did:  
https://www.youtube.com/watch?v=xos2MnVxe-c

Cheers,
--pablo 
