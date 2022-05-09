# django-with-ecs

A Simple Django app hosted on ECS.

Steps:

- CF Script to run Jenkins Master (Inside EC2, Service is executed using the official Jenkins LTS Docker).
- Django application Components.
- Dockerfiles for the Django app.
- CF Script to host the Django app as an ECS Service.
- Jenkins Pipeline script to trigger the aforementioned CF Script.


The Procedure:

1. The Jenkins pipeline will be triggered when there is a new change in the GH repo. 
2. Pipeline will first build the Docker Image using the Dockerfile. And the push the image to the pre-configured Dockerhub repo.
3. Then the pipeline will run the Cloudformation script which will host the Docker application inside an ECS cluster fronted by an ALB.

