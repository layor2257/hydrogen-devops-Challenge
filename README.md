# hydrogen-devops-chanllenge

## Tasks:

1. Provide steps to set up Nginx reverse proxy, as well as reasons for it.

2. There are two node pools in a Kubernetes cluster, which pool contains a single node. The computing resources for each node pool are 1GB Memory and 1 CPU, 3GB Memory, and 2 CPU. How will you deploy an application that requires 2.5 GB of Memory? Kubernetes manifest codes can be pushed to Github, documentation is appreciated

3. You have SSH access to an Ubuntu or Debian Linux server, The disk is full and you need to clean it up, what would you do and which specific commands should you run?

4. The following NodeJs app has been developed, and the team is requesting a complete deployment approach. You have a pair of ACCESS_KEY and SECRET_KEY of an AWS user.

NB: You can use Terraform, Ansible, Bash, etc. Code can be pushed in Github, documentation is appreciated.

```
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello from Hydrogen!')
})

app.listen(port, () => {
  console.log(Example app listening on port ${port})
})
```


## Solutions:

1. Firstly, I would like to define what a reverse proxy is.

   A reverse proxy is a server that sits between a client device and a backend server and directs client requests to the appropriate server.

   Nginx is a reverse proxy server that handles requests on behalf of the backend server instead of clients connecting directly to the backend server and then forwards the requests to 
   the appropriate backend server and returns the response back to the clients.

   These are the steps to set up an Nginx reverse proxy server:

   **Using the `apt` packaging system, we need to first update our local package index**

  ```
  sudo apt update
  sudo apt install nginx
  ```

 **Adjusting the Firewall**: Before testing Nginx, the firewall software needs to be configured to allow access to the service. Nginx registers itself as a service with `ufw` upon 
 installation, making it straightforward to allow Nginx access.

 List the application configurations that `ufw` knows how to work with by typing
 `sudo ufw app list`
 A listing of the application profiles:
 ```
Output
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```
 use this command `sudo ufw allow 'Nginx Full'` 
 
 **Nginx Full**: This profile opens both port 80 (normal, unencrypted web traffic) and port 443 (TLS/SSL encrypted traffic)
 
 Use this command to confirm the ufw status `sudo ufw status`
 
 And finally use this command to start your nginx server `systemctl status nginx`

 This is an example of an Nginx configuration that was done for a webserver called glorypay.com with two backend servers called 
 gloryfront.com and gloryback.com:

 ```
 server {

        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        # root /var/www/html;

        # Add index.php to the list if you are using PHP
        # index index.html index.htm index.nginx-debian.html;

        server_name glorypay.com www.glorypay.com;

        location / {
                # First attempt to serve request as file, then
                proxy_pass http://localhost:8300;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
        }

        
    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/glorypay.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/glorypay.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}


      ## GLORY PAY BACK-END SERVICE
server {

        server_name gloryback.glorypay.com;

#       root /var/www/example.com;
#       index index.html;

        location / {
                # First attempt to serve request as file, then
                proxy_pass http://localhost:8300;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
        }


    listen [::]:443 ssl; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/gloryback.glorypay.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/gloryback.glorypay.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

## GLORYPAY FRONT-END SERVICE
server {

        server_name gloryfront.glorypay.com;

#       root /var/www/example.com;
#       index index.html;

        location / {
                # First attempt to serve request as file, then
                proxy_pass http://localhost:8006;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
        }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/gloryfront.glorypay.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/gloryfront.glorypay.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
```

#### The above proxy configuration was done using nginx in the `/etc/nginx/sites-available` directory with a symlink set up with the `sites-enabled`. The base URL for the server is `glorypay.com` and it contains two backend servers with endpoints called `gloryback.glorypay.com` and `gloryfront.glorypay.com` both opened on port `8300` and `8006` respectively. The client-side HTTPS request is managed using `Certbot`

#### Reason for Nginx reverse proxy:

  **Load Balancing**: Reverse proxies often distribute incoming client requests across multiple backend servers. This helps balance the load and ensures that no single server is 
    overwhelmed by traffic.
 
  **Caching**: Reverse proxies can cache static content or frequently accessed resources, reducing the load on backend servers and improving overall performance for users.
  
  **Centralized Logging and Monitoring**: By intercepting all traffic, reverse proxies can provide a centralized point for logging and monitoring incoming requests, responses, and 
    potential security threats.
  
  **Security**: Using Nginx for whitelisting involves configuring Nginx to allow access only to specified IP addresses or ranges while denying access to others. 



 2. ### The second task

    This question asks me to explain how you would deploy an application that requires 2.5 GB of memory to a Kubernetes cluster that has 
    two node pools, each with specific 
    computing resources. Additionally, the First Node pool has 1GB of Memory and 1 CPU and the Second Node pool has 3GB of Memory and 2 
    CPUs.

    Based on the information provided in the question, the first node pool with 1GB of memory is not suitable for deploying an application 
    that requires 2.5GB of memory. Therefore, 
    you should target the second node pool with 3GB of memory and 2 CPUs for this deployment.

    In order to achieve the usage of the second node pool, I would use `Node affinity rules`. Node affinity is a concept in Kubernetes 
    that allows you to influence the scheduling of 
    pods onto nodes based on custom rules or conditions. Node affinity rules help you control the placement of pods and ensure that they 
    are scheduled on nodes that meet certain 
    criteria or conditions.

    The Kubernetes Deployment Manifest

  ```
  apiVersion: apps/v1
kind: Deployment
metadata:
  name: your-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: your-app
  template:
    metadata:
      labels:
        app: your-app
    spec:
      containers:
        - name: your-app-container
          image: your-app-image:latest
          resources:
            requests:
              memory: "2.5Gi"  # Requests the minimum required memory
            limits:
              memory: "3Gi"    # Specifies the maximum allowed memory

  ```
  
  I would label my node pools:

  ```
  kubectl label nodes <node-name> node-pool=pool-1
  kubectl label nodes <node-name> node-pool=pool-2
  ```

  I would update my deployment manifest with node affinity:

  ```
  spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: node-pool
                    operator: In
                    values:
                      - pool-2
   ```

 Once you have your Deployment manifest ready, you can push it to your GitHub repository. Make sure to replace placeholders like your-app, 
 your-app-image, and update the labels as needed.

 A service manifest to allow the app to be acccessible outside the k8s cluster:

```
apiVersion: v1
kind: Service
metadata:
  name: your-app-service
spec:
  selector:
    app: your-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

 Using `kubectl` configured to connect to your Kubernetes cluster: `kubectl apply -f your-app-deployment.yaml`
