IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.

![img.png](doc/figures/destroy.png)

0. The goal of this phase is to create infrastructure, perform benchmarking/scalability tests of sample three-tier lakehouse solution and analyze the results using:
* [TPC-DI benchmark](https://www.tpc.org/tpcdi/)
* [dbt - data transformation tool](https://www.getdbt.com/)
* [GCP Composer - managed Apache Airflow](https://cloud.google.com/composer?hl=pl)
* [GCP Dataproc - managed Apache Spark](https://spark.apache.org/)
* [GCP Vertex AI Workbench - managed JupyterLab](https://cloud.google.com/vertex-ai-notebooks?hl=pl)

Worth to read:
* https://docs.getdbt.com/docs/introduction
* https://airflow.apache.org/docs/apache-airflow/stable/index.html
* https://spark.apache.org/docs/latest/api/python/index.html
* https://medium.com/snowflake/loading-the-tpc-di-benchmark-dataset-into-snowflake-96011e2c26cf
* https://www.databricks.com/blog/2023/04/14/how-we-performed-etl-one-billion-records-under-1-delta-live-tables.html

2. Authors:

   Group 1 Kacper Muszyński Jakub Kliszko Mikołaj Paszkowski

   https://github.com/kacpermuszynski/tbd-2023z-gr-1

3. :white_check_mark: Replace your `main.tf` (in the root module) from the phase 1 with [main.tf](https://github.com/bdg-tbd/tbd-workshop-1/blob/v1.0.32/main.tf)
at the same time changing each module `source` reference from the repo relative path to a github repo tag `v1.0.33` , e.g.:
```hcl
module "dbt_docker_image" {
  depends_on = [module.composer]
  source             = "github.com/bdg-tbd/tbd-workshop-1.git?ref=v1.0.33/modules/dbt_docker_image"
  registry_hostname  = module.gcr.registry_hostname
  registry_repo_name = coalesce(var.project_name)
  project_name       = var.project_name
  spark_version      = local.spark_version
}
```
4. :white_check_mark: Provision your infrastructure.

    a) setup Vertex AI Workbench `pyspark` kernel as described in point [8](https://github.com/bdg-tbd/tbd-workshop-1/tree/v1.0.32#project-setup) 

    b) upload [tpc-di-setup.ipynb](https://github.com/bdg-tbd/tbd-workshop-1/blob/v1.0.33/notebooks/tpc-di-setup.ipynb) to the running instance of your Vertex AI Workbench


5. :white_check_mark: In `tpc-di-setup.ipynb` modify cell under section ***Clone tbd-tpc-di repo***:

   a)first, fork https://github.com/mwiewior/tbd-tpc-di.git to your github organization.

   b)update `git clone` command to point to ***your fork***.

6. :white_check_mark: Access Vertex AI Workbench and run cell by cell notebook `tpc-di-setup.ipynb`.

    a) in the first cell of the notebook replace: `%env DATA_BUCKET=tbd-2023z-9910-data` with your data bucket.
   
    b) after running first cells your fork of `tbd-tpc-di` repository will be cloned into Vertex AI  enviroment (see git folder).

    c) take a look on `git/tbd-tpc-di/profiles.yaml`. This file includes Spark parameters that can be changed if you need to increase the number of executors and
  ```
   server_side_parameters:
       "spark.driver.memory": "2g"
       "spark.executor.memory": "4g"
       "spark.executor.instances": "2"
       "spark.hadoop.hive.metastore.warehouse.dir": "hdfs:///user/hive/warehouse/"
  ```


7. Explore files created by generator and describe them, including format, content, total size.

   ***Files desccription***

8. Analyze tpcdi.py. What happened in the loading stage?

   ***Your answer***

9. :white_check_mark: Using SparkSQL answer: how many table were created in each layer?

   ***SparkSQL command and output***

   ```python
   # List of all databases (layers)
   databases_df = spark.sql("show databases")
   databases = [row.namespace for row in databases_df.collect()]

   # Initialize a dictionary to hold the count of tables in each layer
   layer_counts = {}

   #For each database
   for db_name in databases:
      # Use the database
      spark.sql(f"use {db_name}")
      
      # Get the list of all tables in the database
      tables = spark.sql("show tables")
      
      # The count of tables in the database is the count of tables in the layer
      layer_counts[db_name] = tables.count()

   Print the count of tables in each layer
   for layer, count in layer_counts.items():
      print(f'Layer: {layer}, Count: {count}')
      ```

      ![img.png](doc/figures/sql-tables.png)

10. Add some 3 more [dbt tests](https://docs.getdbt.com/docs/build/tests) and explain what you are testing. ***Add new tests to your repository.***

   ***Code and description of your tests***

11. :white_check_mark: Modify modules/data-pipeline/resources/dbt-dag.py and add new tasks to Apache Airflow DAG:
* that will execute `dbt run`
* that will execute dbt tests.

  ***The DAG code***

```python
   # Task to execute dbt run
  dbt_run_task = KubernetesPodOperator(
        task_id="dbt-run-task",
        name="dbt-run-task",
        image_pull_policy="Always",
        cmds=["bash", "-c"],
        arguments=["git clone https://github.com/mwiewior/tbd-tpc-di.git && cd tbd-tpc-di"
                "&& dbt run"],
        namespace="composer-user-workloads",
        image="eu.gcr.io/{{ var.value.project_id }}/dbt:1.7.3",
        config_file="/home/airflow/composer_kube_config",
        kubernetes_conn_id="kubernetes_default",
        container_resources={
            'request_memory': '2048M',
            'limit_memory': '4096M',
            'request_cpu': '800m',
            'limit_cpu': '1000m'
        }
    )

    # Task to execute dbt tests
    dbt_test_task = KubernetesPodOperator(
        task_id="dbt-test-task",
        name="dbt-test-task",
        image_pull_policy="Always",
        cmds=["bash", "-c"],
        arguments=["git clone https://github.com/mwiewior/tbd-tpc-di.git && cd tbd-tpc-di"
                "&& dbt test"],
        namespace="composer-user-workloads",
        image="eu.gcr.io/{{ var.value.project_id }}/dbt:1.7.3",
        config_file="/home/airflow/composer_kube_config",
        kubernetes_conn_id="kubernetes_default",
        container_resources={
            'request_memory': '2048M',
            'limit_memory': '4096M',
            'request_cpu': '800m',
            'limit_cpu': '1000m'
        }
    )

    # Adding dependencies
    kubernetes_min_pod >> dbt_run_task >> dbt_test_task 
```


12. Redeploy infrastructure and check if the DAG finished with no errors:

***The screenshot of Apache Aiflow UI***