

Task Overview

The goal of this task was to set up a complete CI/CD pipeline for the n8n application using Jenkins, Docker, and Kubernetes (with Helm).
The task includes automating the build and deployment of the n8n service, pushing the Docker image to Docker Hub, and exposing it to the internet using ngrok.

⸻

Steps Performed

1. Repository Setup
	•	Forked the official n8n repository to my own GitHub account.
	•	Launched an EC2 instance in AWS to use as a Jenkins server.
	•	Installed Docker, Jenkins, Helm, and kubectl on the EC2 server.
	•	Cloned the forked n8n repository into the Jenkins server.

⸻

2. Jenkins Pipeline Setup
	•	Created a Jenkinsfile in the repository for automated build and deployment.
	•	Configured Jenkins credentials for:
	•	GitHub access
	•	Docker Hub login
	•	Kubernetes cluster access K3s

Jenkins pipeline flow:
	1.	Pull the latest code from the GitHub repository.
	2.	Build a Docker image using the Dockerfile from the n8n project.
	3.	Push the image to Docker Hub (hanumath/n8n-task).
	4.	Deploy the updated image to the Kubernetes cluster using Helm.

⸻

3. Docker Build
	•	The n8n repository already has a working Dockerfile, which builds correctly without modification.
	•	The Jenkins pipeline uses this Dockerfile to create the image and push it to Docker Hub.
	•	The image was built successfully, and no major build issues were encountered.

⸻

4. Helm Chart Setup
	•	Created a custom Helm chart for n8n under helm/n8n/.
	•	The chart includes:
	•	Chart.yaml – chart information.
	•	values.yaml – defines the image repository, tag, and service ports.
	•	templates/deployment.yaml and templates/service.yaml – for deploying the container and exposing the service.
	•	The service type was set to NodePort to allow external access.
	•	Deployed the Helm chart using Jenkins pipeline successfully.

⸻

5. Deployment Verification
	•	After deployment, verified that the n8n pod and service were running properly in the Kubernetes cluster:

kubectl get pods -n default
kubectl get svc -n default


	•	The pod was in Running state, and the service was exposed through a NodePort.
	•	Used ngrok to expose the NodePort service publicly.
Example:

ngrok http 31001


	•	Verified that the n8n UI was accessible through the ngrok URL.
	•	Successfully logged in to the n8n interface and confirmed it was working fine.



⸻

Summary of Work Done
	1.	Forked the official n8n repository and cloned it to the EC2 server.
	2.	Installed and configured Jenkins with Docker and Helm.
	3.	Created and added the Jenkinsfile for CI/CD pipeline.
	4.	Built and pushed Docker image to Docker Hub using Jenkins.
	5.	Created and used Helm charts to deploy on Kubernetes.
	6.	Exposed the service through NodePort and ngrok.
	7.	Verified that the application is running fine and reachable via public URL.

URL: https://breanna-paresthetic-semiwildly.ngrok-free.dev/home/workflows

