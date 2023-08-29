# hydrogen-devops-chanllenge

## Tasks:

1. Provide steps to set up Nginx reverse proxy, as well as reasons for it.

2. There are two node pools in a Kubernetes cluster, which pool contains a single node. The computing resources for each node pool are 1GB Memory and 1 CPU, 3GB Memory, and 2 CPU. How will you deploy an application that requires 2.5 GB of Memory? Kubernetes manifest codes can be pushed to Github, documentation is appreciated

3. You have ssh access to an Ubuntu or Debian Linux server, The disk is full and you need to clean it up, what would you do and which specific commands should you run?

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

   **Using the `apt` packaging system, we need to first update or local package index**

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


