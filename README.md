#  Implement DevOps process and principles on Hospital Management System

##  System Requirements: 
1. These are the necessary components in order to implement DevOps processes in the system:
	- Docker Runtime and docker compose. || Needs to be in remote server and local.
	- Jenkins || Needs to be in local server. || You are working on a multi branch pipeline
	- Git || Gitlab https://gitlab.com/asher-lab/SIMPLE-CRUD-APP_DEVOPS


## 🧊 Development
Step 1: Pull the Repo and Create a New Branch (e.g. bugfix or feature branch)
```
commands
```
Step 2: Commit Changes to Local Machine.
```
commands
```

## 🧊Push Code Changes to Repository
Step 1: Push your code changes to `YOUR-BRANCH`
```
commands
```
Step 2: Merge `YOUR-BRANCH` to `MAIN`
```
1.  git checkout <branch name>
2.  git merge main
https://www.varonis.com/blog/git-branching/
```
## 🧊Webhooks
Build the project when triggered using Jenkins
```
Implement using Gitlab and Jenkins
```
## 🧊Testing 
Be sure npm is installed on your system ( of which your Jenkins server is stored). Or you can install the plugin
`npm test`


## 🧊 Build Image 
Ensure the chmod 666 are set:
`
### A .  Create a  `Dockerfile`  in your project (  my-app-crud:1.0 . )
```
git clone https://gitlab.com/asher-lab/SIMPLE-CRUD-APP_DEVOPS.git
cd SIMPLE-CRUD-APP_DEVOPS
npm install --save express mysql body-parser hbs
```
Create a `Dockerfile` within the same directory:
```
FROM node:14.17.5
ENV NODE_ENV=production
WORKDIR /app
COPY ["package.json", "package-lock.json*", "./"]
RUN npm install --production
COPY . .
EXPOSE 8000
CMD [ "node", "index" ]
```
Create a Docker Image:
```
docker build -t my-app-crud:1.0 .
```
## 🧊 Security Scan
https://github.com/docker/scan-cli-plugin

## 🧊 Publish
### A. Publish the image to a remote repository
In certain case you need to perform docker login to access the private repository.
```
docker tag my-app-crud:1.0 asherlab/my-app-crud:1.0
docker push asherlab/my-app-crud:1.0
||| Try: docker pull asherlab/my-app-crud:1.0
```
## 🧊 Deploy

### A.  Preparing the necessary files
Make sure the directory your are on has the following files and folders:
	
- mysqldump/sql.db
- docker-compose.yaml
- prometheus.yml
- credentials and IP Address of another server:
- - be sure that SSH Agent plugin is installed

Make sure your have the docker image named either locally or via remote. In case of remote, change the source of the repository in docker-compose.yaml. In certain case you need to perform docker login to access the private repository:

- my-app-crud:1.0

**THE CONTENTS:**
`1. mysqldump/sql.db`
```
CREATE table product
(
	product_id int auto_increment primary key,
	product_name varchar(200) null,
	product_price int null
)charset=latin1;

INSERT INTO kaushik.product (product_id, product_name, product_price) VALUES (1, 'Product 1', 2000);
INSERT INTO kaushik.product (product_id, product_name, product_price) VALUES (2, 'Product 2', 2000);
INSERT INTO kaushik.product (product_id, product_name, product_price) VALUES (3, 'Product 3', 3000);
INSERT INTO kaushik.product (product_id, product_name, product_price) VALUES (4, 'Product 4', 2000);
INSERT INTO kaushik.product (product_id, product_name, product_price) VALUES (5, 'Product 5', 1500);
```
`2. docker-compose.yaml`
```
version: '3'

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data: {}

services:
  mysql:
    image: mysql:5.6
    ports:
      - 3307:3306
    environment:
      - MYSQL_ROOT_PASSWORD=newpass
      - 'MYSQL_DATABASE=kaushik'
    volumes:
      - ./mysqldump:/docker-entrypoint-initdb.d # mysqldump folder should contain db.sql
    restart: unless-stopped

  my-app-crud:
    image: asherlab/my-app-crud:1.0
    restart: unless-stopped
    network_mode: "host"

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
#    expose:
#      - 9100
    ports:
     - "9091:9100"

    networks:
      - monitoring

  promcontainer:
    image: "prom/prometheus:latest"
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/etc/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
#    expose:
#      - 9090
    networks:
     - monitoring
    ports:
     - "9090:9090"

  grafanacontainer:
    image: "grafana/grafana"
    ports:
     - "3000:3000"
    networks:
     - monitoring
```
`3. prometheus.yml`
```
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'prometheus_master'
    scrape_interval: 5s
    static_configs:
      - targets: ['34.234.35.40:9090']

  - job_name: 'node_exporter_centos'
    scrape_interval: 5s
    static_configs:
      - targets: ['34.234.35.40:9091']

```
```
# ------- kill all apps that uses my needed ports -------#
netstat -lpnt

#---------------- Test if is running ---------------#

docker-compose -f docker-compose.yaml up -d

# ------------ RESETTER ------------------#
docker rm -f $(docker ps -a -q)
docker rmi -f $(docker images -a -q)
```

## 🧊 Full CI/CD Pipeline
This pipeline will:
- Test
- Build
- Publish
- Deploy

`Jenkinsfile`
```
pipeline {
    agent any
	
    stages {
	

		stage("test") {
			steps {
				script {
					echo "Testing the code.."
					sh 'npm test '
				}
			}
		}

		


		stage("build image") {
			steps {
				script {
					echo "Building docker image.."	
					
					
					
						sh 'docker build -t my-app-crud:latest .'
						withCredentials([usernamePassword(credentialsId: 'dockerhub-asherlab', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
					sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"	
					
					}

						sh 'docker tag my-app-crud:latest asherlab/my-app-crud:latest'
						
				   
			   }	
			}
		}


		stage("Scan Image") {
			steps {
				script {



				echo "Performing security scan.."

				
				try {
					sh "docker scan asherlab/my-app-crud:latest"
				} catch (Exception e) {
					sh " echo Security Team PLS fix the vulnerability!"
					
				}
					
					
				}
			}
		}
		

		stage("Publish Image") {
			steps {
				script {
					echo "Publishing Image to docker hub repo.."
					
					withCredentials([usernamePassword(credentialsId: 'dockerhub-asherlab', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]){
					sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"	
					sh 'docker push asherlab/my-app-crud:latest'
					}
					
					
				}
			}
		}


		
		
	stage("deploy") {
				steps {
					script {
						
				echo "Deploying the image to a remote EC2 instances.."
				//login on remote server (ec2 - deployment server) so it can access the repo
				
				// or you can just login on the remote deployment server
				
				//define function				
				def dockerComposeCmd = 'docker-compose -f docker-compose.yaml up -d'
				echo "Deploying the package.."
				
   				sshagent(['EC2-Creds']) { 
   				// some block 
				//located in var/jenkins_home/workspace/keys
   				sh "scp -rp -i ../keys/asher.pem docker-compose.yaml ubuntu@44.194.205.9://home/ubuntu"
   				sh "ssh -o StrictHostKeyChecking=no ubuntu@44.194.205.9 ${dockerComposeCmd}"
   				
				// make sure the host system have: SSH-Agent plugin
					
					}
										
					}
				}
			}
}
}

```


## Deleted authorized_keys but still connected:
https://serverfault.com/questions/695178/deleted-authorized-keys-from-ec2-but-still-have-ppk-file-and-im-connected
https://superuser.com/questions/1295949/can-you-recover-ssh-contents-on-aws-ec2-with-an-open-session-and-keys

## On permission denied on creating a dir
https://unix.stackexchange.com/questions/119358/create-file-in-folder-permission-denied
