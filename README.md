# Deploying a Flask API

This is the project starter repo for the course Server Deployment, Containerization, and Testing.

In this project you will containerize and deploy a Flask API to a Kubernetes cluster using Docker, AWS EKS, CodePipeline, and CodeBuild.

The Flask app that will be used for this project consists of a simple API with three endpoints:

- `GET '/'`: This is a simple health check, which returns the response 'Healthy'. 
- `POST '/auth'`: This takes a email and password as json arguments and returns a JWT based on a custom secret.
- `GET '/contents'`: This requires a valid JWT, and returns the un-encrpyted contents of that token. 

The app relies on a secret set as the environment variable `JWT_SECRET` to produce a JWT. The built-in Flask server is adequate for local development, but not production, so you will be using the production-ready [Gunicorn](https://gunicorn.org/) server when deploying the app.



## Prerequisites 

* Docker Desktop - Installation instructions for all OSes can be found <a href="https://docs.docker.com/install/" target="_blank">here</a>.
* Git: <a href="https://git-scm.com/downloads" target="_blank">Download and install Git</a> for your system. 
* Code editor: You can <a href="https://code.visualstudio.com/download" target="_blank">download and install VS code</a> here.
* AWS Account
* Python version between 3.7 and 3.9. Check the current version using:
```bash
#  Mac/Linux/Windows 
python --version
```
You can download a specific release version from <a href="https://www.python.org/downloads/" target="_blank">here</a>.

* Python package manager - PIP 19.x or higher. PIP is already installed in Python 3 >=3.4 downloaded from python.org . However, you can upgrade to a specific version, say 20.2.3, using the command:
```bash
#  Mac/Linux/Windows Check the current version
pip --version
# Mac/Linux
pip install --upgrade pip==20.2.3
# Windows
python -m pip install --upgrade pip==20.2.3
```
* Terminal
   * Mac/Linux users can use the default terminal.
   * Windows users can use either the GitBash terminal or WSL. 
* Command line utilities:
  * AWS CLI installed and configured using the `aws configure` command. Another important configuration is the region. Do not use the us-east-1 because the cluster creation may fails mostly in us-east-1. Let's change the default region to:
  ```bash
  aws configure set region us-east-2  
  ```
  Ensure to create all your resources in a single region. 
  * EKSCTL installed in your system. Follow the instructions [available here](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html#installing-eksctl) or <a href="https://eksctl.io/introduction/#installation" target="_blank">here</a> to download and install `eksctl` utility. 
  * The KUBECTL installed in your system. Installation instructions for kubectl can be found <a href="https://kubernetes.io/docs/tasks/tools/install-kubectl/" target="_blank">here</a>. 


## Initial setup

1. Fork the <a href="https://github.com/udacity/cd0157-Server-Deployment-and-Containerization" target="_blank">Server and Deployment Containerization Github repo</a> to your Github account.
1. Locally clone your forked version to begin working on the project.
```bash
git clone https://github.com/SudKul/cd0157-Server-Deployment-and-Containerization.git
cd cd0157-Server-Deployment-and-Containerization/
```
1. These are the files relevant for the current project:
```bash
.
├── Dockerfile 
├── README.md
├── aws-auth-patch.yml #ToDo
├── buildspec.yml      #ToDo
├── ci-cd-codepipeline.cfn.yml #ToDo
├── iam-role-policy.json  #ToDo
├── main.py
├── requirements.txt
├── simple_jwt_api.yml
├── test_main.py  #ToDo
└── trust.json     #ToDo 
```

     
## Project Steps

Completing the project involves several steps:

1. Write a Dockerfile for a simple Flask API
2. Build and test the container locally
3. Create an EKS cluster
4. Store a secret using AWS Parameter Store
5. Create a CodePipeline pipeline triggered by GitHub checkins
6. Create a CodeBuild stage which will build, test, and deploy your code

For more detail about each of these steps, see the project lesson.


## Local run/check
1. set JWT_SECRET and LOG_LEVEL in .env_file
- _JWT_SECRET_ - The text string to be used for creating the JWT.
    - _LOG_LEVEL_ - It represents the level of logging. It is optional to be set. It has a default value as 'INFO', but when debugging an app locally, you may want to set it to 'DEBUG'.
2. execute
```
docker run --env-file .env_file -p 8080:8080 simple-jwt-api
```

Check whether Flask service is available:
1. Open the link in a browser : "http://localhost"
It should return text "Healthy"

or  use:
curl.exe --request GET  http://localhost
Result: "Healthy"

2.  Data encryption to the token works fine:
 for /f "delims=" %i in ('curl.exe -s --data "{\"email\":\"abc@xyz.com\",\"password\":\"mypwd\"}" -H "Content-Type: application/json" -X POST http://localhost/auth ^| jq.exe -r ".token"') do set "TOKEN=%i"

Token was generated/data encrypted: 
set "TOKEN=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3OTQwNzQxODgsIm5iZiI6MTc5Mjg2NDU4OCwiZW1haWwiOiJhYmNAeHl6LmNvbSJ9.-11UHKSrNwts9tVmU7Hb4duvb8x3bZdPaKK-E4m7m8w"

3. Data decryption:

curl.exe --request GET "http://localhost/contents" -H "Authorization: Bearer %TOKEN%" | jq.exe .

  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100     58 100     58   0      0    182      0                              0
{
  "email": "abc@xyz.com",
  "exp": 1794074188,
  "nbf": 1792864588
}

## Remote run/check
1. Online Flask service is available:

curl.exe --request GET  afc60e59c23ff4b52b3f9de84df07317-116254502.us-east-1.elb.amazonaws.com
Result: "Healthy"

2.  Data encryption to the token works fine:
 for /f "delims=" %i in ('curl.exe -s --data "{\"email\":\"abc@xyz.com\",\"password\":\"mypwd\"}" -H "Content-Type: application/json" -X POST http://afc60e59c23ff4b52b3f9de84df07317-116254502.us-east-1.elb.amazonaws.com/auth ^| jq.exe -r ".token"') do set "TOKEN=%i"

Token was generated/data encrypted: 
set "TOKEN=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3OTQwNzQxODgsIm5iZiI6MTc5Mjg2NDU4OCwiZW1haWwiOiJhYmNAeHl6LmNvbSJ9.-11UHKSrNwts9tVmU7Hb4duvb8x3bZdPaKK-E4m7m8w"

3. Data decryption:

curl.exe --request GET "http://afc60e59c23ff4b52b3f9de84df07317-116254502.us-east-1.elb.amazonaws.com/contents" -H "Authorization: Bearer %TOKEN%" | jq.exe .

  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100     58 100     58   0      0    182      0                              0
{
  "email": "abc@xyz.com",
  "exp": 1794074188,
  "nbf": 1792864588
}

4. Check kubernetes nodes availability:
kubectl get nodes
NAME                             STATUS   ROLES    AGE   VERSION
ip-192-168-17-166.ec2.internal   Ready    <none>   53m   v1.36.5-eks-117f323
ip-192-168-42-131.ec2.internal   Ready    <none>   53m   v1.36.5-eks-117f323

5. Check pods availability:
kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
simple-jwt-api-5ccdccfd57-4bdwh   1/1     Running   0          27m
simple-jwt-api-5ccdccfd57-6z2kx   1/1     Running   0          27m
simple-jwt-api-5ccdccfd57-nqltl   1/1     Running   0          27m

6. Get svc:
kubectl get svc
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)        AGE
kubernetes       ClusterIP      10.100.0.1       <none>                                                                   443/TCP        61m
simple-jwt-api   LoadBalancer   10.100.239.159   afc60e59c23ff4b52b3f9de84df07317-116254502.us-east-1.elb.amazonaws.com   80:30774/TCP   42m