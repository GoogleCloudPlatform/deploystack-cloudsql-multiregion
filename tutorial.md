# Prepare your Database for Disaster Recovery with Cloud SQL

## Let's get started!

This blueprint creates a [Cloud SQL instance](https://cloud.google.com/sql) with multi-region read replicas as described in the [Cloud SQL for PostgreSQL disaster recovery](https://cloud.google.com/architecture/cloud-sql-postgres-disaster-recovery-complete-failover-fallback) article.

The solution is resilient to a regional outage. 

To get familiar with the procedure needed in the unfortunate case of a disaster recovery, please follow steps described in [part two](https://cloud.google.com/architecture/cloud-sql-postgres-disaster-recovery-complete-failover-fallback#phase-2) of the aforementioned article.

This repo is based on the Cloud Foundation Fabric blueprint available [here](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/blueprints/data-solutions/cloudsql-multiregion).

**Time to complete**: About 10 minutes

Click the **Start** button to move to the next step
  
## Requirements

This blueprint will deploy all its resources into the project defined by the `project_id` variable. 

Please note that we assume this project already exists. 

However, if you provide the appropriate values to the `project_create` variable, the project will be created as part of the deployment.

If `project_create` is left to `null`, the identity performing the deployment needs the `owner` role on the project defined by the `project_id` variable. 

Otherwise, the identity performing the deployment needs `resourcemanager.projectCreator` on the resource hierarchy node specified by `project_create.parent` and `billing.user` on the billing account specified by `project_create.billing_account_id`.

## Deploy the architecture

Before we deploy the architecture, you will need the following information:

- The service project ID.
- A unique prefix that you want all the deployed resources to have (for example: cloudsql-multiregion-hpjy). This must be a string with no spaces or tabs.

During the process, you will be asked for some user input. All necessary variables are explained in the full README file. 

In case the deployment has not started, please run the following command:

``` bash
deploystack install
```


## Test your environment

We assume all those steps are run using a user listed on `data_eng_principals`.

You can authenticate as the user using the following command:

```shell
gcloud init
gcloud auth application-default login
```

Below you can find commands to connect to the VM instance and Cloud SQL instance.

```shell
  $ gcloud compute ssh sql-test --project PROJECT_ID --zone ZONE
  sql-test:~$ cloud_sql_proxy -instances=CLOUDSQL_INSTANCE=tcp:5432
  sql-test:~$ psql 'host=127.0.0.1 port=5432 sslmode=disable dbname=DATABASE user=USER'
```

You can find computed commands on the Terraform `demo_commands` output.

## Cleaning up your environment

The easiest way to remove all the deployed resources is to run the following command in Cloud Shell:

``` bash
deploystack uninstall
```

The above command will delete the associated resources so there will be no billable charges made afterwards.

## Move to real use case consideration

This implementation is intentionally minimal and easy to read. A real world use case should consider:

- Using a Shared VPC
- Using VPC-SC to mitigate data exfiltration

### Shared VPC
The example supports the configuration of a Shared VPC as an input variable. 

To deploy the solution on a Shared VPC, you have to configure the `network_config` variable:

```
network_config = {
    host_project       = "PROJECT_ID"
    network_self_link  = "https://www.googleapis.com/compute/v1/projects/PROJECT_ID/global/networks/VPC_NAME"
    subnet_self_link   = "https://www.googleapis.com/compute/v1/projects/PROJECT_ID/regions/$REGION/subnetworks/SUBNET_NAME"
    cloudsql_psa_range = "10.60.0.0/24"
  }
```

To run this example, the Shared VPC project needs to have:
 - A Private Service Connect with a range of `/24` (example: `10.60.0.0/24`) to deploy the Cloud SQL instance.
 - Internet access configured (for example Cloud NAT) to let the Test VM download packages. 

In order to run the example and deploy Cloud SQL on a shared VPC the identity running Terraform must have the following IAM role on the Shared VPC Host project.
 - Compute Network Admin (roles/compute.networkAdmin)
 - Compute Shared VPC Admin (roles/compute.xpnAdmin)
