# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD and Kubernetes

![image](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/2f165f4b-0e2c-4254-9b2f-be76aa6cfd8e)

The project involves building and deploying a Java application using a CI/CD pipeline. Here are the steps involved:

Version Control: The code is stored in a version control system such as Git, and hosted on GitHub. The code is organized into branches such as the main or development branch.

Continuous Integration: Jenkins is used as the CI server to build the application. Whenever there is a new code commit, Jenkins automatically pulls the code from GitHub, builds it using Maven, and runs automated tests. If the tests fail, the build is marked as failed and the team is notified.

Code Quality: SonarQube is used to analyze the code and report on code quality issues such as bugs, vulnerabilities, and code smells. The SonarQube analysis is triggered as part of the Jenkins build pipeline.

Containerization: Docker is used to containerizing the Java application. The Dockerfile is stored in the Git repository along with the source code. The Dockerfile specifies the environment and dependencies required to run the application.

Container Registry: The Docker image is pushed to DockerHub, a public or private Docker registry. The Docker image can be versioned and tagged for easy identification.

Continuous Deployment: ArgoCD is used to automate the deployment of the containerized application to Kubernetes. Whenever a new version of the application image is pushed to the Git repository, ArgoCD will automatically deploy it to the Kubernetes cluster.

Overall, this project demonstrates how to integrate various tools commonly used in software development to streamline the development process, improve code quality, and automate deployment.


Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:

# Setup an AWS EC2 Instance

Login to an AWS account using a user with admin privileges and ensure your region is set to ap-south-1a Mumbai region.
Move to the EC2 console. Click Launch Instance.
For name use Jenkins
![1](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/3df00fe1-67f0-425b-8c84-4a02b57ce3b2)

Select AMIs as Ubuntu and select Instance Type as t2.medium. Create new Key Pair and Create a new Security Group with traffic allowed from ssh, http and https.
![3](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/30063ccd-7256-4baf-8c1d-6e83b11a8db4)

# Run Java application on EC2
This step is optional. We want to see which application we wanted to deploy on the Kubernetes cluster.
This is a simple Spring Boot-based Java application that can be built using Maven.

```
git clone https://github.com/harikdevops/Jenkins-Zero-To-Hero.git
cd Jenkins-Zero-To-Hero/java-maven-sonar-argocd-helm-k8s/spring-boot-app
sudo apt update
sudo apt install maven
```
![4](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/12952a35-cb1c-4565-a5e9-325e2396d438)
```
mvn clean package
```
![6](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/a3db897a-0d17-4cdb-8479-78885d805de0)
```
mvn -v
```
![5](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/4ac8cdfd-ff79-4e14-baa3-5ccea54a6105)
'''
sudo apt update
sudo apt install docker.io
sudo usermod -aG docker ubuntu
sudo chmod 666 /var/run/docker.sock
sudo systemctl restart docker
docker build -t ultimate-cicd-pipeline:v1 .
'''
![7](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/0640d3cd-8762-42d1-bd69-6d5b89e13254)
```
docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1
```
![8](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/575f6d9b-7deb-4291-8dae-94f24b234f36)

Add Security inbound rule for port 8001
```
http://65.2.177.25:8001/
```
![9](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/52649cde-aeac-452e-8ee6-69b0d944215d)

Prerequisites:

   -  Java application code hosted on a Git repository
   -   Jenkins server
   -  Kubernetes cluster
   -  Helm package manager
   -  Argo CD

Steps:

    1. Install the necessary Jenkins plugins:
       1.1 Git plugin
       1.2 Maven Integration plugin
       1.3 Pipeline plugin
       1.4 Kubernetes Continuous Deploy plugin

    2. Create a new Jenkins pipeline:
       2.1 In Jenkins, create a new pipeline job and configure it with the Git repository URL for the Java application.
       2.2 Add a Jenkinsfile to the Git repository to define the pipeline stages.

    3. Define the pipeline stages:
        Stage 1: Checkout the source code from Git.
        Stage 2: Build the Java application using Maven.
        Stage 3: Run unit tests using JUnit and Mockito.
        Stage 4: Run SonarQube analysis to check the code quality.
        Stage 5: Package the application into a JAR file.
        Stage 6: Deploy the application to a test environment using Helm.
        Stage 7: Run user acceptance tests on the deployed application.
        Stage 8: Promote the application to a production environment using Argo CD.

    4. Configure Jenkins pipeline stages:
        Stage 1: Use the Git plugin to check out the source code from the Git repository.
        Stage 2: Use the Maven Integration plugin to build the Java application.
        Stage 3: Use the JUnit and Mockito plugins to run unit tests.
        Stage 4: Use the SonarQube plugin to analyze the code quality of the Java application.
        Stage 5: Use the Maven Integration plugin to package the application into a JAR file.
        Stage 6: Use the Kubernetes Continuous Deploy plugin to deploy the application to a test environment using Helm.
        Stage 7: Use a testing framework like Selenium to run user acceptance tests on the deployed application.
        Stage 8: Use Argo CD to promote the application to a production environment.

    5. Set up Argo CD:
        Install Argo CD on the Kubernetes cluster.
        Set up a Git repository for Argo CD to track the changes in the Helm charts and Kubernetes manifests.
        Create a Helm chart for the Java application that includes the Kubernetes manifests and Helm values.
        Add the Helm chart to the Git repository that Argo CD is tracking.

    6. Configure Jenkins pipeline to integrate with Argo CD:
       6.1 Add the Argo CD API token to Jenkins credentials.
       6.2 Update the Jenkins pipeline to include the Argo CD deployment stage.

    7. Run the Jenkins pipeline:
       7.1 Trigger the Jenkins pipeline to start the CI/CD process for the Java application.
       7.2 Monitor the pipeline stages and fix any issues that arise.

This end-to-end Jenkins pipeline will automate the entire CI/CD process for a Java application, from code checkout to production deployment, using popular tools like SonarQube, Argo CD, Helm, and Kubernetes.
