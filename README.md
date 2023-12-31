# Java Test App CI/CD

## Overview
This repository is dedicated to building a simple Java application using Docker, Jenkins, Kubernetes (K8s), and ArgoCD. The CI pipeline, orchestrated by Jenkins, utilizes a Jenkinsfile to build the Docker image and push it to the Docker registry, tagged with the commit number. The CD pipeline, also orchestrated by Jenkins, involves a Jenkinsfile responsible for updating the image name in the Kubernetes (K8s) deployment. Subsequently, ArgoCD is triggered to manually build the application with the latest image version. This streamlined CI/CD process ensures efficient development, deployment, and orchestration of the Java application.

## Building Project with Docker

### Steps:

- Clone the repository to your local machine.

- The Dockerfile performs the following:

  A. Create Maven container:
     - Copy `pom.xml` to `/tmp`.
     - Copy folder "src" to `/tmp/src`.
     - Navigate to `/tmp` and run `mvn package`.
     - Generate `devopsarea-01.war`.

  B. Create Tomcat container:
     - Move the file `devopsarea-01.war` from the Maven container to `/webapp` in the Tomcat container.
     - Perform a health check to ensure successful artifact deployment.

- Run 
     ```
     docker build -t devopsarea .
     ```
   - Create a Docker image called `devopsarea`.

- Run 
     ```
     docker run -d -p 8080:8080 --name devopsarea-sample-java-app devopsarea
     ```
   - Create a container named `devopsarea-sample-java-app` and forward the container internal port 8080 to localhost:8080 on the host machine.

- Open in your browser to view the result.
     ```
     http://localhost:8080/
     ```
## Building Project with Jenkinsfile

### Preparation:

1. Run the `ngrok` tool to establish a tunnel between localhost and the public network.
2. Provide parameters to Jenkins:
   - GitHub username.
   - GitHub token.
   - Credential-ID = github.
   - DockerHub username.
   - DockerHub password.
   - Credential-ID = docker-cred.
3. Create a 'prod' namespace.

### Challenges Faced:

 **Using GitHub webhook with Jenkins locally:**
  - Utilized `ngrok` to create a tunnel between localhost and the public network.
  - Configured the webhook URL in GitHub to point to `ngrok` URL/github-webhook/.
  - Added the public key of the Jenkins user to GitHub deploy keys.
  - Enabled "GitHub hook trigger for GITScm polling" in the Jenkins job.

 **CI job triggers CD job:**
  - Maintained separate repositories to avoid infinite loops.

 **Naming Docker image with commit number:**
  - Stored commit number in a variable (`GIT_COMMIT_REV`) and passed it as a parameter to the CD job.

 **Installing ArgoCD as a Kubernetes pod in the 'ArgoCD' namespace.**

 **Artifact version update:**
  - Implemented a file (`artifact_version_update`) to update the deployment.yaml file with the latest image version containing the commit number.

### CI/CD Workflow:

1. Push changes to the GitHub repo (`Java-Test-app`).

2. GitHub webhook triggers the "Java-test-app" job, executing the Jenkinsfile.

3. CI Jenkinsfile stages:
   - Clone the git repo.
   - Build the Dockerfile and name the image `dockerhub-accountname/java-test-app:<commit-number>`.
   - Push the Docker image to the DockerHub Registry.
   - Trigger the "CD" job and pass the commit number as a parameter.

4. "CD" job Jenkinsfile:
   - Run `artifact_version_update` to change the deployment.yaml file with the newer image version containing the last commit number.
   - Push the updated deployment.yaml file to the GitHub repo (`Java-Test-app-CD`).

5. ArgoCD is triggered, deploying the application to the 'prod' namespace.

6. To obtain the EXTERNAL-IP, run the command:
   ```
   kubectl get svc -n prod.
   ```

7. Open in your browser and view the result.
     ```
     http://EXTERNAL-IP:8080/
     ``` 