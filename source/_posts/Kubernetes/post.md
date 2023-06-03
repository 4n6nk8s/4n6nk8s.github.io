---
title: Streamlined Application Deployemet using Kubernetes
date: 2023-06-03 17:38:55
tags:
cover: https://imgur.com/CL8Xz9l.png
categories:
- [Kubernetes]
---

# Introduction 

As second-year engineering students, we embarked our academic group project that pushes us beyond our limits. Being self-proclaimed cloud-native and Kubernetes geeks, we decided to venture into an unfamiliar territory with an advanced project. Our goal? To unravel the mysteries of Heroku, Vercel, and On-Render, the leading PaaS solutions! And here is the twist, we were running out of time, and neither ChatGPT nor Google can help us. We are mad, and we actually did it in 4 days! Fellow adventurers, let's get you to feel the glimpse of what we have been through in just 4 days to build our project codenamed Kli8nt for streamlined application deployments into the Kubernetes realm



Fueled by the increasing demand for a unified platform that encompasses front-end hosting, backend deployment, and database management, our team leveraged our expertise in DevOps, Kubernetes, and web development to establish a robust and scalable solution that is client-friendly and K8s-able, hence the name Kli8nt. It is our way to design a mechanism that's a proof-of-concept to the leading PaaS solutions

# Project Details  : 

## High Level Architecture

Our system consists of various interconnected components that work together seamlessly. At the front-end, users interact with the system and provide the necessary information for application deployment. The backbone of our system is the backend, which handles authentication, data storage, and serves as the communication hub with the clusters. It establishes a websocket connection to deliver real-time logs to the user. To facilitate efficient communication between the backend and clusters, we rely on RabbitMQ, a messaging system that ensures reliable information exchange. The clusters themselves take charge of the build and deployment operations, guided by custom controllers to ensure efficiency and accuracy. Lastly, Kafka, our robust data streaming platform, stores and streams application logs from the clusters to the user. Together, these components form a cohesive system that enables smooth and efficient application deployment.





![](https://i.imgur.com/fEOyFeQ.png)
## User Interface 

The User Interface serves as the primary interaction point for developers using the
Kli8nt platform. Designed with simplicity and ease of use in mind, the UI provides
a seamless experience for users to connect to their GitHub accounts, manage their
repository specifications, and initiate deployments. We aim to streamline the
application management process and empower developers to effortlessly deploy
their applications.

![](https://imgur.com/O5fW0GG.png)

## User Authentication 

Kli8nt leverages the robust authentication capabilities of GitHub OAuth2 as its primary method for user login and authorization. By utilizing this powerful authentication mechanism, users can grant the platform access to their repositories. With the generated Personal Access Token (PAT) associated with the authenticated user, Kli8nt gains the ability to clone both private and public repositories, utilizing the functionalities provided by the GitHub API. This approach ensures secure and seamless integration with GitHub, allowing users to enjoy a streamlined experience while maintaining the necessary permissions and access controls.

![](https://imgur.com/KEoSXvr.png)

## Backend Service 


The backend service is the backbone of the platform, ensuring efficient functionality by handling user requests, processing deployment tasks, managing logs, and facilitating seamless communication between components. It plays a vital role in request handling and routing, ensuring user requests are directed to the appropriate components for further processing. Additionally, it handles user authentication and authorization, ensuring secure access to the platform. Integration with RabbitMQ allows for the handling and queuing of deployment requests, while task scheduling ensures efficient and ordered processing. The backend service also integrates with GitHub's Deployment Environment, facilitating synchronization and updates for deployments. Logs are streamed and stored using Kafka, with a backend endpoint responsible for streaming logs back to users for monitoring. Database interaction with PostgreSQL allows for data storage and retrieval related to user configurations, deployment records, and other relevant information, ensuring data consistency and integrity through GORM. Overall, the backend service acts as a crucial orchestrator, ensuring the smooth operation and effective management of the platform's core functionalities.

![](https://i.imgur.com/WyYpyTw.png)

## The Messaging Queue System

To manage and queue deployment requests, we’ve integrated RabbitMQ. Simply put, it is a software where queues are defined, to which applications connect in order to transfer messages. For when a deployment request is received, the backend enqueues the task into a BUILD queue to the Kubernetes BUILD Operator. This ensures that each task is processed in the right order. The Kubernetes DEPLOY Operator in turn enqueus deployment results back here though the INFORM queue.

![](https://i.imgur.com/zNOA4Lz.png)

## The Build and Deploy Kubernetes Custom Controller

The clusters are responsible for the build and deployment operations. These processes are managed by custom controllers, ensuring efficiency and accuracy in deploying applications.

![](https://i.imgur.com/kTZmlxJ.png)

## The Build Kubernetes Custom Controller 

The build controller plays a critical role in initiating the build process for each application. It acts as a listener for the queue served by RabbitMQ, enabling it to initiate the build process. When a new application is added to the queue, the controller creates an isolated environment specifically for that application. Within this isolated environment, it launches a dedicated build process. This process involves tasks such as cloning the repository and generating a Dockerfile based on the application’s stack and programming language, starting the Kaniko process to initiate the build, which includes compiling the source code, resolving dependencies, and creating a container image then pushing the resulting container image to a private registry.


After completing the build and push process, the isolated environment is terminated and deleted, in addition, the controller publishes a message to the deployment queue served by RabbitMQ. This queue holds the applications that are scheduled to be deployed by the deploy controller. This ensures a clean and efficient workflow, as each build process operates in its isolated context. By leveraging the build controller’s capabilities, applications can be built and prepared for deployment without interference or conflicts with other ongoing builds

## The Deploy Kubernetes Custom Controller 

After the build process concludes and the build controller sends an element to the queue, the deploy controller consumes the queue and initiates the deployment process. This involves creating a deployment object that includes a pod. The application’s Docker image is then pulled and run within this pod. Simultaneously, a cluster IP service is created and associated with the deployment, enabling internal communication within the cluster.

The deploy controller takes charge of exposing the application externally and adding a TLS/SSL certificate. It achieves this by adding a route rule for the service within the ingress configuration. Additionally, it adds TLS/SSL configuration to the Ingress, allowing the application to be accessed securely via HTTPS. By utilizing Nginx Ingress, the controller ensures that the application is reachable and accessible with the desired HTTPS protocol and has a specific subdomain for the application.

Once the application is successfully deployed and exposed to the TLS certificate, the deploy controller notifies the backend server that the application is now ready for user access. This communication informs the backend server that the newly deployed application is available and can be utilized by users.

## The Log Streaming Controller

Another important aspect in such provided service, is that the user would like to have a vision over his application while and after its being prepared and sent to production. To deliver such a feature, we have implemented another controller in go, that’s responsible for listening to Pod’s lifecycle events, made possible thanks to the Shared Informer feature in the Kubernetes API which is a local cache kind of resource made to be consumed. Whenever a new Deployment is up and running, grab its logs and stream them real time to another environment to be stored. This environment being Apache Kafka, an open source distributed message streaming that’s suited for high performance data pipelines. Our Kafka is running in its own managed cluster at Confluent, a KaaS provider with additional cool features that we liked while using them (like a nice user-friendly dashboard, a built-in REST API, etc..). Kafka is going to stream the logs, each application in its own kafka topic or event, directly to the backend, and store them for future requests -so you don’t really have to be there at the right time, whenever you request them you’ll get them and if you are there already you get them live. 

![](https://i.imgur.com/UomqhLd.png)


# The conclusion
Our end-of-year project theoretically proposes and empirically validate a system design able to streamline the process of deploying any mciroservice application living in Github to Kubernetes on Cloud. This project is our gateway to our graduation projects as well as internship opportunities this summer, and would be a great addition to our professional journeys

# Meet the Team 

Along with the immense support from our supervisor Mr. Bassem Ben Saleh and his esteemed jury.
We would like to extend a heartfelt thank you to our families and close ones for their support and encouragement during late nights and long hours of work.

{% links %}
- site: Mohamed Rafraf - Raf²
  url: https://mohamed-rafraf.github.io
  desc: Kubernetes Engineer
  image: https://media.licdn.com/dms/image/C4D03AQGoi9OLQy5yfg/profile-displayphoto-shrink_800_800/0/1635769397295?e=1691020800&v=beta&t=IOYj31PodnHo-vWlY3lHvfLTCAzBeGPclWqCkdQl1bE
  color: "#2296fd"
- site: Adam Lahbib - Adm
  desc: Backend Engineer / DevOps
  url: https://adamlahbib.me
  image: https://media.licdn.com/dms/image/D4D03AQEeg7KDib8qCQ/profile-displayphoto-shrink_800_800/0/1682525314941?e=1691020800&v=beta&t=O-T0VjepTffxrdxGNc4jwwPleeh-rt6RWlMF6gol1VQ
  color: "#de2336"
- site: Med Sofiene Barka - NieMand
  desc: Software Engineer 
  url: https://www.hmlndr.me/
  image: https://pbs.twimg.com/profile_images/1607462313182756867/WHEAVH9d_400x400.jpg
  color: "#02bf1b"
- site: Skander Soltane - RedNax
  desc: Software Developer
  url: https://www.linkedin.com/in/skander-soltane
  image: https://media.licdn.com/dms/image/D4D03AQGD3BI1PkShag/profile-displayphoto-shrink_800_800/0/1676590747987?e=1691020800&v=beta&t=Jo1SN1nqWZCCeJ7KyxP1f6NgWUPw4DyMGmkXPQrYvWg
  color: "#ff0378"
  
- site:  Dr. Bassem Ben Salah
  desc: Our Supervisor
  url: https://www.linkedin.com/in/bassem-ben-salah-3487a259/
  image: https://media.licdn.com/dms/image/C4D03AQEuhp7TFHjE3Q/profile-displayphoto-shrink_800_800/0/1549533675903?e=1691020800&v=beta&t=AuBIN0Obuyu-dqcWr2HWZMS9qJNZKzAJiw5KnuVOLSg
  color: "#de2336"
{% endlinks %}
