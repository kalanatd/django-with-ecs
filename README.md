# django-with-ecs

A Simple Django app hosted on ECS.

Steps:

- CF Script to run Jenkins Master (Inside EC2, Service is executed using the official Jenkins LTS Docker)
- Dockerfiles for the Django app. 
- CF Script to host the Django app as an ECS Service.
- Jenkins Pipeline script to trigger the aforementioned CF Script.

