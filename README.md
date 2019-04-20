https://linuxacademy.com/blog/amazon-web-services-2/deploying-a-containerized-flask-application-with-aws-ecs-and-docker/

LEARN BY DOING FOR JUST $299
Get the highest rated, #1 Cloud Training Platform for only $299 per year! Limited Time Offer.
Save 33% Now 
Linux Academy
 RSS Feed Twitter Facebook LinkedIn Email
Features Courses Login Join Now!

Deploying a Containerized Flask Application with AWS ECS and Docker
 Posted by thaslett, Posted onJune 5, 2018
Containers are popular these days, with good reason. Let’s take fifteen minutes and find out why by deploying our own application using Docker, AWS, and Flask (a Python microframework used for building web applications).

Note: Some of this post will assume you’re using a Linux/Mac system.

Step 1 – Prerequisites

Let’s make sure we have everything we need to make this work:

An AWS Account.
The AWS CLI Installed and Configured (with enough permissions to take all the actions we need to take).
Docker (Docker commands may require sudo, or you can add your current user to the docker group).
The code for this project. You can run git clone https://github.com/linuxacademy/cda-2018-flask-app.git && cd cda-2018-flask-app && git checkout ecs-master
(Make sure you change to the other branch as this contains code specific to ECS)
Step 2 – Make Our Container Image

Start off with the command sudo docker image ls to review the current Docker setup. If this is a new Docker setup, this command probably won’t show much. If Docker has been installed for a while though, there may be some images hanging around already.

Next, let’s look at the Dockerfile in the repository with cat Dockerfile. Our Dockerfile is pretty straightforward. Let’s go over it line by line:

FROM python:3.5-slim – We’re starting with a Python 3.5 image as the base of our Docker image. This means we’re getting Python3.5 and it’s package manager, pip, for free before we make any customizations.
MAINTAINER fernando@linuxacademy.com – This line declares who the author of the image is. You can change it to your email. This format is no longer used, so you may also want change it to the non-deprecated LABEL instruction instead, like this: 
LABEL maintainer="fernando@linuxacademy.com"
USER root – This is telling us which user should be running the image and any commands, like what are in the CMD or RUN sections, later in the Dockerfile.
WORKDIR /app – This sets the working directory for some other commands, like what are in the CMD or RUN sections, that run later on in the Dockerfile.
ADD . /app – This copies all the files and directories from the current directory (specified with the .) to the /app directory.
RUN pip install --trusted-host pypi.python.org -r requirements.txt – This installs the requirements for our application using pip and the requirements.txt file provided.
EXPOSE 80 – This tells Docker which ports to listen on at runtime.
ENV NAME World – This sets an environment variable called NAME to the value of World.
CMD ["python", "app.py"] – This sets the command to be executed when running the image python app.py (the command required to run our Flask application)
Now that we understand exactly what’s in here, let’s make our new Docker image. We’ll build the docker image locally with: sudo docker build -t cda-flask-app .

Then let’s test our image. First, run curl localhost:80 just to make sure you’re not serving up anything locally. You didn’t get anything? Cool, that’s what we wanted.

Next, run this command to run your Docker image locally sudo docker run -p 80:80 cda-flask-app. Now either run this command again: curl localhost:80 or open up a web browser and enter localhost into your web browser’s URL bar.

You should now see something like this:



Yay! We have our site running locally in our new container image.

Step 3 – Send our image up to AWS

Now we’re going to send our newly created and tested image up to AWS.

Start by creating a repository in the Elastic Container Registry:

aws ecr create-repository --repository-name cda-penguin-app1123

Then run this to get a command you can use for logging in to the ECR repository you’ve just created:

aws ecr get-login --region us-east-1 --no-include-email

Then copy and paste the output of that command back into the terminal and run it. This will actually authenticate you with the Elastic Container Repository so you can push your Docker image into it.

Now that we’re logged into ECR, we can put our image into ECR. Let’s do this by:

Making sure we have our image locally by running docker image ls
Tagging the image by running docker tag cda-flask-app:latest ACCT_ID.dkr.ecr.us-east-1.amazonaws.com/ (where ACCT_ID is your own AWS account ID)
Pushing the image up to AWS ECR with docker push ACCT_ID.dkr.ecr.us-east-1.amazonaws.com/hello-world (again, using your own AWS account ID in place of ACCT_ID)
Boom! You should have a container image up in AWS.

Step 4 – Deploy our web application using ECS

The container image that we built locally is now up in the AWS cloud, inside of ECR. Since we’ve done this, we can use that same container image to deploy our application using Amazon ECS.

First, let’s make sure we can see the image we deployed inside of ECR. We do this by navigating to the ECR portion of the ECS console under Amazon ECR > Repositories. It should look something like this, though the repository name and URL for what you uploaded might be a little different.



This means that we have a container image for the application stored inside of AWS and we can now move on to ECS to get it running! Before you do though, let’s make sure to copy down the URI for the repository (we’ll need it shortly). In the AWS console, go to the ECS section and navigate to the Clusters tab. Then press the “Get Started” button.



On the next screen you should see this page:



This is allowing you to define a container and task definition for your new service. Lower on the same page, you’ll need to press “Configure” to set a custom container:



When configuring the container definition, we need to change the value in the box next to “Image*” to the URL of the image that we copied from the ECR page. This allows us to reference the ECR image in our container definition. My URL will probably look slightly different than yours because I gave my application and container a different name, and because our AWS account IDs will be different. We’ll also need to set a memory limit on this page – just use a Hard Limit of 128. Also, make sure to add a TCP port mapping on port 80 as shown in the image before pressing the Update button.



After this update, we should be ready to move on to the next step. We don’t need to change anything regarding the task definition, so just press the “Next” button.



On the next page we’ll have our Container and Task all defined, and won’t actually need to make any changes to the Service configuration either. If this was a production application, we would probably want to set up a load balancer. It’s not though, so, for now, we’ll just press the Next button.



Then we can finish things up by naming the cluster, if we want to, and pressing the Next button one more time.



Then we will land on a page where we can review our configuration before actually creating the new service. It should look something like this:



Further down on that page there should be a blue button called Create:



Once we press it, we should see AWS start creating a bunch of resources for the new service. Let this finish up and then press the “View service” button.



We should be on a new screen. If we’re ever lost, we can get to it again by going to the ECS Console, clicking on the correct cluster, and then clicking on the service. On this page, we click the Tasks tab.



Then from the tasks tab, we can click on the task that’s running for this new service.



Then click on the ENI ID link next to ENI Id:



This will redirect us to the AWS EC2 console which should show an instance running. We can grab the IPV4 IP address of that instance and paste it into a web browser.



Congratulations! We’ve just deployed a containerized application using AWS, Docker, ECS, and Flask!



Now there’s obviously a lot of this process we could configure or change to do more cool things. For example, we could:

Customize the Flask application so it does things other than displaying an adorable penguin website
Configure many of the other settings available when working with ECS, depending on the needs of your own applications
Deploy lots of cool containers with different requirements, other programming languages and much more
But this is certainly enough to get you started! If you want to view this demo in video form, and try it out in one of Linux Academy’s live environments, then sign up for Linux Academy and check out the AWS Certified Developer Associate Level Preparation Course, which contains a hands-on environment with this exact scenario!


AWS Labs hbspt.cta.load(3900131, ‘2e127b9e-4bde-437f-94ac-8e2742bbdeae’, {});
Share
43
Tweet
Share
43 SHARES
AMAZON WEB SERVICES DOCKER LINUX ACADEMY
jay abrams says:
November 26, 2018 at 8:08 pm
this is an incredible tutorial. thank you so much.

REPLY
Leave a Reply
Your email address will not be published. Required fields are marked *

Comment

Name *

Email *

Website


Save my name, email, and website in this browser for the next time I comment.

This site uses Akismet to reduce spam. Learn how your comment data is processed.

Get actionable training and tech advice
We'll email you our latest articles up to once per week.

Enter Email
Subscribe
Linux Academy
 RSS Feed Twitter Facebook LinkedIn Email
Features Courses Login Join Now!

https://427264915360.dkr.ecr.us-east-1.amazonaws.com
