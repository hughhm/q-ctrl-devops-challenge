# q-ctrl-devops-challenge

Challenge - Create a website in GCP or AWS.

Chosen Providor: GCP (First time usage)

Time taken: 1.5 hrs

Intro 
I wanted to try something new with this challenge as i have never used GCP and so focused on the two more interesting deployment methods it offers: Google App Engine and GKE (Google Kubernetes Engine). For a website as simple as this a more straight approach may well be to Host a Static Website in Google Cloud with Cloud Storage (https://codelabs.developers.google.com/codelabs/cloud-webapp-hosting-gcs/index.html#0) or alternatively Google Compute Engine could be used but it would require more overhead than required at this stage due to minimum VM tiers. 

Google App Engine
1. Create a new project in GCP.
2. Create a new repo and include any code required for a php website in gcp (app.yaml, index.php).
3. Open in-browser cloud shell and clone the repo. 
4. Run: gcloud app create, select your desired region (australia southeast) and wait for the service to be created.
5. Run: gcloud app deploy and wait for the service to deploy.
6. Take note of the logging stream for later debugging - gcloud app logs tail -s default
7. Run: gcloud app browse to view the deployed website and confirm it was successful. 
8. Destroy the project if no longer needed - IAM & Admin > Settings > Shutdown (NOTE: This will delete the entire project and all resources within it)


Google Kubernetes Engine
1. Navigate to https://console.cloud.google.com/projectselector/kubernetes, login if required and create a new or select an existing project.

2. Make note of your GCP project id from the dashboard.
3. Create or use an existing repo.

4. Prepare your Dockerfile and add to repo - in this case i have used a template found at GKE/Dockerfile This will install and run PHP behind Apache on port 8080 and Copy the required php/html files to /var/www/html/.
5. Prepare relevant PHP and HTML files and add to repo.

6. Open Google cloud shell and git clone <repo url>, cd into the clone repo.
7. Set the project id for later use: export PROJECT_ID=<your-project-id>

8. Build and tag the Docker image: "docker build -t gcr.io/${PROJECT_ID}/<anynamehere>:v1 ."
9. Check the output for errors and verify the image has been built successfully: "docker images"

10. Authenticate with container registry and then push the image to it:
	- gcloud auth configure-docker
	- docker push gcr.io/${PROJECT_ID}/<anynamehere>:v1
	
11. Now that the image has been built and pushed to the container registry we need to prepare the GKE environment:
	- Set project id: gcloud config set project $PROJECT_ID
	- Set the compute region: gcloud config set compute/zone <your-compute-zone>
	- Build the cluster: gcloud container clusters create <yourclustername>  --num-nodes=2

12. Deploy the site to the cluster using kubernetes deployment:
	- "kubectl create deployment <yourdeployname> --image=gcr.io/${PROJECT_ID}/<anynamehere>:v1"
	- Set a baseline number of replicas (scale instances): "kubectl scale deployment <yourdeployname> --replicas=1"
	- Create a horizontal scale rule: "kubectl autoscale deployment <yourdeployname>  --cpu-percent=80 --min=1 --max=2"
	- Make the app publically available: "kubectl expose deployment <yourdeployname> --name=<yourdeployname>-service" --type=LoadBalancer --port 80 --target-port 8080
	- Confirm the service is active: "kubectl get service"
	
13. Should you wish to update the container down the line, follow instructions 8 and 10 to build and push the new container, then set the GKE to use it with the following:
 - "kubectl set image deployment/<yourdeployname> <anynamehere>=gcr.io/${PROJECT_ID}/<anynamehere>:v2"

Notes: This deployment is very basic and does not provide CI/CD, autoscaling, reliabiltiy, more advanced monitoring or a custom URL. Improvements could be made here by setting up a CI/CD pipeline with Cloud Build to ensure timely deployment and management of code changes, alerting via email or text could be added to ensure immediate notificaiton of website issues.  