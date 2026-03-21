Design, develop and deploy & operate a Containerized app using Aws ECs, ECR,connected to aws managed a database which following the security .
Task :
Task-1 Application ECR.
Task-2 Deploy using ecs.
Task-3 Deploy Same app using EKS
Task-4 Database integraton with application
     Amazon ECS task definitions use container images to launch containers on the container instances in your clusters. In this section, you create a Docker image of a simple web application,
     and test it on your local system or Amazon EC2 instance, and then push the image to the Amazon ECR container registry so you can use it in an Amazon ECS task definition.
                  * Install the Docker
                  . sudo yum update -y
                  . sudo yum install docker
                  . sudo service docker start
                  * Create a Docker image
                  . touch Dockerfile  
                  . nano Dockerfile
***Dockerfile :**  
FROM public.ecr.aws/amazonlinux/amazonlinux:latest
RUN yum update -y && \
yum install -y httpd
RUN echo 'Hello World!' > /var/www/html/index.html
RUN echo 'mkdir -p /var/run/httpd' >> /root/run_apache.sh && \
 echo 'mkdir -p /var/lock/httpd' >> /root/run_apache.sh && \
 echo '/usr/sbin/httpd -D FOREGROUND' >> /root/run_apache.sh && \
 chmod 755 /root/run_apache.sh
EXPOSE 80
CMD /root/run_apache.sh
               . docker build -t hello-world .
               . docker images --filter reference=hello-world
               . docker run -t -i -p 80:80 hello-world

**Push image to Amazon Elastic Container Registry**
          * aws ecr create-repository --repository-name hello-repository --region us-east-1
<img width="740" height="350" alt="Screenshot 2026-03-17 171911" src="https://github.com/user-attachments/assets/0b565f9c-deec-4626-a670-685dc27e4cc2" />
          * docker tag hello-world 863942760608.dkr.ecr.us-east-1.amazonaws.com/hello-repository
          * aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 863942760608.dkr.ecr.us-east-1.amazonaws.com
          * docker push 863942760608.dkr.ecr.us-east-1.amazonaws.com/hello-repository
            
