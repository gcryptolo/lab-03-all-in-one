# LAB-03 All in one - Openshift Build and Deploy

Sample code for the lab-03 of the OpenShift course. We redeploy our sample hello work application using the OpenShift CLI and explore what's happening behind the scenes.
We will also explore the OpenShift web console and see how we can use it to manage our applications.

## sample-hello-work LAB-3 index

This repository is intended to be a sample hello work for the following:

- Create sample java project to test oc cli to deploy on Openshift 

- Technologies used:
    - Java 21
    - Spring Boot 3.2.0
    - Maven 3.9.0
    - Docker
    - OpenShift Sandbox version 4.14
    - OpenShift CLI (oc)

## Prerequisites

- Familiarity with Docker
- Familiarity with OpenShift
- RedHat Account and Sandbox Access  [OpenShift Sandbox](https://developers.redhat.com/developer-sandbox)
- GitHub Account  [GitHub](https://github.com/)
- OC CLI installed


About 20 min of work if you are familiar with Openshift and oc cli.

## Objectives

Recreate the resource created in t first 2 LAB using oc cli commands creating the following resources all in one:
- BuildConfig
- ImageStream
- Deployment
- Service

After we will expose our service using the CLI and explore the resources created by the deployment.

## Steps

1. Clone Project
2. use oc cli to deploy the application
3. explore the resources created by the deployment
4. expose the application using CLI

## 1. Clone Project

Create a new repo on github, and clone this repo by typing:

```bash
   git clone https://github.com/gcryptolo/lab-03-all-in-one.git
 ```

copy the content of cloned project into your repo and push the code to origin.
```bash
   git add .
   git commit -m "Initial commit"
   git push origin main
```
## 2. use oc cli to deploy the application

Login into Openshift
```bash
   oc login --token=s<your_tocken> --server=<your_sandbox_url
```

You can retrieve login string from Openshift Sanbox Console clicking on the top right corner and select Copy Login Command

![img.png](doc%2Fimg%2Fimg.png)

Now we are logged in and can create our resources all in one typing:

```bash
   oc  oc new-app ./ --name lab-03-all-in-one --docker-image=<route_to_image_registry>/openshift/ubi8-openjdk-21:1.18
```

Remember to replace <route_to_image_registry> with the route to your image registry, in the fist LAB you learn how to retrive it.

**Important** The directory that contains the source code must be a git repository. 

The new-app command will create a BuildConfig, ImageStream, Deployment, and Service for you as you can see in the output of the command:


![img_1.png](doc%2Fimg%2Fimg_1.png)

## 3. explore the resources created by the deployment

 1. **BuildConfig:**

![img_2.png](doc%2Fimg%2Fimg_2.png)

As you can see in the source fragment the type is Git so if in your directory you have only the source code the app never be built.
It checks for Git repo and download the code from origin.

You can change the source type to Binary if you want to upload directly, check Openshift Documentation https://docs.redhat.com/en/documentation/openshift_container_platform/3.11/html/developer_guide/builds#binary-source

 2. **ImageStream:**

It create an image stream as we create in the first 2 LAB, 


 3. **Deployment:**

```yaml
spec:
  replicas: 1
  selector:
    matchLabels:
      deployment: lab-03-all-in-one
  template:
    metadata:
      creationTimestamp: null
      labels:
        deployment: lab-03-all-in-one
      annotations:
        openshift.io/generated-by: OpenShiftNewApp
    spec:
      containers:
        - name: lab-03-all-in-one
          image: 'image-registry.openshift-image-registry.svc:5000/giovanni-manzone-dev/lab-03-all-in-one@sha256:3e9fcff549f17db1b53ceba9e6c679465a9d61e3db1f23ad9244eabd893cd63d'
          ports:
            - containerPort: 8080
              protocol: TCP
            - containerPort: 8443
              protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

As you can see it create a simple deployment with 1 replica and a rolling update strategy.
There aren't any resource limit set or liveness probe, you can add them if you want to test the application.
In the next LABs we will explore how to add them.

    4. **Service:**
    
```yaml
spec:
  clusterIP: 172.30.224.29
  ipFamilies:
    - IPv4
  ports:
    - name: 8080-tcp
      protocol: TCP
      port: 8080
      targetPort: 8080
    - name: 8443-tcp
      protocol: TCP
      port: 8443
      targetPort: 8443
  internalTrafficPolicy: Cluster
  clusterIPs:
    - 172.30.224.29
  type: ClusterIP
  ipFamilyPolicy: SingleStack
  sessionAffinity: None
  selector:
    deployment: lab-03-all-in-one
  ```

So now we have our app deployed and exposed as service into Openshift

## 4. expose the application using CLI

And now we finish the LAB exposing the application using the CLI.
```bash
   oc expose svc lab-03-all-in-one
``` 

and also the route is exposed for us and reachable from the internet:

![img_3.png](doc%2Fimg%2Fimg_3.png)

## Conclusion

In this lab we finished to exploring Openshift build and deploy using Build Config and cli.

In the first 2 LABS we create all resource by hands and in this LAB we create all resources in one command using the oc cli, is importanto you understand all the steps and the resources created by one single command.


In the next LAB we will explore how to create a pipeline to build and deploy our application using Openshift Pipelines.
See you in the next LAB.