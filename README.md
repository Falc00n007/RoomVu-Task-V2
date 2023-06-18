![Logo](https://www.roomvu.com/_next/image?url=%2Fimages%2Flanding%2Fnew-homepage%2Flogo.svg&w=3840&q=50)

# DevOps Project  sample task for RoomVu.


# Task
* Deploy a multi-container application on Kubernetes using Docker. The application consists of three services: a web server, a Laravel base project, and a database. The web server is nginx,and the database is built using MySQL.

* A basic example Laravel project (https://laravel.com/docs/10.x/installation#your-first-laravel-project).

* A Mysql database.
* An Nginx server as a web server.

## Requirements

* Create a Kubernetes cluster using any cloud provider or local environment.
* Create a Dockerfile for the nginx web server that serves as an entry point for applications.
* Create a Dockerfile for the FPM that installs and copies the application code into the container.
* Create a Dockerfile for the database that installs MySQL.
* Use Kubernetes deployment and service objects to deploy the application.
* Use Kubernetes' ingress object to expose the application to the internet.
* Test the deployment by using health check commands and verifying that all services are running correctly and are healthy.
* Run laravel migration after each deployment to update the database structure.

# What has been done :

* **sub task 1:** Create a Kubernetes cluster using any cloud provider or local environment.

In my case, here's a general overview  to create a Kubernetes cluster on my local machine using Minikube or MicroK8s, Because I did not have access to any cloud server or  colocation  Server and I had limited resources i my Core i7 laptop, Anyway, my method went ahead and was tested.

My personal infrastructure build requirements has two main k8s simulation tools and I used them.

**Minikube:**

* I'd Installed Minikube on my local machine.

* Start the Minikube cluster.

* I'd verified the status of the cluster using the command kubectl cluster-info. yes it was up and running.

* I'd deployed applications to the cluster using YAML files. but on real production I suggest to use pre-prepared related project Helm  
  charts. because it's so fast And usable to develop and maintain more professional production.

* Access the deployed applications through their corresponding services. in this case i just used kubernetes dashboard.

**MicroK8s:**

* I'd Installed MicroK8s on your local machine.

* I'd Initialized the MicroK8s cluster using the command microk8s enabled.

* I'd Verify the status of the cluster using the command microk8s kubectl cluster-info.

* I'd deployed applications to the cluster using YAML files.About the helm, I suggest the same as the previous explanation above.

* I'd Accessed the deployed applications through their corresponding services.

* **Please** note that these are just general steps and the specific commands and configurations may vary depending on your operating system and other factors. It's also important to ensure that your machine meets the minimum requirements for running a Kubernetes cluster, such as sufficient CPU and memory resources. I had memory problems. But if you launch in a cloud service or a dedicated server, you will have no problem. Be sure to let me know if you have any installation issues.

# Current structure and test project files:
<pre>
.
├── **apache-fpm**
│   └── var
│       └── www
│           └── html
├── composer.json
├── Dockerfile
├── **mysql**
│   ├── factories
│   │   └── UserFactory.php
│   ├── migrations
│   │   ├── 2014_10_12_000000_create_users_table.php
│   │   ├── 2014_10_12_100000_create_password_resets_table.php
│   │   ├── 2014_10_12_100000_create_password_reset_tokens_table.php
│   │   ├── 2019_08_19_000000_create_failed_jobs_table.php
│   │   └── 2019_12_14_000001_create_personal_access_tokens_table.php
│   └── seeders
│       └── DatabaseSeeder.php
├── **nginx**
│   └── nginx.conf
├── **readme.md**
├── **roomvu-deploy-cheat.md**
└── **roomvu-deployment.yaml**

**10 directories, 13 files**
</pre>

# Docker file and Create image:

This Dockerfile is used to a Docker image for a Laravel 10 application. Here's a breakdown of each section:

<pre>
# Use the official PHP image as the base image
FROM php:8.1-apache
</pre>

This line specifies the base image for our Docker container, which is the official PHP 8.1 Apache image.

<pre>
# Install the required packages and extensions
RUN apt-get update \
    && apt-get install -y git zip unzip libpng-dev libonig-dev libxml2-dev \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd \
    && pecl install redis \
    && docker-php-ext-enable redis
</pre>

This section installs the required packages and extensions needed to run the Laravel application. It updates the package repository, installs some dependencies like Git, Zip, and Unzip, then installs PHP extensions like pdo_mysql, mbstring, exif, pcntl, bcmath, and gd. Finally, it installs Redis via PECL (a PHP extension installer) and enables its inclusion.

<pre>
# Copy the application code to the container
COPY . /var/www/html
</pre>

This line copies all files in the current directory (denoted by `.`) to the `/var/www/html` directory inside the container.
<pre>
# Set the document root
ENV APACHE_DOCUMENT_ROOT /var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
</pre>

This section sets the document root for the Apache web server running inside the container to `/var/www/html/public`. It then uses `sed` to replace instances of `/var/www/html` and `/var/www/` with `${APACHE_DOCUMENT_ROOT}` in the Apache server configuration files.

<pre>
# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
</pre>

This line installs Composer, a dependency manager for PHP, into the container.

<pre>
# Install dependencies
RUN composer install --no-interaction --no-scripts --no-progress --prefer-dist
</pre>

This section installs the application's dependencies using Composer.
<pre>
# Set the working directory
WORKDIR /var/www/html
</pre>

This line sets the working directory to `/var/www/html`, which is where the Laravel application resides inside the container.
<pre>
# Expose port 80
EXPOSE 80
</pre>

This line exposes port 80, which is the default HTTP port, so that it can be accessed from outside the container.
<pre>
# Start Apache
CMD ["apache2-foreground"]
</pre>

This line starts the Apache web server 8.2 for laravel 10 in the foreground when the container is launched on kubernetes cluster.


To **create a Docker image** from Dockerfile, the **docker build command** with the appropriate options. Here's syntax:

<pre>
docker build -t roomvu-app.
</pre>


# Registry and project image container transfer and preparation:

Due to problems accessing the public services registry from Iran and problems like this,
I used Docker registry 2.0 on my machine for storing and distributing my own Docker images.

* **what is the docerk registry 2.0 ? **
Docker Registry 2.0 is a tool that is used to store and distribute Docker images. It allows you to create and manage your own private registry, where you can store and share Docker images within your organization or with other trusted users.

* how to use it, on my **local machine** ?

**Run a local registry: Quick Version**
<pre>
$ docker run -d -p 5000:5000 --restart always --name registry registry:2
</pre>

* So, i push roomvu-app docker images to my local registery runed bove on localhost and exposed port 5000 like this command:

<pre>
$ docker tag roomvu-app localhost:5000/roomvu-app

$ docker push localhost:5000/roomvu-app
</pre>

In this case, when you use **docker images** command, you can see your application images added to local images repositye with 
Prefix like this : 

<pre>
REPOSITORY                     TAG                 IMAGE ID            CREATED               SIZE
localhost:5000/roomvu-app      latest              7e0aa2d69a15        3 minutes ago         795MB
</pre>


# How To Deployment roomvu-app on kubernetes ? 

* To create and Deployment roomvu-app on kubernetes cluster, you can use **roomvu-deployment.yaml**.

At the first, let's to introduce what is the file content for!


This YAML file contains several Kubernetes manifests for deploying and managing a Roomvu application. Let's break it down line by line:
<pre>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: roomvu-app
  labels:
    app: roomvu-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: roomvu-app
  template:
    metadata:
      labels:
        app: roomvu-app
    spec:
</pre>
The first part specifies the API version and kind of Kubernetes object we're creating, which is a deployment in this case. The **metadata** section provides a name for the deployment and labels to identify it. The **spec** section includes information about how many replicas of the application to create and how to select them using label selectors. Finally, the template section defines the pod **template** that will be used to run the application.

<pre>
      containers:
      - name: roomvu-app
        image: localhost:5000/roomvu-app:latest
        ports:
        - containerPort: 80
        env:
        - name: DB_HOST
          value: mysql
        - name: DB_PORT
          value: "3306"
        - name: DB_DATABASE
          value: my_database
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
</pre>

This section specifies the **containers** to run in the pod template. In this case, there is only one container named **roomvu-app**. It uses an image hosted on **localhost:5000**, exposes port **80**, and sets environment variables for the database connection details, which are sourced from a Kubernetes secret named **mysql-secret**.

<pre>
---
# mysql persistent volume yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/mysql
</pre>

This is a manifest for a Kubernetes **PersistentVolume**, which is a way to store data outside of a container's ephemeral filesystem. This particular volume uses a host path at **/data/mysql** and provides 2GB of storage.
<pre>
---
# mysql persistent volume claim yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  selector:
    matchLabels:
      app: mysql
</pre>

This manifest creates a **PersistentVolumeClaim** that will bind to the previously defined **PersistentVolume**. It specifies that it needs read/write access and 2GB of storage, and selects the **PersistentVolume** using a label match.

<pre>
---

# Services manifest are used with [ load balancing, Port Mapping, External Access ]

apiVersion: v1
kind: Service
metadata:
  name: roomvu-app
  labels:
    app: roomvu-app
spec:
  ports:
  - name: http
    port: 80
    targetPort: 80
  selector:
    app: roomvu-app
</pre>

This is a **Service** manifest, which provides network connectivity to pods running in a deployment. This service exposes port **80** and targets the **roomvu-app** deployment using label selectors.

<pre>
---
# mysql-secret yaml

apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  username: cm9vbXZ1Og==
  password: cm9vbXZ1Og==

</pre>

This manifest creates a Kubernetes **Secret** object named **mysql-secret**, which is used to store sensitive data like passwords. The encoded values for **username** and **password** are included in the **data** section.

<pre>
---
# The ingress manifest is define to expose your Laravel 10 application to external clients.

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roomvu-app
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: roomvu-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: roomvu-app
            port:
              name: http
</pre>

This manifest creates an **Ingress** resource, which exposes the **roomvu-app** deployment to external clients through a URL. This particular In use.

# how to check the task is done!

To browse the web application named "roomvu-app" that is clustered on Kubernetes and **exposed** on port **80**, you first need to find the IP address of the ingress controller. Based on me, it sounds like the ingress controller is deployed on a Kubernetes cluster within my local network IP range of **192.168.122.0/24**.

Here's command to list all running Kubernetes pods:

<pre>
kubectl get pods -n <namespace>
</pre>

Replace **<namespace>** with the namespace where the Roomvu app is deployed. If you're not sure what namespace to use, try **kubectl get namespaces** to list all available namespaces.

Once you have the name of the ingress controller pod, you can get its IP address using the following command:

<pre>
kubectl get pod <pod-name> -n <namespace> -o jsonpath="{.status.hostIP}"
</pre>

Replace **<pod-name>** and **<namespace>** with the actual pod name and namespace where the ingress controller is deployed.

Now that you have the IP address of the ingress controller, you can open a web browser and navigate to the IP address to access the Roomvu app. For example, my IP address of the ingress controller is **192.168.122.10**, you would enter **http://192.168.122.10** in your web browser.

Assuming the ingress controller has been properly configured with an Ingress resource for the Roomvu app, you should be able to access it at the root URL or the path specified in the Ingress rules. For example, if the Ingress rule specifies **/roomvu** as the path for the Roomvu app, you would enter **http://192.168.122.10/roomvu** in your web browser to access the app.

I hope this helps! Let me know if you have any further questions.

call me on +98 939 63 10 462 or mail me  farshidrahimi.ca@gmail.com

* **Final and important explanation**

**This is changed and customized and named version 2, Version 2 of this product has the same features and functionality as Version 1, but with improved flexibility for customization thanks to the separation of manifest files.

This change will make it easier for DovOps guys to tailor the product to their specific needs. The development and implementation steps for Version 2 remain the same as in previous versions, leveraging the expertise of DevOps guys.**


