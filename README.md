# DE Zoomcamp 2024 - Project1 

This repository contains a brief description of my DE Zoomcamp 2024 Project 1

## Problem statement

The Retailrocket t has collected a large dataset of E-commerce i.e  a file with behaviour data (events.csv), a file with item properties (item_properties.сsv) and a file, which describes category tree (category_tree.сsv). The data has been collected from a real-world ecommerce website. It is raw data, i.e. without any content transformations, however, all values are hashed due to confidential issues. The purpose of publishing is to motivate researches in the field of recommender systems with implicit feedback.  The goal of this project is to create a streamlined and efficient process for analyzing e-commerce by implementing Data Engineering process flows basics.

## About the Dataset
[Retailrocket recommender system](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset) 

The dataset consists of three context files i.e. : 
1. a file with behaviour data (events.csv)
2. a file, which describes category tree (category_tree.сsv).
3. a file with item properties (item_properties_part1.сsv & item_properties_part2.csv)

The data has been collected from a real-world ecommerce website. It is raw data, i.e. without any content transformations, however, all values are hashed due to confidential issues.

The behaviour data, i.e. events like clicks, add to carts, transactions, represent interactions that were collected over a period of 4.5 months. A visitor can make three types of events, namely **view**, **addtocart** or **transaction**. 

## Technologies / Tools
- Containerisation : Docker
- Cloud : GCP
- Infrastructure as code (IaC) : Terraform
- Workflow orchestration : Mage-ai
- Data Warehouse : BigQuery
- Batch processing : pyspark
- IDE : VS Code, Jupyter Notebook
- Language : Python
- Visualisation : Google Looker Studio

## Project Architecture

Kaggle dataset is downloaded into the Google VM, then ingested to Google Cloud Storage Buecket as Data Lake. Next, the data will be stored in BigQuery as a Data Warehouse. All data flows are executed using the Mage-ai workflow orchestration tool. A Spark job is run on the data stored in the Google Storage Buecket or in BigQuery.
The results are written to a dafaframe and/or table in Postgres. A dashboard is created from the Looker Studio.

The end-to-end data pipeline includes the below steps:
- Kaggle dataset is downloaded into the Google VM
- The downloaded CSV files (raw) are then uploaded to a folder in Google Cloud bucket (parquet) as Data Like
- Next, the data will be stored in BigQuery as a Data Warehouse
- A new table is created from this original table with correct data types as well as partitioned by Month and Clustered by type of event for optimised performance
- Spin up a dataproc clusters (amster and worker) and execute the pyspark jobs for procusts analys purposes
- Configure Google Looker Studio to power dashboards from BigQuery Data Warehouse tables

You can find the detailed Architecture on the diagram below:

![image](https://github.com/garjita63/dezoomcamp2024-project1/assets/77673886/ef363cea-67c7-4a10-9dc7-0516dab7008d)


## Reproducing from Scratch

### Setup GCP
- Create GCP Account.
- Setup New Project and write down your Project ID.
- Configure service account to get access to the project and download auth-keys (.json). Change auth-keys name if required.
  Please provide the service account the permissions below (*sorted by name*):
  ```
  1. BigQuery Admin
  2. Cloud SQL Client
  3. Compute Admin
  4. Compute Engine Service Agent
  5. Compute Network Admin
  6. Compute Storage Admin
  7. Dataproc Service Agent
  8. Editor
  9. Logs Bucket Writer
  10. Owner
  11. Storage Admin
  12. Storage Object Admin
  ```
      
- Enable the following options under the APIs and services section:
  ```
  1. Identity and Access Management (IAM) API
  2. IAM service account credentials API
  3. Compute Engine API (if you are going to use VM instance)
  ```
  

### Terraform as Internet as Code (IaC) to build infrastructure
- Download Terraform from here: [https://www.terraform.io/downloads](https://www.terraform.io/downloads)
- Under terraform folder, create files **main.tf** (required) and **variables.tf** (optional) to store terraform variables. 
- main.td contents
  ```
  1. Google Provider Versions
  2. resource "google_service_account"
  3. resource "google_project_iam_member"
  4. resource "google_compute_firewall"
  5. resource "google_storage_bucket"
  6. resource "google_storage_bucket_iam_member"
  7. resource "google_bigquery_dataset"
  8. resource "google_dataproc_cluster" (cluster_config : master_config, worker_config, software_config : image_version = "2.2.10-debian12"
      optional_components   = ["DOCKER", "JUPYTER"])
  ```
- terraform init : command initializes the directory, downloads, teh necesary plugins for the cinfigured provider, and prepares for use.
- terraform plan : too see execution plan
- erraform apply : to apply the changes
  
If you would like to remove your stack from the Cloud, use the terraform destroy command.


### Reproducibility

After terrafor apply done :

- Assign External IP Address for Master and Workers Clusters. You can use either Console or gcloud :

**Console** :

From VM Instance (Compute Engine) - SSH

![image](https://github.com/garjita63/dezoomcamp2024-project1/assets/77673886/2d4ff3e3-a28a-4739-a17c-39d64ae4683e)

![image](https://github.com/garjita63/dezoomcamp2024-project1/assets/77673886/b67244eb-3b31-4f7d-ada6-76f261ba1887)

![image](https://github.com/garjita63/retailrocket-ecommerce-batch/assets/77673886/b0b4c8b8-84bb-40fa-bdde-cd1a517ba399)

![image](https://github.com/garjita63/retailrocket-ecommerce-batch/assets/77673886/b2ab4aaf-24db-49cc-9d35-c828777bb4e3)

![image](https://github.com/garjita63/retailrocket-ecommerce-batch/assets/77673886/096daaa8-c50d-44bf-8dcb-c6f0b9e30b9b)

**gloud shell** (local or cloud) :
```
gcloud compute instances add-access-config <master cluster> --access-config-name="project1-dataproc-m-config"
gcloud compute instances add-access-config <worker cluster 0> --access-config-name="project1-dataproc-m-config"
gcloud compute instances add-access-config <worker cluster 0> --access-config-name="project1-dataproc-m-config"
```

*Provide master and worker cluster names.*

- Set up Mage-ai, PostgreSQL and pgAdmin through Master SSH.

  Copy repsistories.sh into VM. repsistories.sh is script for installing docker network and bring up docker containers of Mage-ai, postgresql and pgAdmin. .
  ```
  #############Install Docker network#############
  #create a network most containers will use
  sudo docker network create dockernet >> /root/dockernet.log
  sudo docker network ls >> /root/dockernet.log
  
  #############Bring up docker containers############
  cat > /root/docker-compose.yml <<- "SCRIPT"
  
  version: '3'
  services:
    magic:
      image: mageai/mageai:latest
      command: mage start dezoomcamp
      container_name: dezoomcamp-mage
      build:
        context: .
        dockerfile: Dockerfile
      environment:
        USER_CODE_PATH: /home/src/dezoomcamp
        POSTGRES_DBNAME: dezoomcampdb
        POSTGRES_SCHEMA: public
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD: postgres316
        POSTGRES_HOST: vm-ikg-dezoomcamp
        POSTGRES_PORT: 5432
      ports:
        - 6789:6789
      volumes:
        - .:/home/src/
        - /root/.google/credentials/key-ikg-dezoomcamp-2024.json
      restart: on-failure:5
    postgres:
      image: postgres:14
      restart: on-failure
      container_name: dezoomcamp-postgres
      environment:
        POSTGRES_DB: dezoomcampdb
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD: postgres316
      ports:
        - 5432:5432
    pgadmin:
      image: dpage/pgadmin4
      container_name: dezoomcamp-pgadmin
      environment:
        - PGADMIN_DEFAULT_EMAIL=admin@admin.com
        - PGADMIN_DEFAULT_PASSWORD=root
      ports:
        - 8080:80
        
  SCRIPT
  
  sudo docker compose -f /root/docker-compose.yml up -d
  ```

  chmod +x repositories.sh

  sudo ./repositories.sh

  ==> *Mage-ai, postgresql and pgAdmin would be installed and up running.*


Check mage :

![image](https://github.com/garjita63/retailrocket-ecommerce-batch/assets/77673886/b3906b1d-0b46-4166-af52-525f86b60a0c)

Check pgadmin :

![image](https://github.com/garjita63/retailrocket-ecommerce-batch/assets/77673886/03991861-af32-4840-9d9d-d06f476da686)

Stop jupyter notebook
```
sudo systemctl stop jupyter
```

Restart Jupyter by using script below
```
jupyter-notebook  --port=8888 --ip=0.0.0.0 --no-browser
```

 ![image](https://github.com/garjita63/retailrocket-ecommerce-batch/assets/77673886/e78fc04d-9055-4aeb-ac27-5b877a99e1ec)

- Increase memory size for cluster if required

  ```
  jupyter notebook --generate-config
  ```
  
Open /home/smrhitam/.jupyter/jupyter_notebook_config.py 

Modify the
![image](https://github.com/garjita63/retailrocket-ecommerce-batch/assets/77673886/2464f7a3-aad7-4514-add2-412e36321bff)

- Spark amster and worker clusters

  Edit ~/.bashrc fileand add lines below:
  ```
  export SPATH=$SPARK_HOME/bin:$SPARK/sbin:$PATH
  ```

  Start amster and worker clusters
  ```
  start-all.sh
  ```

  Try run spark by using dataset on hdfs

  Copy dataset folder into /user/<some folder>
  ```
  hdfs dfs -mkdir /user/smrhitam
  hdfs dfs -copyFromLocal  ecommerce-dataset/ /user/s<some folder>
  ```

  Login to Web Master Cluster


  Login to Web Worker Cluster
   


## Dashboard

![locker-studio2](https://github.com/garjita63/retailrocket-ecommerce-batch/assets/77673886/73839329-bb0a-426e-bb95-44da5718504c)

![locker-studio1](https://github.com/garjita63/retailrocket-ecommerce-batch/assets/77673886/4ca8c142-1f90-4514-ab90-f5241f04f6ef)

