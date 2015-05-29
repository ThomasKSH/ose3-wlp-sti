# OpenShift 3 STI for WAS Liberty Profile on RHEL7  

**Table of contents:**

* [Overview](#overview)
* [Required Downloads](#requirememts)
* [Usage](#usage)

## Overview

The following repo contains 3 directories with the files needed to build:

* **wlp-dockerfile**: A base Docker image that runs WAS Liberty Profile 8.5.5.5 on top of RHEL7.1 + IBM's JDK 1.7
* **wlp-sti-dockerfile**: An STI Docker image that takes already packaged ears, wars, jars and a single server.xml file and deploys them in the wlp base image to build a final wlp-sample-app image. 
* **wlp-sample-app**: Sample application files + WLP config file to build the final docker image 

## Required Downloads

### RHEL 7.1 Docker Images 

To the best of my knowledge, rhel7 base images are not openly accessible from Red Hat. These can be obtained via download from access.redhat.com or directly via docker pull from registry.access.redhat.com

   docker pull registry.access.redhat.com/rhel7.1

### IBM JDK and WAS Liberty Profile Binaries

Due to the size of the ibm-java-x86_64-sdk-7.1-2.10.bin and wlp-runtime-8.5.5.5.jar binary files they have not been added to the repo. You will have to download them from www.ibm.com and copy them into the wlp-dockerfile directory before build the wlp base image.

## Usage

The first step is to build the **wlp** and **wlp-sti** images. 

   docker build -t wlp wlp-dockerfile/
   docker build -t wlp-sti wlp-sti-dockerfile/

Once these images are ready in your local machine you have the option of either performing a local build of the final **wlp-sti-sample** image via STI CLI or by configuring the Openshift 3 application with a BuildConfig element that will take care of teh build when commanded via **osc start-build** or when invoked via webhook.  

### Command Line Interface STI

This method has the prerequisite of having the sti command line installed. 

   https://github.com/openshift/source-to-image/releases  

Once that is ready, we provision the application resources (route, service, deploymentconfig) via json file.

   OSE3_USER=joe
   OSE3_PROJECT=wlp-sti-project
   DOCKER_REGISTY=$OSE3_INTERNAL_REGISTRY_IP:$OSE3_INTERNAL_REGISTRY_PORT
   osc new-project --admin='$OSE3_USER' --display='WAS Liberty Profile Sample Application' $OSE3_PROJECT
   osc create -f cli-sti-app-config.json -n $OSE3_PROJECT

We then proceed to build the final image and push it into OSE3's internal Docker registry.  

   sti build wlp-sample-app/ wlp-sti wlp-sample-app   
   docker push $DOCKER_REGISTRY:$OSE3_PROJECT/wlp-sample-app

When OSE3 detects the new image, the deployment controller that is expecting it should spool up one pod of the wlp-sti-sample. The results can be checked via browser under the domain wlp-sti-sample.cloudapps.example.com 

### PaaS STI

Pending!

## Final Notes 

DISCLAIMER: I am a true believer of the Copy&Paste religion and as such most of the stuff here have been originated somewhere else. Praise or Blame them, not me. :p  

And finally, when you get it working, remember to scream: https://www.youtube.com/watch?v=xos2MnVxe-c
