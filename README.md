# Automating Data pipelines using Terraform

- Whenever we create a a New Project, Let it be Fictional or Real, We always try to reduce the friction of configuring the environment (installing dependencies, configuring folders, obtain credentials) and much more.

- However we can use Docker, by writing just a Docker compose.yaml and a few DockerFiles and one can create exactly the same environment with just one command `(docker compose up)`

- When one wants to develop a new data project with cloud based tools (S3, glue, lambda, EMR) Docker can't help in that case, as the components needs to be instantiated in the providers infrastructure, and there are two main ways of doing this: Manually on the UI or Using API's.

- For example: One can access AWS UI, Search for S3 and create a new Bucket or can write a code in python to create this same instance making a request on the AWS API.

- And this is the exactly kind of problem that Terraform comes to solve.

Read more about Terraform : https://developer.hashicorp.com/terraform/intro

# Implementation

![architecture](../src/terraform-data-pipeline.png)

- This Project aims to automate the process of infrastructure creation.

- The process involved three steps, controlled by a local Airflow instance. These steps included downloading and uploading the PDF file to S3 storage, extracting texts from the PDFs using a Lambda function, and segmenting the extracted text into questions using a Glue Job.

# Setting Up the Environment

- Create An IAM user for Terraform - Add the following policies `(AmazonS3FullAccess, AWSGlueConsoleFullAccess, AWSLambda_FullAccess, IAMFullAccess)`
- In local Environment, on the same path as of Docker-compose.yaml file, there's a `(.env)` file, add your AWS Credentials.
- These variables will be passed to the docker-compose.yaml file to be used by terraform.
- In `(main.tf)` file, the first phase will be to configure the cloud Provider.
- This is what a Terraform configuration file looks like — a set of blocks with different types, each one with a specific function.
- The terraform block fixes the versions for Terraform itself and for the AWS provider.
- A variable is exactly what the name suggests — a value assigned to a name that can be referenced throughout the code. 

# Note: 
    - There is no value being assigned to the variable, this is because, in the docker-compose.yaml file, the value of these variables was set using environment variables in the system. When a variable value is not defined, Terraform will look at the value of the environment variable TF_VAR_<var_name> and use its value. I’ve opted for this approach to avoid hard-coding the keys.

- Run the following command to Setup Terraform - `(docker compose run terraform init)`
- Next Is Creating S3 Bucket - use Command `(docker compose run terraform apply)`, to create an S3 Bucket.
- Similarly confiure the Lambda Function - that creates a new IAM role and allows AWS Lambda Functions to assume it. The next step attaches the Lambda Basic Execution policy to this role to allow the Lambda Function to execute without errors.
- Creates a new access policy for our S3 bucket, allowing basic CRUD operations on it.
- Again, instead of hard-coding the bucket’s ARN, one can reference this attribute using *aws_s3_bucket.terraform-job-data-bucket.arn*.
- The lambda_function.zip file is a compressed folder that must have a lambda_function.py file with a lambda_handler(event, context) function inside. It must be on the same path as the main.tf file.
- The next phase configures another lambda function to attach a trigger - It must trigger everytime a new data is uploaded to the S3 bucket.
- This is a case where one must specify an explicit dependency between resources, as the *bucket_notification* resource needs to be created after the *allow_bucket_execution*.
- This can be easily achieved by using the *depends_on* argument.
- Adding a module to the Glue Job - In the ./terraform folder, there is a folder ‘glue’ with a glue.tf file inside it.
- Re-initialize terraform using `(docker compose run terraform init)`- Further Used the Plan and apply command.

# Delete Project
- Delete the project to Save the cost - Run `(docker compose run terraform destroy)`

# Key TakeAways:
1. With modules and arguments, we can create fully parametrized complex infrastructures.
2. The code above doesn’t just create a specific job for our pipeline. By just changing the value of the terraform-job-data-bucket-access-policy-arn variable, one can create a new job to process data from an entirely different bucket.
3. It’s possible, for example, to simultaneously create a complete infrastructure for a project for the development, testing, and production environments, using just variables to alternate between them.