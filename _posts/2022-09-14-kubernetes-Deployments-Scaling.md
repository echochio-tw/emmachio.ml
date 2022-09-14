---
layout: post
title: kubernetes Deployments, Scaling, Updates, and Rollbacks
date: 2022-09-14
tags: kubernetes
---

網路看到的保留一下

Activity: Deployments, Scaling, Updates, and Rollbacks 

**In this activity, you'll scale, update, and rollback**  

**To complete this activity, first install Docker:** 

[https://docs.docker.com/docker-for-windows/install/  ](https://docs.docker.com/docker-for-windows/install/)

**We are using the recently installed minikube.**  

1. **Create a folder or directory, c:\my-nginx-app** 

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.001.png">

**Open Powershell, and start Minikube, run: minikube start --force**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.002.png">

2. **Create a new file called Dockerfile, and open it for editing.** 

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.003.png">

3. **Enter lines:  FROM nginx** 

**MAINTAINER <*name*>   <*email*>**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.004.png">

4. **Open Powershell as administrator, and cd to directory: c:\my-nginx-app**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.005.png">

5. **Set the Docker environment. Run: minikube docker-env and minikube docker-env | Invoke-Expression.**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.006.jpeg">

6. **Run the Docker build. Run the exact line: docker build -t nginx:1.17.10 .  Image nginx, version 1.17.10 is downloaded and built.** 

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.007.jpeg">

7. **List the Docker images. Run: docker images**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.008.png">

8. **Create the Kubernetes deployment. Run: kubectl create deployment greetings-app -- image=grettings-app:ver1.0**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.009.png">

9. **Get the deployments and pods. Run: kubectl get deployments,pods -l app=my-nginx**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.010.jpeg">

10. **Scale up the deployment to 3 replicas. Run: kubectl scale deployment my-nginx -- replicas=3**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.011.png">

11. **Look at the new pods created as the replicas. Run: kubectl get pods -l app=my-nginx**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.012.jpeg">

**Review the output.** 

13. **Create a service to access the pods and container. The service is of type nodeport and map the execution port 80, to port 80.** 

**Run: kubectl create service nodeport my-nginx --tcp=80:80**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.013.png">

14. **Get the public IP address. Run: minikube ip**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.014.png">

15. **Get the execution port. Run: kubectl get services -l app=my-nginx**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.015.png">

**The mapped port is: 31996.**  

**Access the application with url: <minikube ip>:<mapped port>  In this example: http://172.17.107.38:31996** 

16. **Enter the URL into a browser**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.016.jpeg">

17. **Get the image on which the deployment is based. Run: kubectl describe deployment my-nginx**  

**See the deployment’s image: it should be nginx:1.17.10** 

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.017.jpeg">

18. **Get the revision history. Run: kubectl rollout history deployment my-nginx**  

**kubectl rollout history deployment my-nginx –revision=1**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.018.png">

19. **Get information on revision 1. Run: kubectl rollout history deployment my-nginx -- revision=1**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.019.jpeg">

20. **Build a new version 1.18.0 image for deployment my-nginx. This is a later version  Run: docker build -t nginx:1.18.0  .**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.020.png">

**View the new image at version 1.18.0. Run: docker images**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.021.png">

21. **Set the deployment image to the new nginx 1.18.0 version.** 

**Run: kubectl set image deployment my-nginx nginx=nginx:1.18.0 --record**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.022.png">

**The deployment’s image is updated.**  

22. **Describe the deployment and see the deployment’s new image. Run: kubectl describe deployment my-nginx**  

**The deployment is now running image version 1.18.0**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.023.jpeg">

23. **View the deployment’s history. Run: kubectl rollout history deployment my-nginx**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.024.png">

24. **Get the details on revision 2. Run: kubectl rollout history deployment my-nginx -- revision=2**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.025.jpeg">

25. **Get the information on the deployments, pods, and replicas. Run: kubectl get deployments,replicaset,pods -l app=my-nginx**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.026.jpeg">

**Notice there are 2 replicasets, but only 1 replicaset is active. The inactive replicaset is for revision 1, and is there for roll back.**  

26. **Run   a rollback to revision 1 of deployment my-nginx.**  

**List the revision history. Run: kubectl rollout history deployment my-nginx**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.027.png">

**To go back to revision 1, run: kubectl rollout undo deployment my-nginx –to-revision=1  Run the kubectl rollout undo deployment my-nginx --to-revision=1**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.028.png">

27. **List the history. Run: kubectl rollout history deployment my-nginx**  

**Notice there is no longer a revision 1, but the current revision is revision 3. Revision 1, is now revision 3.**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.029.png">

28. **Get the rollout history for the current deployment nginx, and see the image for the deployment. The image will be nginx, version 1.17.10, which was the original version before the rolling update.**  

**Run: kubectl rollout history deployment my-nginx --revision=3**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.030.jpeg">

29. **List the deployments, replicasets and pods for application my-nginx.  Run: kubectl get deployments,replicasets,pods -l app=my-nginx**  

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.031.png">

**Which replicaset is active now?**  

30. **The application my-nginx is still running** 

<img src="/images/posts/picture/kubernetes/Aspose.Words.0072a41e-5a88-4a85-8849-2604e9b99695.032.jpeg">
