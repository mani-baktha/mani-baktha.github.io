---
aliases:
- /Courses/Data Engineering/2022/01/29/data-engineering-zoomcamp
categories:
- Courses
- Data Engineering
date: '2022-01-29'
description: Learning journey for Data Engineering Zoomcamp
layout: post
title: Data Engineering Bootcamp Weeks 1-2 (Ongoing)
toc: true

---

# Data Engineering Weeks 1-2 (Ongoing)

The objective of this post is to document my learning journey while taking the data engineering zoomcamp. I use Windows+WSL for Linux support. I will defer from posting installation related issues here, unless it seems helpful either for reference in the future or to the other course takers.

**Notes for other learners**
- If you are taking the data engineering zoomcamp, please follow the course at the [official zoomcamp repository](https://github.com/DataTalksClub/data-engineering-zoomcamp). The course creators have done a nice job in compiling this step by step. 
- Please use the notes here only as a complementary or additional resource
- You can ask questions in the channel `#course-data-engineering` in [DataTalks.Club](https://datatalks.club)
- This is an innovative and community based live course provided by some kind-hearted people for free. A good way to contribute back is to document the issues you face on the [official FAQ page](https://docs.google.com/document/d/19bnYs80DwuUimHM65UV3sylsCn2j1vziPOwzBwQrebw/edit?usp=sharing)


## Week 1 - Docker

**Docker** is a to package applications into containers which can
be deployed/shipped anywhere. Docker achieves this through OS virtualization technology. Each docker container has its own OS, libraries and configuration files in an isolated space. But docker containers can colloborate with each other though channels. Let us say we have built a python application which runs on our local windows machine. If we **containerize** this application, then we can take the exact same container and make it run on another machine running another OS and on the cloud. I have already used a fair amount of docker. Therefore, it was easy to pick up. 

> In this course, docker is chosen so that the *data pipeline* we are going to develop can be containerized. Then, this containerized pipeline can be deployed on the cloud with services like AWS Batch or Kubernetes jobs. The Spark and Serverless Technologies(AWS Lambda, Google Functions) explored in this course are all containerized.

The following exercises were performed hands-on during this week

### **Setup containerized `postgres` with persistence on local disk** 

We will later ingest New York taxis dataset into a postgres database. For persistence, I mapped an empty local folder `ny_taxi_postgres_data` to `/var/lib/postgresql/data` and this
ran into permission issues because the released postgres container has only user "postgres". I tried some suggested workarounds and finally settled on this one - we can create a local docker volume and map it to postgres data folder to avoid the permissions issue.
 
 
`docker volume create --name dtc_postgres_volume_local -d local`  

```
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v dtc_postgres_volume_local:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

### **Containerize python application**
I built a simple docker image of a python application which uses pandas. 

> In fact, we can collect all python requirements into a `requirements.txt` file which can be used by `pip` on the container to prepare python environments in a clean way. 


## Week 2 - Terraform and GCP

### Terraform
1. Terraform is an open source tool which allows to handle "Infrastructure-as-Code". For real beginners, By infrastructure, we mean not only computing infratructure (AWS EC2, Google Cloud machines) but also other **resources** like databases. If this does not make sense to you, please read [this page](https://learn.hashicorp.com/tutorials/terraform/infrastructure-as-code?in=terraform/docker-get-started)

2. Where? Terraform allows to provision infrastructure locally or on the cloud with various cloud providers like AWS, GCP etc. This provisioning of infrastructure can be done for testing, production etc. In Terraform, **Providers** are plugins that implement resource types. Docker, AWS, Google Cloud, Alibaba Cloud are all providers.

3. How? Using **configuration files** which are source-controlled. This approach helps to keep track of what infrastructure we used for which purpose in a clean and efficient manner. Moreover, Hashicorp, the company behind Terraform provides us with small, reusable Terraform configurations called **Modules**. We can use these to manage a group of related resources as if they were a single resource.

*I recommend doing the official [Terraform docker Interactive Lab]() in which terraform uses docker as a provider. The resources provided are a docker image and docker container. No installations are required to follow this lab.*

### GCP
This course introduces GCP as a provider which will provide us with the following 2 resources:
1. Data Lake - Google Cloud Storage(or GCS)
2. Data Warehouse - BigQuery

#### Data Warehouse vs Data Lake

This is essentially ETL vs ELT. Data warehousing has existed for a long time. It uses structured data, relational and clean in the order of Terabytes. Business Analysts use data warehouses for specific use-cases like BI, reporting and batch processing. Data Lakes came about later taking into account the fact that businesses wanted data to be made immediately useful, as soon as it is available. So datalakes can have unstructured, semi-structured or structured data to the order of even Petabytes. The use cases are for streaming analytics, machine learning and AI.

E=Extract, T=Transform, L=Load. ETL is for small amounts of data or *Schema on Write* approach - a schema is defined, the relationshipa are clearly defined and then the data is written. ELT on the other hand is a *Schema on Read* approach where data is written first and we worry about schema when reading data. This approach provides support for data lakes.


### **Provision datalake and datawarehousing with Terraform on Google Cloud Platform**

The objective of the homework is to provision a datalake and a datawarehouse on Google Cloud Platform using Terraform.

First, on the google cloud platform (300$ free credits), it is necessary to setup 
1. A new goole cloud project (and note down its project ID) 
2. A new service account (through which to access the services provided by google cloud) and download this account's credentials as a json file and 
3. Set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable to this location  
`$ export GOOGLE_APPLICATION_CREDENTIALS="<path/to/your/service-account-authkeys>.json"`


`$ cd /mnt/Projects/data-engineering-zoomcamp/week_1_basics_n_setup/1_terraform_gcp/terraform`  

Here, we have two files `main.tf` and `variables.tf`. In `variables.tf`, we can set the google cloud project ID from the previous step 

`$ terraform init`

```
terraform init

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v4.8.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

`$ terraform plan`

```

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_bigquery_dataset.dataset will be created
  + resource "google_bigquery_dataset" "dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "trips_data_all"
      + delete_contents_on_destroy = false
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "europe-west6"
      + project                    = "fast-pagoda-339723"
      + self_link                  = (known after apply)

      + access {
          + domain         = (known after apply)
          + group_by_email = (known after apply)
          + role           = (known after apply)
          + special_group  = (known after apply)
          + user_by_email  = (known after apply)

          + view {
              + dataset_id = (known after apply)
              + project_id = (known after apply)
              + table_id   = (known after apply)
            }
        }
    }

  # google_storage_bucket.data-lake-bucket will be created
  + resource "google_storage_bucket" "data-lake-bucket" {
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "EUROPE-WEST6"
      + name                        = "dtc_data_lake_fast-pagoda-339723"
      + project                     = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = true
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type = "Delete"
            }

          + condition {
              + age                   = 30
              + matches_storage_class = []
              + with_state            = (known after apply)
            }
        }

      + versioning {
          + enabled = true
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

`$ terraform apply`

```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_bigquery_dataset.dataset will be created
  + resource "google_bigquery_dataset" "dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "trips_data_all"
      + delete_contents_on_destroy = false
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "europe-west6"
      + project                    = "fast-pagoda-339723"
      + self_link                  = (known after apply)

      + access {
          + domain         = (known after apply)
          + group_by_email = (known after apply)
          + role           = (known after apply)
          + special_group  = (known after apply)
          + user_by_email  = (known after apply)

          + view {
              + dataset_id = (known after apply)
              + project_id = (known after apply)
              + table_id   = (known after apply)
            }
        }
    }

  # google_storage_bucket.data-lake-bucket will be created
  + resource "google_storage_bucket" "data-lake-bucket" {
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "EUROPE-WEST6"
      + name                        = "dtc_data_lake_fast-pagoda-339723"
      + project                     = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = true
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type = "Delete"
            }

          + condition {
              + age                   = 30
              + matches_storage_class = []
              + with_state            = (known after apply)
            }
        }

      + versioning {
          + enabled = true
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_bigquery_dataset.dataset: Creating...
google_storage_bucket.data-lake-bucket: Creating...
google_storage_bucket.data-lake-bucket: Creation complete after 1s [id=dtc_data_lake_fast-pagoda-339723]
google_bigquery_dataset.dataset: Creation complete after 1s [id=projects/fast-pagoda-339723/datasets/trips_data_all]
```

`$ terraform destroy`

```
google_storage_bucket.data-lake-bucket: Refreshing state... [id=dtc_data_lake_fast-pagoda-339723]
google_bigquery_dataset.dataset: Refreshing state... [id=projects/fast-pagoda-339723/datasets/trips_data_all]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_bigquery_dataset.dataset will be destroyed
  - resource "google_bigquery_dataset" "dataset" {
      - creation_time                   = 1643547397011 -> null
      - dataset_id                      = "trips_data_all" -> null
      - default_partition_expiration_ms = 0 -> null
      - default_table_expiration_ms     = 0 -> null
      - delete_contents_on_destroy      = false -> null
      - etag                            = "uSEBYtpaZBci9jzHBKd49A==" -> null
      - id                              = "projects/fast-pagoda-339723/datasets/trips_data_all" -> null
      - labels                          = {} -> null
      - last_modified_time              = 1643547397011 -> null
      - location                        = "europe-west6" -> null
      - project                         = "fast-pagoda-339723" -> null
      - self_link                       = "https://bigquery.googleapis.com/bigquery/v2/projects/fast-pagoda-339723/datasets/trips_data_all" -> null

      - access {
          - role          = "OWNER" -> null
          - user_by_email = "data-engineer-service-account@fast-pagoda-339723.iam.gserviceaccount.com" -> null
        }
      - access {
          - role          = "OWNER" -> null
          - special_group = "projectOwners" -> null
        }
      - access {
          - role          = "READER" -> null
          - special_group = "projectReaders" -> null
        }
      - access {
          - role          = "WRITER" -> null
          - special_group = "projectWriters" -> null
        }
    }

  # google_storage_bucket.data-lake-bucket will be destroyed
  - resource "google_storage_bucket" "data-lake-bucket" {
      - default_event_based_hold    = false -> null
      - force_destroy               = true -> null
      - id                          = "dtc_data_lake_fast-pagoda-339723" -> null
      - labels                      = {} -> null
      - location                    = "EUROPE-WEST6" -> null
      - name                        = "dtc_data_lake_fast-pagoda-339723" -> null
      - project                     = "fast-pagoda-339723" -> null
      - requester_pays              = false -> null
      - self_link                   = "https://www.googleapis.com/storage/v1/b/dtc_data_lake_fast-pagoda-339723" -> null
      - storage_class               = "STANDARD" -> null
      - uniform_bucket_level_access = true -> null
      - url                         = "gs://dtc_data_lake_fast-pagoda-339723" -> null

      - lifecycle_rule {
          - action {
              - type = "Delete" -> null
            }

          - condition {
              - age                        = 30 -> null
              - days_since_custom_time     = 0 -> null
              - days_since_noncurrent_time = 0 -> null
              - matches_storage_class      = [] -> null
              - num_newer_versions         = 0 -> null
              - with_state                 = "ANY" -> null
            }
        }

      - versioning {
          - enabled = true -> null
        }
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

google_storage_bucket.data-lake-bucket: Destroying... [id=dtc_data_lake_fast-pagoda-339723]
google_bigquery_dataset.dataset: Destroying... [id=projects/fast-pagoda-339723/datasets/trips_data_all]
google_storage_bucket.data-lake-bucket: Destruction complete after 1s
google_bigquery_dataset.dataset: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
```