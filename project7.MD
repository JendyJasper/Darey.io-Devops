# DEVOPS TOOLING WEBSITE SOLUTION

> We want to introduce a set of DevOps tools that will help our team in day to day activities in managing, developing, testing, deploying and monitoring different projects.
> The tools we want our team to be able to use are well known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of:

- Jenkins – free and open source automation server used to build CI/CD pipelines.
- Kubernetes – an open-source container-orchestration system for automating computer application deployment, scaling, and management.
- Jfrog Artifactory – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
- Rancher – an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.
- Grafana – a multi-platform open source analytics and interactive visualization web application.
- Prometheus – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
- Kibana – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.

#### Create 3 Webservers and 1 storage server. Use RHEL-8.6.0_HVM-20220503-x86_64-2-Hourly2-GP2 AMI for RHEL 8.
<img width="1437" alt="Screenshot 2023-06-27 at 4 28 36 PM" src="https://github.com/JendyJasper/Darey.io-Devops/assets/29708657/7e73ba5a-d05f-4d36-812e-99c9f28be1a7">

## Prepare NFS Server

- Configure LVM on the Server.
- Format them as xfs
- Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs
- Create mount points on /mnt directory for the logical volumes as follow:
  - Mount lv-apps on /mnt/apps – To be used by webservers
  - Mount lv-logs on /mnt/logs – To be used by webserver logs
  - Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8
