# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD and Kubernetes

![CICD drawio](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/62e15773-88af-4a92-baee-474e9fb75596)




The project involves building and deploying a Java application using a CI/CD pipeline. Here are the steps involved:

Version Control: The code is stored in a version control system such as Git, and hosted on GitHub. The code is organized into branches such as the main or development branch.

Continuous Integration: Jenkins is used as the CI server to build the application. Whenever there is a new code commit, Jenkins automatically pulls the code from GitHub, builds it using Maven, and runs automated tests. If the tests fail, the build is marked as failed and the team is notified.

Code Quality: SonarQube is used to analyze the code and report on code quality issues such as bugs, vulnerabilities, and code smells. The SonarQube analysis is triggered as part of the Jenkins build pipeline.

Containerization: Docker is used to containerizing the Java application. The Dockerfile is stored in the Git repository along with the source code. The Dockerfile specifies the environment and dependencies required to run the application.

Container Registry: The Docker image is pushed to DockerHub, a public or private Docker registry. The Docker image can be versioned and tagged for easy identification.

Continuous Deployment: ArgoCD is used to automate the deployment of the containerized application to Kubernetes. Whenever a new version of the application image is pushed to the Git repository, ArgoCD will automatically deploy it to the Kubernetes cluster.

Overall, this project demonstrates how to integrate various tools commonly used in software development to streamline the development process, improve code quality, and automate deployment.


Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:

#  Setup an AWS EC2 Instance

Login to an AWS account using a user with admin privileges and ensure your region is set to ap-south-1a Mumbai region.
Move to the EC2 console. Click Launch Instance.
For name use Jenkins

![1](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/98fff9f2-2590-48dd-9d43-0dee9fc16525)

Select AMIs as Ubuntu and select Instance Type as t2.medium. Create new Key Pair and Create a new Security Group with traffic allowed from ssh, http and https.

![3](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/729b2d4b-e2df-4d0f-b4ef-920f7e25ef2d)

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

```
sudo apt update
sudo apt install docker.io
sudo usermod -aG docker ubuntu
sudo chmod 666 /var/run/docker.sock
sudo systemctl restart docker
docker build -t ultimate-cicd-pipeline:v1 .
```

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


# Now we are going to deploy this application on Kubernetes by CICD pipeline.

# Continuous Integration
# 1. Install and Setup Jenkins

Step 1: Install Jenkins
Follow the steps for installing Jenkins on the EC2 instance -Jenkins.
Pre-Requisites:
 - Java (JDK)
### Run the below commands to install Java and Jenkins

Install Java
```
sudo apt update
sudo apt install openjdk-11-jre
```
Verify Java is Installed
```
java -version
```
Now, you can proceed with installing Jenkins
```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.


- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).
- 
![11](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/7eff3af6-578a-4b13-a259-0e7ea1841530)

After you login to Jenkins, - Run the command to copy the Jenkins Admin Password - sudo cat /var/lib/jenkins/secrets/initialAdminPassword - Enter the Administrator password

![12](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/26e2e44c-de65-476c-a69a-05454c299eeb)
![14](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/08935f0e-5a8d-4602-9210-3a336e57734c)

### Click on Install suggested plugins
![13](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/d4af563f-b979-409d-9e30-3716268b17bc)
### Wait for the Jenkins to Install suggested plugins
![16](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/00d16e83-a274-4af6-846a-c883a40fcf28)

Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]
![17](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/66247865-cea8-4110-aa5e-d7fb706cdd96)
![18](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/202cac49-31b9-4edc-9b0c-b2ce7444bda0)


Jenkins Installation is Successful. You can now starting using the Jenkins
![19](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/9f68e6ed-b040-4dd7-9e3b-dfa1b1fcce01)

## Next Steps
## 2. Configure a Sonar Server locally
SonarQube is used as part of the build process (Continuous Integration and Continuous Delivery) in all Java services to ensure high-quality code and remove bugs that can be found during static analysis.
### Configure a Sonar Server locally

```
sudo apt install unzip
sudo adduser sonarqube
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
sudo mv sonarqube-9.4.0.54424 /opt
sudo unzip *
sudo mv sonarqube-9.4.0.54424/ sonarqube
sudo chmod -R 755 /opt/sonarqube/
sudo chown -R sonarqube:sonarqube /opt/sonarqube/
sudo cd sonarqube/bin/linux-x86-64/
./sonar.sh start
```

Hurray !! Now you can access the `SonarQube Server` on `http://<ip-address>:9000` 
![20](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/134981d6-260f-4eaa-b18b-814b9ea912e0)

Enter Login as admin and password as admin.
Change with new password.
# Credential for SonarQube that we will use in Jenkins pipeline 
Go to the right-hand corner A then click on My Account ==> Security => Generate Tokens

![21](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/955d874c-674e-4f21-8162-cdc1db5be071)


## 3. Docker Slave Configuration

Run the below command to Install Docker

```
sudo apt update
sudo apt install docker.io
```
 
### Grant Jenkins user and Ubuntu user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

Once you are done with the above steps, it is better to restart Jenkins.

```
http://<ec2-instance-public-ip>:8080/restart
```

The docker agent configuration is now successful.

# 4. Create Credentials in Jenkins
# Step 1: Create Sonarqube Credentials in Jenkins
-Please keep the below credentials ID name (sonarqube, docker-cred, github) as it is. Else according to your credentials, you need to make changes in Jenkinsfile
Step 1: Credential for SonarQube
Go to Manage Jenkins ==> Crendentials ==> System ==> Global credentials
![22](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/2a0bf471-772d-4ffc-8c65-6e8f8083dea2)

# Step 2: Create DockerHub Credentials in Jenkins
- Setup Docker Hub Secret Text in Jenkins

You can set the docker credentials by going into -

Goto -> Jenkins -> Manage Jenkins -> Manage Credentials -> Stored scoped to jenkins -> global -> Add Credentials
![23](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/77d7c361-98fa-449d-866f-031901823d3c)

# Step 3: Create GitHub credentials in Jenkins
Goto GitHub ==> Setting ==> Developer Settings ==> Personal access tokens ==> Tokens(Classic) ==> Generate new token

![2](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/51fc23f8-f80a-46f9-a7c5-3025983a9d92)

![24](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/451871e1-b1a1-48e3-952b-04a7aba5bd34)


![25](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/79ffa36f-34ff-42c5-9c81-be6d0f6d9faa)


# Install the necessary Jenkins plugins
Goto Jenkins Dashboard \==> Manage Jenkins \==> Plugins \==> Available plugins

- Docker Pipeline
![26](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/ca3fa569-f28c-4080-b8d1-46a98d300f3d)

- SonarQube Scanner
![27](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/2e3f54e8-b9a0-497c-a4a2-9b496d1d5e63)

- GitHub Integration
  ![28](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/755456fd-d37e-4def-b9b5-bc03fe2b0c1c)

- Do Jenkins Restart
  ![29](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/0938cf99-4aeb-47cc-bc7d-d0e265b81580)

# Create a new Jenkins pipeline
 Github: https://github.com/harikdevops/Jenkins-Zero-To-Hero.git

Click on New Item. Select Pipeline and Enter an Item name.
![30](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/e6515885-b8d7-4577-8e23-b344a970e923)

![31](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/3292ce56-73a6-44ee-ba9e-f1ee5009bf19)

Select your repository where your Java application code is present. Make changes in Repository according to your DockerHub Id and GitHub Id in spring-boot-app/JenkinsFile and spring-boot-app-manifest/deployment.yml

![32](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/2e5cc563-9519-49ed-a1c9-14e52818fb91)

Specify the branch and JenkinsFile path accordingly.

![33](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/844877dd-abd9-4026-a9ed-0f95bb748210)


Update this URL in spring-boot-app/JenkinsFile with your SonarQube URL(EC2 server)
```
http://65.2.177.25:9000/
```
![34](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/bfbc6055-357a-4d8d-b582-300b8a236f12)

# Run Pipeline
![35](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/04440d3b-8756-4a8c-b358-7b85a43bf81b)

# deployement.yml file gets updated with the latest image.
![36](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/bfb6e0a1-ab06-45a6-8f38-a9f965c0636f)

# SonarQube Output will be like this.

![37](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/ea3784ad-d20c-456c-b7ae-f052da4b4bca)

# Check DockerHub, that a new image is created for your Java application.

![38](https://github.com/harikdevops/Jenkins-Zero-To-Hero/assets/142023175/af6b4a35-b7b7-4ad8-9320-ec5f9e120600)

This way, we completed CI ( Continuous Integration) Part. Java application is built, SonarQube completed static code analysis and the latest image is created, push to DockerHub and updated Manifest repository with the latest image.

# Continuous Delivery Part






ArgoCD is utilized in Kubernetes to establish a completely automated continuous delivery pipeline for the configuration of Kubernetes. This tool follows the GitOps approach and operates in a declarative manner to deliver Kubernetes deployments seamlessly.

Argo CD Setup
Argo CD is a very simple and efficient way to have declarative and version-controlled application deployments with its automatic monitoring and pulling of manifest changes in the Git repo, but it also has easy rollback and reverts to the previous state, not manually reverting every update in the cluster.
In this section, I provided a guide on achieving continuous deployment on Minikube. However, I understand that for Windows OS users, the process of setting up Virtual Box can be quite daunting, especially for those who are not familiar with the technology. Additionally, configuring Minikube and ArgoCD on a Virtual Box/EC2 instance can require more manual setup and configuration. Given these challenges, I would like to offer an alternative solution by explaining how to deploy the Java application on Amazon EKS. For that you need to refer to my blog - Continuous Delivery with Amazon EKS and ArgoCD



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
