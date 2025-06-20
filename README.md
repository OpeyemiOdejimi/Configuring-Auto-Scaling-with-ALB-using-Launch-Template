# Mini Project - Creating AWS resources with functions & introducing Arrays
We'll begin this project by creating two functions:

One for provisioning EC2 instances and
Another for setting up S3 buckets.
These functions will streamline the process of resource creation and enable us to automate tasks effectively.

Function to provision EC2 instances
To programmatically create EC2 instances, you must use the official documentation to understand how to use the aws cli to create instances.

From the Available Commands, you will be able to interact with AWS programmatically.

Alt text

If you search for run-instances on the page with Control F on your key board, click on it and it will take you to the detailed documentation on the sub-command to create EC2 instances.

Alt text

Here is an example of how you would create EC2 instances using the command line
```
    aws ec2 run-instances \
        --image-id "ami-0cd59ecaf368e5ccf" \
        --instance-type "t2.micro" \
        --count 5 \
        --key-name MyKeyPair \
        --region eu-west-2

```

> Note: Make sure you have a key pair created in your aws console. The you can replace the MyKeyPair with you key pair name.

On the same page, if you search for one of the arguments, you will be able to read more about how to pass different arguments to the cli.

Alt text

For the command to work: A keypair must already exist. You must create a keypair from the console. To create a new key pair.

Navigate to the AWS EC2 console

Follow the image below to create a key pair

Alt text

Now, lets update the shell script and create a function that will be responsible for creating EC2 instances.

```
#!/bin/bash

# Function to create EC2 instances

create_ec2_instances() {"\n\n    # Specify the parameters for the EC2 instances\n    instance_type=\"t2.micro\"\n    ami_id=\"ami-0cd59ecaf368e5ccf\"  \n    count=2  # Number of instances to create\n    region=\"eu-west-2\" # Region to create cloud resources\n    \n    # Create the EC2 instances\n    aws ec2 run-instances \\\n        --image-id \"$ami_id\" \\\n        --instance-type \"$instance_type\" \\\n        --count $count\n        --key-name MyKeyPair\n        \n    # Check if the EC2 instances were created successfully\n    if [ $? -eq 0 ]; then\n        echo \"EC2 instances created successfully.\"\n    else\n        echo \"Failed to create EC2 instances.\"\n    fi\n"}

# Call the function to create EC2 instances
create_ec2_instances

```

Lets highlight some new areas

$?: This is a special variable that holds the exit status of the last executed command. In this case, it checks if the aws ec2 run-instances command was successful. exit status that equals 0 is interpreted as succesful. Therefore if exit code is "0", then echo the message to confirm that the previous command was successful.

We have once again used environment variables to hold the value of ami_id, count and region and replaced with their respective values with 
a
m
i
i
d
,
ami 
i
​
 d,count and $region

Define function to create S3 buckets & learn about Arrays
Before diving into creating S3 buckets, it's beneficial to brush up on AWS S3 bucket concepts. If you need a quick refresher, consider watching this informative video.

The AWS CLI reference for S3 can be found here. We will be using it in the script.

In this section, our objective is to create five distinct S3 buckets, each designated for storing data related to Marketing, Sales, HR, Operations, and Media.

To achieve this, we'll utilize a fundamental data structure in shell scripting known as an "array." this is because, we need one single variable holding all the data, and then have the capability to loop through them.

Arrays in Shell Scripting

An array is a versatile data structure that allows you to store multiple values under a single variable name. Particularly in shell scripting, arrays offer an efficient means of managing collections of related data, making them invaluable for our task ahead.

Below is what the function woould look like.

```
# Function to create S3 buckets for different departments
create_s3_buckets() {"\n    # Define a company name as prefix\n    company=\"datawise\"\n    # Array of department names\n    departments=(\"Marketing\" \"Sales\" \"HR\" \"Operations\" \"Media\")\n    \n    # Loop through the array and create S3 buckets for each department\n    for department in \"${departments[@]"}"; do
        bucket_name="${company}-${department}-Data-Bucket"
        # Create S3 bucket using AWS CLI
        aws s3api create-bucket --bucket "$bucket_name" --region your-region
        if [ $? -eq 0 ]; then
            echo "S3 bucket '$bucket_name' created successfully."
        else
            echo "Failed to create S3 bucket '$bucket_name'."
        fi
    done
}

# Call the function to create S3 buckets for different departments
create_s3_buckets

```

Lets break down each part of the code.

This line begins the definition of a shell function named create_s3_buckets.

```
# Function to create S3 buckets for different departments
create_s3_buckets() {"\n    ```\n- Here, we define a variable named company and assign it the value \"datawise\". This variable will serve as the prefix for all S3 bucket names, ensuring their uniqueness. You should replace **datawise** with any other company name of your choice or any other unique identifier. This is because S3 buckets **MUST** be unique globally.\n  \n    ```\n    # Define a company name as prefix\n    company=\"datawise\"\n\n    ```\n\n- This is where we define a variable that is an array. An array named **departments** is declared, containing the names of different departments. Each department name will be used to construct the name of an S3 bucket.\n  ```\n    # Array of department names\n    departments=(\"Marketing\" \"Sales\" \"HR\" \"Operations\" \"Media\")\n  ```\n\n- This line initiates a loop that iterates over each element in the departments array. For each iteration, the value of the current department name is stored in the variable **department**.\n\n    ```\n    for department in \"${departments[@]"}"; do

```
```
    The syntax ${departments[@]} in Bash refers to all elements in the array departments.

    [@]: This is an index or slice syntax specific to arrays in Bash. It signifies that we want to access all elements of the array.

    If you were interested in accessing a single element from the array, you would still use the syntax $departments[index]}, where index is the position of the element you want to access. Remember that array indexing in Bash starts from 0.

```
An example is;
```
departments=("Marketing" "Sales" "HR" "Operations" "Media")

# Accessing the fourth element (Operations) from the array
echo "${departments[3]}"

```

Output;

```
Operations

```
Within the loop, we construct the name of the S3 bucket using the company prefix, the current department name, and the suffix "-Data-Bucket". This ensures that each bucket name is unique.

```
bucket_name="${company}-${department}-Data-Bucket"

```
Using the AWS CLI (aws s3api), we create an S3 bucket with the specified name from the variable ($bucket_name), in the specified AWS region (your-region). Make sure to replace "your-region" with the actual AWS region where you want to create the buckets.

```
    # Create S3 bucket using AWS CLI
    aws s3api create-bucket --bucket "$bucket_name" --region your-region

```

This line checks the exit status of the previous command (aws s3api create-bucket). A value of 0 indicates success, while non-zero values indicate failure.

```
    if [ $? -eq 0 ]; then

```

Based on the exit status of the aws s3api command, we print a corresponding message indicating whether the bucket creation was successful or not.

```
        echo "S3 bucket '$bucket_name' created successfully."
        else
            echo "Failed to create S3 bucket '$bucket_name'."
        fi
    done

```
This line marks the end of the create_s3_buckets function definition.
```
 }

```

**The complete script till this stage will now look like this.**

```
#!/bin/bash

# Environment variables
ENVIRONMENT=$1

check_num_of_args() {"\n# Checking the number of arguments\nif [ \"$#\" -ne 0 ]; then\n    echo \"Usage: $0 <environment>\"\n    exit 1\nfi\n"}

activate_infra_environment() {"\n# Acting based on the argument value\nif [ \"$ENVIRONMENT\" == \"local\" ]; then\n  echo \"Running script for Local Environment...\"\nelif [ \"$ENVIRONMENT\" == \"testing\" ]; then\n  echo \"Running script for Testing Environment...\"\nelif [ \"$ENVIRONMENT\" == \"production\" ]; then\n  echo \"Running script for Production Environment...\"\nelse\n  echo \"Invalid environment specified. Please use 'local', 'testing', or 'production'.\"\n  exit 2\nfi\n"}

# Function to check if AWS CLI is installed
check_aws_cli() {"\n    if ! command -v aws &> /dev/null; then\n        echo \"AWS CLI is not installed. Please install it before proceeding.\"\n        return 1\n    fi\n"}

# Function to check if AWS profile is set
check_aws_profile() {"\n    if [ -z \"$AWS_PROFILE\" ]; then\n        echo \"AWS profile environment variable is not set.\"\n        return 1\n    fi\n"}

# Function to create EC2 Instances
create_ec2_instances() {"\n\n    # Specify the parameters for the EC2 instances\n    instance_type=\"t2.micro\"\n    ami_id=\"ami-0cd59ecaf368e5ccf\"  \n    count=2  # Number of instances to create\n    region=\"eu-west-2\" # Region to create cloud resources\n    \n    # Create the EC2 instances\n    aws ec2 run-instances \\\n        --image-id \"$ami_id\" \\\n        --instance-type \"$instance_type\" \\\n        --count $count\n        --key-name MyKeyPair\n        \n    # Check if the EC2 instances were created successfully\n    if [ $? -eq 0 ]; then\n        echo \"EC2 instances created successfully.\"\n    else\n        echo \"Failed to create EC2 instances.\"\n    fi\n"}

# Function to create S3 buckets for different departments
create_s3_buckets() {"\n    # Define a company name as prefix\n    company=\"datawise\"\n    # Array of department names\n    departments=(\"Marketing\" \"Sales\" \"HR\" \"Operations\" \"Media\")\n    \n    # Loop through the array and create S3 buckets for each department\n    for department in \"${departments[@]"}"; do
        bucket_name="${company}-${department}-Data-Bucket"
        # Create S3 bucket using AWS CLI
        aws s3api create-bucket --bucket "$bucket_name" --region your-region
        if [ $? -eq 0 ]; then
            echo "S3 bucket '$bucket_name' created successfully."
        else
            echo "Failed to create S3 bucket '$bucket_name'."
        fi
    done
}

check_num_of_args
activate_infra_environment
check_aws_cli
check_aws_profile
create_ec2_instances
create_s3_buckets

```
