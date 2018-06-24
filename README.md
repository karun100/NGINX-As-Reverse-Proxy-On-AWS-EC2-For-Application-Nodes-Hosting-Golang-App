# NGINX-As-Reverse-Proxy-On-AWS-EC2-For-Application-Nodes-Hosting-Golang-App

1.**First will deploy Golang app to 2 x application nodes on Amazon EC2.**


- Launch 2 x AWS EC2 Instances as application nodes.To create the setup have choosen AMI (Amazon Machine Image).However any other Linux flavor can be used.

   - Configure **"Security Groups"** to expose the port number server will be listening on.As per the Golang application code used in this example **port number 8484** will be enabled under **Custom TCP** and **rule 0.0.0.0/0**  will be applied on the **INCOMING traffic** in the **Source** column. It means anyone anywhere can connect via the specified port i.e **8484**.For **SSH Access** port **22** will already be allowed by default.
  - When prompted for **key pair** either choose an existing one or create a new key pair.This private key file will be used to connect by SSH to the instance.
  
- Now will be installing Golang on 2 x AWS EC2 Instances using **yum** package manager as below:-

```
sudo yum update -y
sudo yum install -y golang
```
- Need to set up Go environment variables. Mainly will need to setup 3 environment variables i.e GOROOT, GOPATH and PATH.**GOPATH** can be any directory on the system whereas **GOROOT** is where **GO** package is installed. Below commands can be added in ‘~/.bash_profile’ file as well to make this startup configuration permanent.To verify environment vaiables are set correctly can use **go env** etc. 

```
export GOROOT=/usr/lib/golang
export GOPATH=$HOME/projects
export PATH=$PATH:$GOROOT/bin
```

- As final step on application nodes or backend servers will be deploying Go App as per the below code in a **.go** file.Application node will be listening on the **port number 8484** as per **Security Group** policy.Now simply run Go App using command **go run <application_name>.go**.

```
package main

import (
	"fmt"
	"net/http"
	"os"
)

func handler(w http.ResponseWriter, r *http.Request) {
	h, _ := os.Hostname()
	fmt.Fprintf(w, "Hi there, I'm served from %s!", h)
}

func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServe(":8484", nil)
}
```

- To verify Go App installed correctly and appliction node listening correctly on **port number 8484** in an web browser hit enter to the DNS name of AWS EC2 instances.In my case on sending a HTTP request to the web node returned correctly the response as below:-

```
Hi there, I'm served from <application node hostname>!
```

2.**Nginx As Reverse Proxy to be deployed on a web node.It should load balance the traffic to 2 app servers or nodes deployed above hosting Golang app**.

- Launch 1 x AWS EC2 Instances as web node to deploy nginx.To create the setup have choosen AMI (Amazon Machine Image) Ubuntu 16.04 Server.However any other Linux flavor can also be used.

   - Configure **"Security Groups"** to expose the port number frontend or web server will be listening on.It can be that web server is listening on port 80 and application node on 8484.However have choosen here  same **port number 8484** for brevity keeping web server and application node listening on the same **port**.Hence **port number 8484** will be enabled under **Custom TCP** and **rule 0.0.0.0/0**  will be applied on the **INCOMING traffic** in the **Source** column. It means anyone anywhere can connect to the web server via the specified port i.e **8484**.For **SSH Access** port **22** will already be allowed by default.
   
  - When prompted for **key pair** either choose an existing one or create a new key pair.This private key file will be used to connect by SSH to the instance.
  
- Now will be installing nginx on 1 x AWS EC2 Instance as below:-

```
sudo apt-get install -y nginx
```
- Check the status of nginx installed using below command. Also on trying to access EC2 DNS name in a web browser on port 8484, it shows ‘Welcome to nginx!’. This means that nginx is running.

```
sudo systemctl status nginx 
```
- Nginx server configuration is stored in ‘etc/nginx’ and the main file is nginx.conf. Also each of the sites that we enable is available in ‘sites-available’.We can use a separate file for each virtual domain or site hosting.There is only one site as default and to check if it is enabled please run command as below:-

```
ls -la sites-enabled/
```


