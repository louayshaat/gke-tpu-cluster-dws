## Create a cluster
These steps guide you through the cluster creation process.

Note: If you create multiple clusters using these same cluster blueprints, ensure that all VPCs and subnet names are unique per project to prevent errors.

1. Launch [Cloud Shell](https://cloud.google.com/shell/docs/launching-cloud-shell). You can use a different environment; however, we recommend Cloud Shell because the dependencies are already pre-installed for Cluster Toolkit. If you don't want to use Cloud Shell, follow the [instructions to install dependencies](https://cloud.google.com/cluster-toolkit/docs/setup/install-dependencies) to prepare a different environment.

1. Clone the Cluster Toolkit from the git repository:

    ```sh
    cd ~
    git clone https://github.com/GoogleCloudPlatform/cluster-toolkit.git
    ```

1. Install the Cluster Toolkit:

    ```sh
    cd cluster-toolkit && git checkout main && make
    ```

1. Create a Cloud Storage bucket to store the state of the Terraform deployment:

    ```sh
    gcloud storage buckets create gs://BUCKET_NAME \
        --project=PROJECT_ID \
        --default-storage-class=STANDARD \
        --location=COMPUTE_REGION \
        --uniform-bucket-level-access
    gcloud storage buckets update gs://BUCKET_NAME --versioning
    ```

    Replace the following variables:\
    BUCKET_NAME: the name of the new Cloud Storage bucket.\
    PROJECT_ID: ID of the project where the bucket is being created.\
    COMPUTE_REGION: the compute region where you want to store the state of the Terraform deployment.


1. In the `gke-tpu-7x-deployment.yaml` file, fill in the following settings in the terraform_backend_defaults and vars sections to match the specific values for your deployment:

    `bucket`: the name of the Cloud Storage bucket you created in the previous step.
    
    `deployment_name`: the name of the deployment.
   
    `project_id`: your Google Cloud project ID.
   
    `region`: the compute region for the cluster.
   
    `zone`: the compute zone for the node pool of TPU 7x machines or TPU 6e.
   
    **`enable_flex_start`**: set to `true` to enable DWS Flex Start.
   
    **`autoscaling_min_node_count`**: set to `0` (required for Flex Start).
   
    **`autoscaling_max_node_count`**: set to the required node count for your topology (e.g., `2` for a `2x2x2` topology).
   
    `authorized_cidr`: The IP address range that you want to allow to connect with the cluster.
   
   
    To modify advanced settings, edit `gke-tpu-7x.yaml`.

1. Generate [Application Default Credentials (ADC)](https://cloud.google.com/docs/authentication/provide-credentials-adc#google-idp) to provide access to Terraform.

    ```sh
    gcloud auth application-default login
    ```

1. Deploy the blueprint to provision the GKE infrastructure using TPU 7x machine types:

    ```sh
    cd ~/cluster-toolkit
    ./gcluster deploy -d \
    gke-tpu-7x-deployment.yaml \
    gke-tpu-7x.yaml
    ```

1. When prompted, select (A)pply to deploy the blueprint.

## Note

* DWS Flex Start does not work with static nodes. So, static_node_count cannot be set.
* To use DWS Flex Start, `auto_repair` should be set to `false`.

## Running Workloads with DWS Flex Start

The Cluster Toolkit automatically generates a pre-configured, DWS-compliant job file located in your deployment folder (e.g., `gke-tpu-7x-flex/primary/my-job-xxxx.yaml`).
