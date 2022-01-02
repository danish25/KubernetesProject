# Architectural design of URL shortening service using Azure Cloud Platform

Please find the file ```URLShortner.png``` or ```URL_Shortening_Service.drawio``` for the proposed architecture of URL shortening service.


# KubernetesProject

## Create a Dockerfile:  
Refer to the Dockerfile in the repository

## Minikube:
Run the below command
```
   minikube start
```
## Build the docker image:  
Make sure we are running docker containers in Linux environment.  
Run the below command so that we are using docker instance from minikube:
```
@FOR /f "tokens=*" %i IN ('minikube -p minikube docker-env --shell cmd') DO @%i
```
Run the command in the same location where we have our docker fie:
```
   docker build -t mywebapp .
```
Run the image in Minikube:
```
Kubectl create deployment sample --image=mywebapp --port 8080
```
Run ``` kubectl get pod ``` to check the status of the container.  

If you are getting the error: ```ImagePullBackOff``` or ```ErrImagePull```, then follow the below steps:  
```Kubectl edit deployment sample``` 

A file will open. Scroll down and you will see a parameter ```ImagePullPolicy```.  

By default, it will be set to ```Always```. Change it to ```Never```, save the file and close the file.
Then check the status of the pod using ```kubectl get pod```.

## Use Persistent Volume:  
The file pvandpvc.yml contains the configuration of persistent volume and persistent volume claim.  

Run the below command:
   ```Kubectl apply -f pvandpvc.yml```
   
It will create the PV and PVC.

Now, we need to link this PV to the deployment that we have created.  
Run the command 
```
   Kubectl edit pod <PODNAME>
```
It will open the file in Notepad. 

In container section under spec section, add the below piece of code:
```
- mountPath: “/usr/local/tomcat/webapp”
  name: pvsample
 ```
 
In spec section, add the below lines of code:
```
Volumes:
- name: pvsample
  persistentVolumeClaim:
    claimName: pvcsample
```

Save the file and exit Notepad.  
Run the below command to check the status of the pod  
```
Kubectl get pod
```
## Create the service
Now, we need to create a service to provide the static IP address to the deployment that we just created.
To create the service, run the below command:  
```
Kubectl expose deployment/sample --type=NodePort
```

Now run the below command:  
```
Minikube service sample --url
```

The above command will provide a URL ```http://<IPADDRESS>:<PORT>```

Add ```/sample``` at the end of the above URL and browse the URL to see if the application is up or not. The URL will look like:
   ```
   http://<IPADDRESS>:<PORT>/sample
   ```
## Add a self-signed certificate to Minikube  
### Create a self signed certificate

Make sure that openssl is installed in your system. Run the below command to generate the certificate.   
```
openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout tls_self.key -out tls_self.crt -subj "/CN=*.${DOMAIN}" -days 365
```
### Add the certificate as a secret to Kubectl
```
kubectl create secret tls mysamplesecret --cert=tls_self.crt --key=tls_self.key
```
### Create Ingress
Use below command to get minikube IP Address:  
```
minikube ip
```
Use the above Ip address below.In, my case the IP address was 192.168.49.2   
**Note:** We are using nip.io for the CDN so that we do not have to modify the hosts file

Run the below command to create the ingress

```
kubectl create ingress sample --rule="sample.192.168.49.2.nip.io=sample:8080,tls=mysamplesecret"
```

Browse the URL ```https://sample.192.168.49.2.nip.io/sample``` and check if it is working.
