# Solution
## Assignment -I
## One time setup automation
### Requirements
- ansible version > 2.9
- Route53 entry vinga.tk
- Kubernets 3 node cluster.(Tested on AWS KOPS version 
### steps:

``` git clone https://github.com/vinga2805/ms.git ```

``` ansible-playbook fullstack.yml```
### Whats included ?
- 3 nodes Elasticsearch cluster for for storing application logs
- Kibana Dashboard for logs visulatization.
  - Index pattern app-logs*
- Prometheus for metrics and Alerting
- Grafana Dashboard for Application and Infrastructure monitoring with inbuilt 2 Dashboards.
   - jmx-prometheus-exporter (Application with jmx monitoring)
   - mysqld-prometheus (Mysql Monitoring)
- MysqlDB for storing the user and audit data.
- Nginx Ingress along with certmanager for issuing SSL cert for vinga.tk
- 100ms application for creating DynamoDB via UI and template.

### Output
- Once the playbook run successfully below urls will be automatically configured in r53 with nginx ingress.
- http://prometheus.vinga.tk
- http://grafana.vinga.tk
- http://kibana.vinga.tk
- http://ms-prod.vinga.tk/teslaDyDB/
- http://ms-stage.vinga.tk/teslaDyDB/
- Everything is tested successfully and uploaded the screenshots as evidance.

## Continous Integration and Deployment
### Requirements
- Jenkins server with below plugins installed
  - Docker pipeline 
  - Github Authentication
  - pipeline
- Maven with version 3.6.0
- Java openjdk 1.8.0_252

### Steps:
- Create a Jenkins pipeline Job and with below Jenkinsfile in "pipeline script from SCM"
  - app/Jenkinsfile
  
### Whats included ?
- Dockerfile for application.
- Dockerfile for fluentbit which will be running as a sidecar.
- Jenkinsfile to perform CI and CD
- update_image.py for updating the image in helm chart.

### What does it do?
- Clone the repo
- Build the jar file with maven
- Build the Docker image and names it accoring to the environment.
  - SNAPSHOT --> ${pomVersion}-snapshot.${gitBranch}.${shortCommit}
  - release  --> ${pomVersion}
- Push the built image with dockerhub with my credentials.
- Remove the image from Jenkins box
- Clone the same repo in subdirectory values-files
- Update the tag in the values.yaml 
- Bump up the chart version
- Check if the application is already deployed if not then it will bootstrap the application for the first time. (this can be first blue deployment)
- If the application is already deployed then check the slot if it is blue or green.
- If the slot is blue then parallel setup for green will be created for stage environment which will point to https://ms-stage.vinga.tk/teslaDyDB/
- Testing will be performed on the staged environment(green),once successful then will shift point the prod service pointing to green environment.
- If the testing fails then we will simply decomission the green environment.
- Once the prod service is pointed to green environment, we will decomission the blue environment.
- Perform healthcheck of the production application.
- If the Healthcheck passes then it will push the changes in github
- If the Healthcheck fails it performs rollback via helm.
- Use sample dynamoDB template to test the application.

### How to use the application
- Login https://ms-prod.vinga.tk/teslaDyDB
- Register new user with password. (AWS credentials are mandatory)
- you can login with username and password.
- You can create a dynamoDB from the console by providing all the mandatory details.
- You can also pass json template to create dynamoDB. (sample-template in the root for the project)

### Deployment flow
![alt text](https://github.com/vinga2805/ms/blob/master/artifacts/Screenshot%202021-05-18%20at%203.15.44%20PM.png)

![alt text](https://github.com/vinga2805/ms/blob/master/artifacts/Screenshot%202021-05-18%20at%203.18.09%20PM.png)

![alt text](https://github.com/vinga2805/ms/blob/master/artifacts/Screenshot%202021-05-18%20at%203.19.28%20PM.png)

![alt text](https://github.com/vinga2805/ms/blob/master/artifacts/Screenshot%202021-05-18%20at%203.22.18%20PM.png)

![alt text](https://github.com/vinga2805/ms/blob/master/artifacts/Screenshot%202021-05-18%20at%203.24.08%20PM.png)

### Note
As we know there are many tools in the market which supports B/G, Canary deployment out of the box,Helm is definately not right choice for this,
but to demonostrate the length and breadth of DevOps skill sets, I implemented this way.(Ansible/Jenkins(groovy), Kubernetes, helm, certmanager)  
