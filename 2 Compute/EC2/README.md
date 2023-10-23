- Elastic Compute Cloud (EC2)
    
    

    
    - Regional Service
    - EC2 (Elastic Compute Cloud) is an **Infrastructure as a Service (IaaS)**
    - Stopping & Starting an instance may change its ***public IP but not its private IP***
    - **AWS Compute Optimizer** recommends optimal AWS Compute resources for your workloads
    - There is a vCPU-based On-Demand Instance soft limit per region
    
    ## User Data
    
    - Some commands that run when the instance is launched for the first time (doesn't execute for subsequent runs)
    - Used to automate **dynamic** boot tasks (that cannot be done using AMIs)
        - Installing updates
        - Installing software
        - Downloading common files from the internet
    - Runs with the **root user privilege**
    - **userdata script**
        
        Amazon Linux 2 user data script:
        
        `#!/bin/bash -ex
        wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip
        unzip FlaskApp.zip
        cd FlaskApp/
        yum -y install python3 mysql
        pip3 install -r requirements.txt
        amazon-linux-extras install epel
        yum -y install stress
        export PHOTOS_BUCKET=${SUB_PHOTOS_BUCKET}
        export AWS_DEFAULT_REGION=<INSERT REGION HERE>
        export DYNAMO_MODE=on
        FLASK_APP=application.py /usr/local/bin/flask run --host=0.0.0.0 --port=80`
        
    
    ## Instance Classes
    
    - **General Purpose**
        - Great for a diversity of workloads such as **web servers** or **code repositories**
        - Balance between compute, memory & networking
    - **Compute Optimized**
        - Great for compute intensive tasks
            - Batch Processing
            - Media Transcoding
            - HPC
            - Gaming Servers
    - **Memory Optimized**
        - Great for **in-memory databases** or **distributed web caches**
    - **Storage Optimized**
        - Great for storage intensive tasks (accessing local databases)
            - OLTP(Online Transaction Processing) systems
            - Distributed File System (DFS)
    
    ## Security Groups
    
    - ***Only contain Allow rules***
    - External firewall for EC2 instances (if a request is blocked by SG, instance will never know)
    - Security groups rules can reference a resource by IP or Security Group
    - Default SG
        - *inbound traffic from the same SG is allowed*
        - all outbound traffic is allowed
    - New SG
        - all inbound traffic is blocked
        - all outbound traffic is allowed
    - ***A security group can be attached to multiple instances and vice versa***
    - Bound to a VPC (and hence to a region)
    - Recommended to maintain a separate security group for SSH access
    - Blocked requests will give a **Time Out** error
    
    ## IAM Roles for EC2 instances👇
    
    - Never enter AWS credentials into the EC2 instance, instead attach [**IAM Roles]** to the instances ie IAM ReadOnly access
    
    ## Purchasing Options
    
    ### **On-demand Instances**
    
    - Pay per use (no upfront payment)
    - Highest cost
    - No long-term commitment
    - Recommended for short-term, uninterrupted and **unpredictable** workloads
    
    ### **Standard Reserved Instances**
    
    - ***Reservation Period: 1 year or 3 years***
    - Recommended for steady-state applications (like database)
    - 👉**Sell unused instances** on the Reserved Instance Marketplace
    
    ### **Convertible Reserved Instances**
    
    - *Can change the instance type*
    - Lower discount
    - 👉**Cannot sell unused instances** on the Reserved Instance Marketplace
    
    ### **Scheduled Reserved Instances**
    
    - reserved for a time window (ex. everyday from 9AM to 5PM)
    
    ### **Spot Instances**
    
    - Work on a bidding basis where you are willing to pay a specific max hourly rate for the instance. Your instance can terminate if the spot price increases.
    - **Spot blocks** are designed not to be interrupted
    - Good for workloads that are resilient to failure
        - Distributed jobs (resilient if some nodes go down)
        - Batch jobs
    
    ### **Dedicated Hosts**
    
    - ***Server hardware is allocated to a specific company (not shared with other companies)***
    - **3 year reservation** period
    - Billed per host
    - 👉Useful for software that have **BYOL (Bring Your Own License)** or for companies that have strong regulatory or compliance needs
    
    ### **Dedicated Instances**
    
    - Dedicated hardware
    - Billed per instance
    - ***No control over instance placement***
    
    ### **On-Demand Capacity Reservations**
    
    - ***Ensure you have the available capacity in an AZ to launch EC2 instances when needed***
    - Can reserve for a recurring schedule (ex. everyday from 9AM to 5PM)
    - No need for 1 or 3-year commitment (independent of billing discounts)
    - Need to specify the following to create capacity reservation: - AZ - Number of instances - Instance attributes
    - **Reserved Capacity & Instances Comparison**
        
        !https://notes.arkalim.org/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220516114146.jpg
        

    
    ## `Spot Instances
    
    ### **Spot Requests**
    
    - ***One-time**: Request once opened, spins up the spot instances and the request closes.*
    - ***Persistent**:*
        - Request will stay disabled while the spot instances are up and running.
        - It becomes active after the spot instance is interrupted.
        - If you stop the spot instance, the request will become active only after you start the
        - spot instance.
    - You can only cancel spot instance requests that are open, active, or disabled.
    - *Cancelling a Spot Request does not terminate instances. You must first cancel a Spot Request, and then terminate the associated Spot Instances.*
    
    ### **Spot Fleets👇**
    

    
    - ***Combination of spot and on-demand instances (optional) that tries to optimize for cost or capacity***
    - **Launch Templates must be used to have on-demand instances in the fleet**
    - Can consist of instances of different classes
    - Strategies to allocate Spot Instances:
        - L**owest Price** - from the pool with the lowest price (cost optimization, short workload)
        - **diversified** - distributed across all pools (great for availability, long workloads)
        - C**apacity Optimized** - pool with the optimal capacity for the number of instances
        
        ## Private vs Public IP (IPv4)
        
    
    ```
    
    ▶️ **By default, your EC2 machine comes with:
    
    • A private IP for the internal AWS Network
    
    • A public IP,fortheWWW.
    
    • When we are doing SSH into our EC2 machines:We can't use a private IP because we  are not in the same network Hence, We can only use the public IP
    
    • If your machine is stopped and then started,the public IP can change**
    ```
    
    ## Elastic IP
    
    - **Static Public IP** that you own as long as you don't delete it
    - Can be attached to an EC2 instance (even when it is stopped)
    - ***Soft limit of 5 elastic IPs per account***
    - Doesn’t incur charges as long as the following conditions are met (EIP behaving like any other public IP randomly assigned to an EC2 instance):
        - ***The Elastic IP is associated with an Amazon EC2 instance***
        - ***The instance associated with the Elastic IP is running***
        - ***The instance has only one Elastic IP attached to it***
    
    ## Placement Groups (Placement Strategies)
    
    - **Cluster Placement Group (optimize for network)**
        - ***All the instances are placed on the same hardware (same rack)***
        - Pros: Great network (10 Gbps bandwidth between instances)
        - *Used in **HPC** (minimize inter-node latency & maximize throughput)*
        - Image
            - 
                
               
                
    - **Spread Placement Group (maximize availability)**
        - ***Each instance is in a separate rack (physical hardware) inside an AZ***
        - Supports Multi AZ
            - 👉Up to 7 instances per AZ per placement group (ex. for 15 instances, need 3 AZ)
        - *Used for critical applications*
        
        
        
    - **Partition Placement Group (balance of performance and availability)**
        - ***Instances in a partition share rack with each other***
        - If the rack goes down, the entire partition goes down
        - 👉***Up to 7 partitions per AZ***
        - ***Used in big data applications (Hadoop, HDFS, HBase, Cassandra, Kafka)***
        - Image
            - 
                
              
                
    
    > ***If you receive a capacity error when launching an instance in a placement group that already has running instances, stop and start all of the instances in the placement group, and try the launch again. Restarting the instances may migrate them to hardware that has capacity for all the requested instances.***
    > 
    
    ## Elastic Network Interface (ENI)
    
    - ***ENI is a virtual network card that gives a private IP to an EC2 instance***
    - ***A primary ENI is created and attached to the instance upon creation and will be deleted automatically upon instance termination.***
    - We can create additional ENIs and attach them to an EC2 instance to access it via multiple private IPs.
    - ***We can detach & attach ENIs across instances***
    - **ENIs are tied to the subnet** (and hence to the AZ)
    
    ## Instance States
    
    - **Stop**
        - EBS root volume is preserved
    - **Terminate**
        - EBS root volume gets destroyed
    - **Hibernate**
        - 👉 ***Hibernation saves the contents from the instance memory (RAM) to the EBS root volume***
        - ***EBS root volume is preserved***
        - **👉T*he instance boots much faster as the OS is not stopped and restarted***
        - *When you start your instance:👇*
            - *EBS root volume is restored to its previous state*
            - *RAM contents are reloaded*
            - *Processes that were previously running on the instance are resumed*
            - *Previously attached data volumes are reattached and the instance retains its instance ID*
        - Should be used for applications that take a long time to start
        - **Not supported for Spot Instances**
        - Max hibernation duration = ***60 days***
    - **Standby**
        - Instance remains attached to the [ASG](https://notes.arkalim.org/notes/aws%20solutions%20architect%20associate/Auto%20Scaling%20Group%20(ASG)) but is temporarily put out of service (the ASG doesn't replace this instance)
        - Used to install updates or troubleshoot a running instance
    
    ## EC2 Nitro
    
    - Newer virtualization technology for EC2 instances
    - Better networking options (enhanced networking, HPC, IPv6)
    - Higher Speed EBS (64,000 EBS IOPS max on Nitro instances whereas 32,000 on non-Nitro)
    - Better underlying security
    
    ## vCPU & Threads
    
    - vCPU is the total number of concurrent threads that can be run on an EC2 instance
    - Usually 2 threads per CPU core (eg. 4 CPU cores ⇒ 8 vCPU)
    
    ## Storage
    
    [Instance Store](https://notes.arkalim.org/notes/aws%20solutions%20architect%20associate/Instance%20Store)[Elastic Block Storage (EBS)](https://notes.arkalim.org/notes/aws%20solutions%20architect%20associate/Elastic%20Block%20Storage%20(EBS))[Elastic File System (EFS)](https://notes.arkalim.org/notes/aws%20solutions%20architect%20associate/Elastic%20File%20System%20(EFS))
    
    ## Monitoring
    
    [CloudWatch ⮕ EC2 Monitoring](https://notes.arkalim.org/notes/aws%20solutions%20architect%20associate/CloudWatch#ec2-monitoring)
    
    ## Amazon Machine Image (AMI)
    
    - AMIs are the image of the instance after installing all the necessary OS, software and configuring everything.
    - It boots much faster because the whole thing is pre-packaged and doesn’t have to be installed separately for each instance.
    - Good for static configurations
    - **Bound to a region** (can be copied across regions)
    
    > When the new AMI is copied from region A into region B, it automatically creates a snapshot in region B because AMIs are based on the underlying snapshots.
    > 
    
    ## Patching of ec2 instance
    
    - ec2 patching
        1. **Create an IAM role and attach the AmazonEC2RoleforSSM Managed policy.**
            
            Go to the IAM console and create a new role. In the **Role name** field, enter a name for your role. In the **Select permissions** section, select the **AmazonEC2RoleforSSM** Managed policy. Click **Create role**.
            
        2. **Install the SSM Agent on your EC2 instances.**
            
            The SSM Agent is a lightweight agent that communicates with Systems Manager. You can install the SSM Agent on your EC2 instances using the following steps:
            
            - Connect to your EC2 instance using SSH.
            - Run the following command to install the SSM Agent:
                
                `sudo yum install -y amazon-ssm-agent`
                
            - Restart your EC2 instance.
        3. **Create a custom patch baseline.**
            
            A patch baseline defines the patches that you want to install on your EC2 instances. You can create a custom patch baseline using the following steps:
            
            - Go to the Systems Manager console and select **Patch Manager**.
            - In the **Patch baselines** tab, click **Create patch baseline**.
            - In the **Baseline name** field, enter a name for your patch baseline.
            - In the **Patch selection** section, select the patches that you want to include in your patch baseline.
            - Click **Create**.
        4. **Set the patch group for the custom patch baseline.**
            
            A patch group is a collection of EC2 instances that you want to patch. You can set the patch group for the custom patch baseline using the following steps:
            
            - Go to the Systems Manager console and select **Patch Manager**.
            - In the **Patch groups** tab, click **Create patch group**.
            - In the **Patch group name** field, enter a name for your patch group.
            - In the **Patch baseline** field, select the custom patch baseline that you created in the previous step.
            - In the **Instances** section, select the EC2 instances that you want to include in your patch group.
            - Click **Create**.
        5. **Create a maintenance window.**
            
            A maintenance window is a scheduled time when Systems Manager can perform tasks, such as patching your EC2 instances. You can create a maintenance window using the following steps:
            
            - Go to the Systems Manager console and select **Maintenance Windows**.
            - Click **Create maintenance window**.
            - In the **Maintenance window name** field, enter a name for your maintenance window.
            - In the **Schedule** section, select the time and frequency for your maintenance window.
            - In the **Tasks** section, select the task that you want to run during your maintenance window. In this case, you would select the **Run Patch Baseline** task.
            - Click **Create**.
        6. **Register your EC2 instances for the maintenance window.**
            
            This will allow Systems Manager to patch your EC2 instances during the maintenance window. You can register your EC2 instances for the maintenance window using the following steps:
            
            - Go to the Systems Manager console and select **Maintenance Windows**.
            - In the **Registered targets** tab, click **Register targets**.
            - In the **Target type** field, select **EC2 instance**.
            - In the **Target IDs** field, enter the IDs of the EC2 instances that you want to register for the maintenance window.
            - Click **Register**.
        7. **Verify the patch compliance report.**
            
            This report will show you the status of the patches on your EC2 instances. You can verify the patch compliance report using the following steps:
            
            - Go to the Systems Manager console and select **Patch Manager**.
            - In the **Patch compliance** tab, click **View report**.
    
    ## Instance Metadata
    
    - Url to fetch metadata about the instance (http://169.254.169.254/latest/meta-data)
    - This URL is internal to AWS and can only be hit from the instance
    
    ## EC2 Classic & ClassicLink
    
    - Instances run in single network shared with other customers (this is how AWS started)
    - **ClassicLink** allows you to link EC2-Classic instances to a VPC in your account
    
    ## Billing
    
    - **Reserved instances will be billed regardless of their state** (billed for a reserved period)
    - **On-demand instances in `stopping` state when preparing to hibernate will be billed**
    - If an instance is running, it will be billed
    - In all the other cases, an instance will not be billed
    
    ## Run Command
    
    - Systems Manager **Run Command** lets you remotely and securely **manage the configuration** of your **managed instances**. A *managed instance* is any EC2 instance that has been configured for **Systems Manager**.
    - Run Command enables you to **automate common administrative tasks** and perform ad-hoc configuration changes at scale.
    - You can use Run Command from the **AWS Console**, the AWS CLI, AWS Tools for Windows PowerShell, or the AWS SDKs. Run Command is offered at no additional cost.
    
    ## Instance Tenancy
    
    - **Default**: Instance runs on shared hardware
    - **Dedicated**: Instance runs on single-tenant hardware
    - **Host**: Instance runs on dedicated host
    
    > Tenancy of an instance can only be changed from host to dedicated or dedicated to host after the instance has been launched. Dedicated instance tenancy takes precedence over Default instance tenancy
    > 
    
    ## Troubleshooting
    
    - The following are a few reasons why an instance might immediately terminate:
        - *`You’ve reached your EBS volume limit.`*
        - *`An EBS snapshot is corrupt.`*
        - *`The root EBS volume is encrypted and you do not have permissions to access the KMS key for decryption.`*
        - *`The instance store-backed AMI that you used to launch the instance is missing a required part.`*
