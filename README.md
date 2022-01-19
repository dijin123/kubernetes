# How to Deploy Docker Container to a Kubernetes using Docker Desktop in Windows

**Kubernetes Installation with Docker**

**Step 1: Install & Setup Hyper-V**
Windows as we all know, have their own virtualization software and it's called Hyper-V which is basically something like VirtualBox on steroids. Hyper-V can manage your virtual machines (VM) using the default GUI tool provided by Microsoft for free, or through command line. Enabling Hyper-V can be easily done but first, make sure your PC meets the following requirements: Your OS should be Windows 10 (Enterprise, Pro or Education) with minimum 4GB RAM and CPU Virtualization support, although you have to double check if it's enabled in your BIOS settings.

You can remove or add features that don't come up pre-installed during the installation of Windows, like Hyper-V. Always remember that some of the features require internet access to download extra components from Windows Update. Follow the next steps to enable Hyper-V on your machine.
1.	Go to Control Panel
2.	On your left panel, click on Programs
3.	Then click Programs and Features followed by Turn Windows features on and off.
4.	Check Hyper-V and Windows Hypervisor Platform
5.	Click OK

Your system will now start installing Hyper-V on the background, it may need to reboot a couple of times until everything is configured properly. Don't expect any notification or something! Run the following command as Administrator on powershell and verify if Hyper-V is installed successfully on your machine: 
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V

**Step 2: Install Docker for Windows**

Kubernetes is build on top of Docker, theoretically is just a tool that communicate with your Docker containers and manage everything at an enterprise level. To install Docker  Docker Desktop for Windows (stable) - Install Docker Desktop on Windows | Docker Documentation. Follow the setup's instructions and after you click Finish, Docker already setup a Linux virtual machine on top of Hyper-V and start all the necessary services without manually configure anything. After the installation, you can either "communicate" with Docker using the CLI tool docker or running HTTP requests though its API.

**Step 3: Install Kubernetes on Windows 10**
Docker comes with a handy GUI tool where you can modify some settings or install & enable Kubernetes. Follow the following instructions, WARNING! If Docker is installed successfully and you can't locate its tray icon, you have to reboot your system. If the problem persists, check the official troubleshooting guide here https://docs.docker.com/docker-for-windows/troubleshoot/. Now follow the instructions to install Kubernetes.
1.	Right-click the Docker tray icon
2.	Click "Settings"
3.	On the left panel click "Kubernetes"
4.	Check Enable Kubernetes and click "Apply"
During installation, Docker is going to install additional packages and dependencies. It may take around 5~10 minutes and the installation time depends on your Internet speed and your PC performance. Wait until the 'Installation complete!' popup message is shown up. After installing Kubernetes you can make sure that everything is working fine using the Docker app. If both services (Docker & Kubernetes) are running successfully without any errors then both icons at bottom left will go green.
Example.
 
**Step 4: Deploying the Dashboard UI**
The Dashboard UI is not deployed by default. To deploy it, run the following command:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml

**Step 5: Accessing the Dashboard UI **
To protect your cluster data, Dashboard deploys with a minimal RBAC configuration by default. Currently, Dashboard only supports logging in with a Bearer Token. Create a new user using the Service Account mechanism of Kubernetes, grant this user admin permissions and login to Dashboard using a bearer token tied to this user.

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

Use kubectl apply -f dashboard-adminuser.yaml to create them.

**Step 6: Getting a Bearer Token**
Now we need to find the token we can use to log in. Execute the following command:
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"

It should print something like:

eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQlitASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hwxznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1VLa6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabINX6gP5-Tuuw2svnV6NYQ

Now copy the token and paste it into on the login screen.

**Step 7: Command line proxy**

You can enable access to the Dashboard using the kubectl command-line tool, by running the following command:
kubectl proxy

Kubectl will make Dashboard available at
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

 
Click the Sign in button and that's it. You are now logged in as an admin.
 
**Clean up and next steps**
Remove the admin ServiceAccount and ClusterRoleBinding.

kubectl -n kubernetes-dashboard delete serviceaccount admin-user
kubectl -n kubernetes-dashboard delete clusterrolebinding admin-user

**
show deployment**
kubectl get deployments

NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
vfe-hello-wrold   1         1         1            1           14m

**delete deployment**
kubectl delete deployments vfe-hello-wrold

deployment.extensions "vfe-hello-wrold" deleted

**show services**
kubectl get services

NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
vfe-hello-wrold   NodePort    10.101.23.36     <none>        8083:31532/TCP   14m
  
**delete services**
kubectl delete service vfe-hello-wrold

service "vfe-hello-wrold" deleted


