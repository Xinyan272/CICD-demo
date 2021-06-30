# CICD-demo

## Introduction
The repo for Tekton pipeline yaml & Flux2 deploy yaml & test app example
 
* **fluxcd-manifests**: Applying flux needs these yaml templates(automation deploy & automation image update included )
* **tekton-manifests**: kaniko pipeline & 2 methods of monitoring git changes(cronjob for git pull, git webhook)

> CICD structure:
![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2Fae30c178-2339-4549-a455-7b319289b995%2FUntitled.png?table=block&id=723631fc-2615-4e97-8a65-565a55db4c87&spaceId=1d921e6d-5547-4afd-ba4f-4e4c3b27b4ce&width=2880&userId=&cache=v2)

---
## Details

### 1. Flux2
* Install flux to cluster [installation doc](https://fluxcd.io/docs/installation/)
      
    Eg:
    ```bash
    flux bootstrap github \
      --owner=Xinyan272\
      --repository=test-flux \
      --branch=demo \
      --path=./cluster-env/demo1 \
      --personal \
      --read-write-key \
      --components-extra=image-reflector-controller,image-automation-controller    
    ```
* Deploy a remote application from Git

    Git repo has in addition to source code and deploy yaml files, flux kustomization monitor these yaml files regularly and synchronize deploy.
    
    synchronize manual:
  
    ```bash
    flux reconcile kustomization flux-system --with-source
    ```

* Image update automation
    1. imageRepository  ： The image repository that needs to be monitored (Eg: imageRepoUrl/xinyan/test)
    2. imagePolicy : The ImagePolicy type gives rules for selecting a “latest” image from a scanned ImageRepository. [imagePolicy examples](https://fluxcd.io/docs/components/image/imagepolicies/#examples)
    3. imageUpdateAutomation: An automation process that will update a git repository, based on image policiy objects in the same namespace.

    **Attention:**
    1. The automatic image update mechanism needs to comment imagePolicy after 'image' of deploy yaml file, otherwise it cannot be updated. Eg:
        ```yaml
        - image: imageRepoUrl/xinyan/test # {"$imagepolicy": "namespace:imagePolicyName"}
        ```
    2. Every different image needs imageRepo & imagePolicy respective 

### 2. Tekton 
Using kaniko task & pipeline to build image and push to remote image repository.

2 methods of synchronizing git 
* git pull -> cronjob 
  
  A timed job to periodically monitor whether git has changed, and trigger a new pipelineRun if there is a change, nothing do if not.
  
  *reference: [win5do/tekton-cicd-demo](https://github.com/win5do/tekton-cicd-demo/tree/master/src/pull)*

* git webhook -> tekton trigger

  The pipelineRun is triggered by git webhook. As soon as git is updated, a new pipelineRun will be created automatically.

