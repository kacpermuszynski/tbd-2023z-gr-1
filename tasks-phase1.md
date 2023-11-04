1. Authors:

   ***Group 1***
   Kacper Muszyński
   Jakub Kliszko
   Mikołaj Paszkowski

   ***https://github.com/kacpermuszynski/tbd-2023z-gr-1***
   
2. Fork https://github.com/bdg-tbd/tbd-2023z-phase1 and follow all steps in README.md.

   :white_check_mark:
    ***Repo cloned, URL above***

3. Select your project and set budget alerts on 5%, 25%, 50%, 80% of 50$ (in cloud console -> billing -> budget & alerts -> create buget ; unclick discounts and promotions&others while creating budget).

   :white_check_mark:
    ***Done, screenshot below***
  ![img.png](doc/figures/buget-and-alerts.png)

4. From available Github Actions select and run destroy on main branch.

   :white_check_mark:
    ***Done, screenshot below***
  ![img.png](doc/figures/ga-destroy.png)

5. Create new git branch and add two resources in ```/modules/data-pipeline/main.tf```:
    1. resource "google_storage_bucket" "tbd-data-bucket" -> the bucket to store data. Set the following properties:
        * project  // look for variable in variables.tf
        * name  // look for variable in variables.tf
        * location // look for variable in variables.tf
        * uniform_bucket_level_access = false #tfsec:ignore:google-storage-enable-ubla
        * force_destroy               = true
        * public_access_prevention    = "enforced"
        * if checkcov returns error, add other properties if needed
       
    2. resource "google_storage_bucket_iam_member" "tbd-data-bucket-iam-editor" -> assign role storage.objectUser to data service account. Set the following properties:
        * bucket // refer to bucket name from tbd-data-bucket
        * role   // follow the instruction above
        * member = "serviceAccount:${var.data_service_account}"

    ***insert the link to the modified file and terraform snippet here***

    ***Link to file*** https://github.com/kacpermuszynski/tbd-2023z-gr-1/blob/master/modules/data-pipeline/main.tf

    ***Snippet***
    ```terraform
    resource "google_storage_bucket" "tbd-data-bucket" {
        project                     = var.project_name
        name                        = var.data_bucket_name
        location                    = var.region
        uniform_bucket_level_access = false #tfsec:ignore:google-storage-enable-ubla
        public_access_prevention    = "enforced"
        force_destroy               = true
    }

    resource "google_storage_bucket_iam_member" "tbd-data-bucket-iam-editor" {
        bucket = google_storage_bucket.tbd-data-bucket.name
        role   = "roles/storage.objectUser"
        member = "serviceAccount:${var.data_service_account}"
    }
    ```

    Create PR from this branch to **YOUR** master and merge it to make new release. 
    
    ***place the screenshot from GA after successful application of release with this changes***

    **Successfully GA output with changes:**

    ![img.png](doc/figures/bucket-success-ga.png)

6. Analyze terraform code. Play with terraform plan, terraform graph to investigate different modules.

    ***describe one selected module and put the output of terraform graph for this module here***

    **Terraform plan**

    Terraform plan is a command in Terraform, which lets you check what resources, will be created, deleted or update without making any changes to the real environment.

    Terraform plan is similar to terraform apply, it says how many resources will be changed, but the changes do not apply. It accepptes variables and informes about errors.

    Terraform plan reads the Terraform state file, so it it is up to date will recent configuration of all of the resources.

    **Output of Terraform graph**
    ![img.png](doc/figures/graph.png)
   
7. Reach YARN UI
   
   ***place the port and the screenshot of YARN UI here***
   
8. Draw an architecture diagram (e.g. in draw.io) that includes:
    1. VPC topology with service assignment to subnets
    2. Description of the components of service accounts
    3. List of buckets for disposal
    4. Description of network communication (ports, why it is necessary to specify the host for the driver) of Apache Spark running from Vertex AI Workbech
  
    ***place your diagram here***

9. Add costs by entering the expected consumption into Infracost

   ***place the expected consumption you entered here***

   ***place the screenshot from infracost output here***

10. Some resources are not supported by infracost yet. Estimate manually total costs of infrastructure based on pricing costs for region used in the project. Include costs of cloud composer, dataproc and AI vertex workbanch and them to infracost estimation.

    ***place your estimation and references here***

    ***what are the options for cost optimization?***
    
12. Create a BigQuery dataset and an external table
    
    ***place the code and output here***
   
    ***why does ORC not require a table schema?***
  
13. Start an interactive session from Vertex AI workbench (steps 7-9 in README):

    **Screenshot of Vertex AI workbench**

    ![img.png](doc/figures/vertex-hello-world.png)
   
14. Find and correct the error in spark-job.py

    ***describe the cause and how to find the error***

15. Additional tasks using Terraform:

    1. Add support for arbitrary machine types and worker nodes for a Dataproc cluster and JupyterLab instance

    ***place the link to the modified file and inserted terraform code***
    
    3. Add support for preemptible/spot instances in a Dataproc cluster

    ***place the link to the modified file and inserted terraform code***
    
    3. Perform additional hardening of Jupyterlab environment, i.e. disable sudo access and enable secure boot
    
    ***place the link to the modified file and inserted terraform code***

    4. (Optional) Get access to Apache Spark WebUI

    ***place the link to the modified file and inserted terraform code***
