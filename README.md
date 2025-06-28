# Container Monitoring with Prometheus, Grafana,  Docker

## _A tutorial on Docker metrics monitoring with Prometheus and Grafana._

Container Monitoring with Prometheus, Grafana,  Docker

Monitoring Docker containers is essential to ensure applications run smoothly, efficiently, and reliably by tracking resource usage and detecting performance issues early. Since containers can unpredictably consume CPU, memory, and network bandwidth, monitoring helps prevent bottlenecks, crashes, and downtime.

To achieve this, cAdvisor collects detailed, real-time resource metrics from each Docker container and feeds them to Prometheus, which stores and processes these metrics for scalable monitoring and alerting. Grafana then visualizes this data with customizable dashboards, making it easy to understand container health, spot trends, and respond proactively. Together, cAdvisor, Prometheus, and Grafana create a robust and comprehensive container monitoring solution.


By harnessing the power of these cutting-edge technologies, we'll build a sleek, all-in-one monitoring tool that keeps a vigilant eye on the health and vital stats of every Docker container in your ecosystem—giving you real-time insights and total control like never before!

Lets get started 

Pre requisites 
- Create a AWS account
- create a aws instance (free tire , AWS linux instance is best)
- In the instance install docker , grafana
	sudo dnf update -y – update packages
	sudo dnf install docker -y – installing docker command 
- start and stop
	sudo systemctl enable docker
	sudo systemctl start docker 
– run Docker commands without sudo
	sudo usermod -aG docker $USER 
		
Install grafana	
Import the GPG key:
```bash
wget  -q  -O gpg.key https://rpm.grafana.com/gpg.key  
sudo  rpm  --import gpg.key
```
    
Create /etc/yum.repos.d/grafana.repo with the following content:
```bash
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt			
```
			
To install Grafana OSS, run the following command:
	sudo dnf install grafana

Note , if any issue , use this commands to Enable and Start Grafana , also check status

sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server



After installation, you can quickly open your browser and enter your EC2 public IP address followed by port 3000 to view the Grafana login page: http://<your-server-ip>:3000.
Use the default username and password (admin/admin).
If you see the image below, you can assume that Grafana has been successfully installed.

### Grafana login

[![11.png](https://i.postimg.cc/dtW4fmNz/11.png)](https://postimg.cc/6yvr2RX0)

Check if any container are running 

			docker ps -a 
### Contains running

[![1.png](https://i.postimg.cc/QdGkw4kY/1.png)](https://postimg.cc/tYkxVzcF)


Once you've confirmed that no containers are currently running on the EC2 instance, we will try to create a few containers for the purpose of integrating them with Grafana and Prometheus.

We’ll use simple and commonly used containers for learning purposes — such as nginx, httpd, and WordPress.
You can create any number of containers depending on your requirements.


Command to pull and run the nginx container , httpd container , wordpress container
```sh
docker pull nginx:latest
docker run -d --name nginXX -p 8081:81 nginx (run this directly by skping the above command)

docker pull httpd
docker run -d --name HttpD -p 8082:82 httpd

docker pull wordpress:php8.4-fpm-alpine
docker run -d --name Wordpress -p 9091:91 wordpress
```

Once you have successfully created a container, run the docker ps command to view all active containers.

In the same directory, create two files: docker-compose.yml and prometheus.yml.

Before we proceed further, it's important to understand that for monitoring small to medium-sized applications, we’ll be using cAdvisor. There are many tools available for monitoring containers, but we have chosen cAdvisor for its simplicity and effectiveness.

A Brief Overview of cAdvisor
(Official documentation: https://github.com/google/cadvisor)

cAdvisor (Container Advisor) provides insight into the resource usage and performance characteristics of running containers. It runs as a daemon that collects, aggregates, processes, and exports data related to containers.

For each container, cAdvisor tracks:

Resource isolation parameters
Historical resource usage
Histograms of resource usage over time
Network statistics

This data is exported at both the container and machine level, making it highly useful for monitoring and analysis.

 Use Cases for cAdvisor
Lightweight monitoring on single Docker hosts
Ideal for development and testing environments
Commonly paired with Prometheus to collect basic container statistics
Great for quick proof-of-concept setups

In these files docker-compose.yml and prometheus.yml we will edit them one by one 

add to github

docker-compose.yml (code in the repo)

## Explaination for the above yml in simple terms

This docker-compose.yml file sets up a monitoring stack with Prometheus, cAdvisor, and Redis as services. Prometheus is configured to scrape metrics using a local prometheus.yml config file and depends on cAdvisor for container metrics. cAdvisor collects resource usage and performance metrics from Docker containers, and Redis is included as a sample monitored service.
In the provided docker-compose.yml, Redis is included as a sample container for monitoring purposes — it’s not being used by Prometheus or cAdvisor themselves.

 prometheus.yml (code in repo)

This YAML snippet tells Prometheus to:

Create a job called cadvisor.Scrape (collect) metrics every 5 seconds.
Get those metrics from the service running at cadvisor:8080 (i.e., the cAdvisor container).
In simple terms:
It configures Prometheus to pull container metrics from cAdvisor every 5 seconds.



---

Before running Docker Compose, it’s important to first check whether it is installed on your system. You can do this by running the command `docker compose version`. If Docker Compose is not installed, you’ll need to install it manually. Begin by creating the required directory using `sudo mkdir -p /usr/local/lib/docker/cli-plugins`. Then, download the Docker Compose plugin by running:

```bash
sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 \
  -o /usr/local/lib/docker/cli-plugins/docker-compose
```

Once downloaded, make the plugin executable by running `sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose`. After this, you can verify the installation again using `docker compose version`.

Once Docker Compose is successfully installed, you can start your services by running `docker compose up`. If everything is configured correctly, you should see all the services—such as cAdvisor and Prometheus—running.

[![2.png](https://i.postimg.cc/sgY9rYTj/2.png)](https://postimg.cc/9DfqP73K)

To confirm that cAdvisor is running, open a web browser and navigate to `http://<instance_ip>:8080`, replacing `<instance_ip>` with the public IP address of your instance. If successful, you should see the cAdvisor dashboard.
[![4.png](https://i.postimg.cc/jq1hQdWb/4.png)](https://postimg.cc/yWF97KXQ)

Similarly, to check if Prometheus is accessible, open the browser and go to `http://<instance_ip>:9090`.

[![3.png](https://i.postimg.cc/c48BDyY8/3.png)](https://postimg.cc/dLwC1f5q)

Within Prometheus, you’ll see various container metrics that can be explored by using the **Execute** button after entering PromQL queries. While Prometheus provides access to raw metrics data, the true visualization and monitoring capabilities come from integrating it with Grafana. Prometheus functions as the data collector and time-series database, whereas Grafana serves as the dashboard and visualization layer.

As mentioned earlier, Grafana is already installed on the system. The next step will be to configure it so it can pull data from Prometheus and display meaningful dashboards.


Login to grafana
[![5.png](https://i.postimg.cc/B6YBzJM2/5.png)](https://postimg.cc/njmmjf8L)

Click on connection and Data Sources

[![6.png](https://i.postimg.cc/x8By0trC/6.png)](https://postimg.cc/SY7zdLxF)

In the Connections enter the Ip of host (AWS Instance with the port of grafana 3000)

[![7.png](https://i.postimg.cc/nrXGbNPg/7.png)](https://postimg.cc/WqPk0Yjw)

Hit save , success message is appeares

[![8.png](https://i.postimg.cc/635hz8L9/8.png)](https://postimg.cc/cK2YL4bj)

[![9.png](https://i.postimg.cc/GpjQkFD7/9.png)](https://postimg.cc/KKRBbgJB)

Click on the Dashbords , on to right corner click Import and enter 193 (is the docker grafana pre-defined dash board templete)

[![10.png](https://i.postimg.cc/j5rpj4bM/10.png)](https://postimg.cc/VSDGGM5t)

Up on Success , you will see the Container details , Memeory details etc ..

[![11.png](https://i.postimg.cc/dtW4fmNz/11.png)](https://postimg.cc/6yvr2RX0)



### Conclusion
In this tutorial, we’ve successfully demonstrated how to set up a comprehensive container monitoring solution using Prometheus, Grafana, cAdvisor, and Docker. This stack empowers you to monitor the health, performance, and resource usage of your Docker containers in real-time. By leveraging cAdvisor to collect metrics, Prometheus for data storage and alerting, and Grafana for visualization, you gain deep insights into your containerized applications. This setup helps to proactively identify performance bottlenecks, reduce downtime, and optimize resource utilization, ensuring that your containers run smoothly and efficiently. Whether in development or production, this monitoring system enhances visibility and control, making it a vital tool for modern DevOps practices and container management.
