# Coworking Analytics Service Deployment

This project deploys the coworking analytics application as a Docker container on Amazon EKS.  
The container image is built from the application source, tagged with a semantic version, and stored in Amazon ECR for deployment.  
Amazon CodeBuild serves as the CI component by automatically building and pushing a new image to ECR when repository changes are triggered.  
Kubernetes pulls the image from ECR and runs it as a Deployment exposed through a Service.  
The application uses a ConfigMap for non-sensitive environment variables and a Secret for the database password.  
PostgreSQL is exposed internally through a Kubernetes Service so the application can connect by service name instead of IP address.  
This design keeps configuration separate from code and makes the deployment easier to update across environments.  
CloudWatch Container Insights is used to review container logs and confirm that the application is running normally in the cluster.  
The Dockerfile uses a lightweight Python base image and includes comments for commands that perform setup beyond simple file copy or package installation.  
Semantic versioning such as `1.0.0`, `1.0.1`, and `1.1.0` makes releases easier to track, test, and roll back.  
To release a new build, a developer updates the code, creates a new image version through CodeBuild, and updates the Kubernetes manifest to reference the new tag.  
Applying the updated manifests triggers Kubernetes to roll the new version out in a controlled way.  
The `deployment/` directory contains the Kubernetes YAML files for the application Deployment, Services, ConfigMap, and Secret.  
Deployment health is validated with `kubectl get pods`, `kubectl get svc`, `kubectl describe deployment coworking`, and `kubectl describe svc postgresql-service`.  
A successful deployment shows ready pods, reachable services, and application logs without runtime or database errors.  
Reasonable CPU and memory requests and limits should be added so the pod can be scheduled predictably and avoid overusing cluster resources.  
A small general-purpose instance such as `t3.medium` is a practical choice for this application because it balances cost and performance for a lightweight containerized API.  
Costs can be reduced by using small node groups, cleaning up unused ECR images, shortening log retention, and shutting down nonproduction resources when they are not needed.  

## Stand-Out Suggestions

Reasonable CPU and memory requests and limits help Kubernetes schedule the pod reliably and prevent resource contention.  
A `t3.medium` is a good fit because it provides enough compute and memory for a small API workload without unnecessary cost.  
Costs can be saved by rightsizing nodes, removing unused images, reducing CloudWatch retention, and stopping idle environments.