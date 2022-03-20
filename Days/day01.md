# EC2

- Is a web service that provides resizable compute capacity in the cloud. It reduces the time required to obtain and boot new server instances to minutes, allowing you to quickly scale capacity, both up and down, as your computing requirements change.
- **Pricing Models**
    1. **On Demand**
        - A fixed rate by the hour (or by the second) with no commitment. 
        - Low cost and flexibility with no upfront cost
        - Ideal for auto scaling groups and unpredictable workloads
        - Applications being developed or tested on EC2 for the first time
    2. **Reserver**
        - Purchase or commit to usgae of EC2 instances in advance for significant discounts over On-Demand pricing
        - Capacity is reserver for a term of 1 or 3 years
        - AWS Biling automatically applies discounted rates when you launch an instance that maches your purchase RI
        - Use for applications:
            - Application with steady state or predictable usage
            - Applications that require reserved capacity
            - Upfront payments reduce the hourly rate even more
            - Can be used in Auto Scaling Groups
            - Can be used in Placement Groups
            - Can be shared across multiple accounts within **Consolidated Biling**
        - **Three types**
            1. **Standard** - Commitment of 1 or 3 years, charged whether it's on or off. Offer up to 75% off demand instances.
            2. **Convertible** - offer up to 54% off on demand capability to change the attributes of the RI as long as the exchange results in the creation of Reserver Instances of equal or greater value
            3. **Scheduled** - reserved for specific periods of time, accrue charges hourly, billed in monthly increments over the term (1 year)
        - __Reserved Instances that are terminated__ are **still billed** until the end of their term according to their payment option.
        - *Note* - if it is scheduled to run only a short period, it might not be efficient to use reserved instances, because you need to pay for the whole year
    3. **Spot** 
        - Take advantage of unused EC2 capacity in the AWS cloud
        - Spot instances are availbale at up to  a 90% discount compared to On-Demand prices
        - Use for stateless, fault-tolerant, flexiable applications such as big data, containerized workloads, CI/CD, web servers, HPC
        - Price is determined by long term trends in supply and demand for EC2 spare capacity
            - Pay the Spot price that's in effect for the current hour for the instances that you launch
            - Receive two-minute interruption notice when these instanes are to be reclaimed by EC2, because EC2 needs the capacity back
            - Instances are not interrupted because of higher competing bids
        - To reduce the impact of interruptions, difersify and run your application across multiple capacity pools
            - Can also stop or hibernate rather than terminate instnaces when capacity is no longer available 
        - *Note*. If the spot instace is terminated by EC2, you will not be charged for a partial hour of usage. However, if you terminat the instance yourself, you will be charged for any hour in which the instance ran
    4. **Dedicated Hosts**
        - Physical servers dedicated jsut for your use
        - Available On-Demand and Reserved pricing models
        - Useful if you have server-bound software licenses
        - Good for regulatory compliance or licensing requirements
        - Predictable performance
        - Complete isolation
        - Most expensive option
        - Have control over number of cores
    5. **Dedicated Instances**
        - Virtualized instances on hardware just for you
        - Uses physically dedicated EC2 servers
        - Does not provide the additional visibility and controls of dedicated hosts
        - May share hardware with other non-dedicated instances in the same account
        - Available as On-Demand, Reserver instances, and Spot instances
- You are limited to running up to total of 20 On-Demand instances across the instance family, purchasing 20 Reserved instances
- Linux instances are charged by seconds, all other instances by hour
- Data between instances in different regions is charged (in and out)
- **On-Demand Capacity Reservation** and **Zonal Reserved Instances** provide a capacity reservation for EC2 instances in a specific AZ


## Security groups
- All Inbound traffic is blocked by default
- All Outbound traffic is allowed
- Changes to Security groups take immediately
- You can have multiple EC2 on the same security group and vice versa
- Security groups are stateful - if you create an inbound rule allowing traffic in, that traffic automatically allowed as outbound
- You cannot block specific IP using security groups, only Network Access Control List
- You can only specify allow rules, not deny

## AMI
Can be selected on
- Region
- OS
- Architecture (32-bit or 64-bit)
- Launch Permissions
- Storage for the Root device
    1. Instance Store (Emhemeral storage) - cannot be stopped. If underlying host fails, you will lose your data
    2. EBS backed volumes - can be stopped. Will not lose your data if instace is stopped
- By default both ROOT volumes will be deleted on EC2 termination. However, with EBS volumes, you can tell AWS to keep the root device volume
- You can't delete a snapshot of the root device of an EBS volume used by a registered AMI. You must first deregister the AMI before you can delete the snapshot.

## ENI, ENA, EFA

- **ENI - elastic network interface**
    - Essentially a virtual network card. Choose for basic networking, or manage multiple networks at low cost. Can include:
        1. Primary private IPv4 address from the IPv4 address range of your VPC
        2. One or more secondary private IPv4 addresses from the IPv4 address range of your VPC
        3. One Elastic IP address (IPv4) per private IPv4 address
        4. One public IPv4 address
        5. One or more IPv6 addresses
        6. One or more security groups
        7. A mac address
        8. A source/destination check flag
        9. Description
    - You can attach a network interface (ENI) to an EC2 instance in the following ways —
        1. When it's running. **Hot attach**
        2. When it's stopped. **Warm attach**
        3. When the instance is being launched. **Cold attach**
    - **eth0** is the primary network interface and cannot be moved or detached.
        - By default, it is the only ENI created with an EC2 instance when launched
    - ENI is bound to an AZ, you can specify which subnet/AZ you want the ENI to be added in
    - Default interfaces are terminated with instance termination
        - Manually added interfaces are not terminated by default
        - You can change termination behavior
- **EN - Enhanced networking**
    - Uses single root I/O virtualization to provide high-performance networking capabilities on supported instance types
        1. Provides higher I/O performance and lower CPU utilization when compared to traditional virtualised network interface. Provides higher bandwidth, packet per second performance and lower inter-instance latencies. There is no additional charge for using enhanced networking, but EC2 must support it.
        2. To use where you want good network performance
        3. Can be enabled using:
            - *Elastic Network Adapter (ENA)* which supports network speeds of up to 100Gbps for supported instance types. Always choose this method
            - *Virtual Function (VF)* interface - suports network speeds of up to 10Gbps for supported instance types. This is typically used on older instances. 
        4. Choose for when you need speeds between 10Gbps and 100Gbps and reliable and high throughput.
- **Elastic Fabric Adapter* *
    - A network device that you can attach to your EC2 instance to accelerate High Performance Computing and ML applications
    - Choose for when you need to accelerate High Performance Comouting (HPC) and ML applications or if you need to do an OS by-pass. 


## EC2 placement groups
- When you launch a new EC2 instance, the EC2 service **attempts to place the instance in such a way that all of your instances are spread out across underlying hardware to minimize correlated failures.**
- The name for placement group must be **unique** within your AWS account
- You cant merge placement groups
- You can move existing instance into a placement group. Before you can move the instance, the instsance must be stopped. It *can only be achieved using AWS CLI or SDK*, not yet with a console
1. **Clustered Placement group**
    - Grouping instances within a **single AZ** (put them very close together)
    - Recommented for applications that need **low network latency, high network throughput, or both**
    - Only **certain instances** can be launched in this group
    - Can't span multiple AZ, all other groups can
    - AWS recommneds **homogenous instances** within clustered placement group
2. **Spread Placement group**
    - Each instance is placed on **distinct** underlying hardware
    - Recommended for applications that have a small number of **critical instances that should be kept separate from each other**
        - For example if one applications fails, it should not effect the others
    - Can have in **different AZ within a region**
    - Think like separate instances
3. **Partitioned**
    - AWS divides each group into **logical segments called partitions.**
    - Each partition within a placement group has its own **set of racks**. Each eack has its **own network and power source.**
    - No two partitions within a placement group share the same racks, allowing you to **isolate** the impact of hardware failure within your application
    - Think like multiple instances


## Auto Scaling
- Monitors your applications and automatically adjusts capacity to maintain steady, predictable performance at the lowest possible cost
- Available services for **auto-scaling** include:
    1. **EC2**
    2. **ECS**
    3. **DynamoDB**
    4. **Aurora**
- You create a collections of EC2 isntances (Auto Scaling groups) and AWS automatically provides **horizontal** scaling for your instances
- It is a **region-specific** service
- No cost for auto scaling, only for resources
- **Launch configuration** is the template used to create new EC2 instances and include parameters such as instance family, type, AMI, key pair and security groups
- **Types**
| Scaling        | Description                                                      | When to use                                                               |
|----------------|------------------------------------------------------------------|---------------------------------------------------------------------------|
| **Maintain**   | Ensures the required number of instances are running             | Use when you always need a known number of instances running at all times |
| **Manual**     | Manually change desired capacity                                 | Use when your needs change rarely enough that you're ok to make manual changes       |
| **Scheduled**  | Adjust min/max on specific dates/times or recurring time periods | Use when you know when your busy and quiet times are. Useful for ensuring enough instances are available befure very busy times |
| **Dynamic**    | Scale in response to system load or other triggers using metrics | useful for changing capacity based on system utilisation |
| **Predictive** | Predict capacity required ahead of time using ML                 | Useful for when capacity, and number of instances is unknown |

- Scaling policies for **Dynamic scaling**
| Scaling policy             | What it is                                                                                       | When to use |
|----------------------------|--------------------------------------------------------------------------------------------------|-------------|
| **Target Tracking policy** | Adds or removes capacity as required to keep the metric at or close to the specific target value | You want to keep the CPU usage of our ASG at 70% |
| **Simple Scaling Policy**  | Waits for the health check and cool down periods to expire before re-evaluating                  | Useful when load is erratic. AWS recommends step staling instead of simple in most cases |
| **Step Scaling Policy**    | Increases or decreases the configured capacity of the Auto Scaling group based on a set of scaling adjustments, know as a step adjustment | You want to vary adjustments based on the size of the alarm breach |

- **Scaling based on Amazon SQS**
    - Uses a custom metric that's sent to Amazon CloudWatch that measures the number of messages in the queue per EC2 instance in the Auto Scaling group
    - **Target tracking policy** scales based on the custom metric and a set target value. CloudWatch alarms invoke the scaling policy
- Uses **serivce-linked roles** - is a unique type of IAM role that is linked directly to an AWS service

__ASG Behavior and Configuration__
- Termination policies control the instances which are terminated first when a scale-in event occurs
    - The default termination policy ensures thats EC2 instances span AZ evenly for high availability
    - You can enable **Instance Protection** which prevents Auto Scaling from scaling in and terminating the EC2 instances

## SPOT instances and fleets
- Use unused EC2 capacity in the AWS cloud. Are available at up to 90% discount comparaed to On-Demand. Can use for stateless, fault-tolerant or flexiable applications, such as big data, containeriz workloads, CI/CD, web servers, HPC and other test workloads. They are not good for persistent workloads, critical jobs and databases
- To use the Spot Instances, you must first decide on your maximum Spot price. The instance will be provisioned so long as the Spot price is below your maximum Spot price. Hourly spot price varies depending on capacity and region
- If spot price goes above your maximum, you have two minutes to choose wheter to stop or terminate your instance
- You can use *Spot block* to stop your Spot Instances from being terminated even if the Spot price goes over your maximum Spot price. You can set Spot blocks for between one to six hours currently. 
- **Procces of creating a spot instance**
    1. Create a request (Maximum price, Desired amount of instances, Launch specifications, Request type (one time or persistent), Timeline (Valid from and valid until))
    2. Launch the instance. If the price increases, one time instances will be removed, while persistent will be removed and launched again once th eprice meets the criteria 

__Spot fleets__
- Is a collection of Spot Instances and optionally, On-Demand instances.
- The spot fleet attempts to maintain its target capacity fleet if your spot instances are interupted.
- Tasks done:
    1. Set up different launch pools. Define things like EC2 instance type, OS and AZ
    2. You can have multiple pools, and the fleet will choose the best way to implement depending on the strategy youd define. Strategies:
        - **capacityOptimized** - The spot instances come from the pool with optimal capacity for the number of instances launching
        - **diversified** - The spot instaces are distributed across all pools
        - **lowerPrice** - the spot instances come from the pool with the lowest price. This is the default strategy
        - **InstancePoolsToUseCount** - the spot instaces are distributed across the number of spot instance pool ypu specify. This parameter is valid only when used in combination with lowestPrice
    3. Spot fleets will stop launching instances once you reach your price threshold or capacity desire

## EC2 Hipernate
- When you hibernate an EC2 instance, the OS is told to perform hibernation. It saves the contents from the instance memory RAM to your EBS root volume.
- OS does not need to reboot, since the RAM contents are reloaded, rovides much faster boot up
- It requires to have root volume encryption
- Instance RAM must be less than 150 GB
- Can't be hibernated for more than 60 days

## Cloudtrail
- increases visibility into your user and resource activity by recording AWS Management COnsole actions and API calls
- Can create a **trail** to enable continuous delivery of CloudTrail events to an Amazon S3 bucket
- Trails enable you to store records indefinitely
- if you don't configure a trail, you can still view the most recent events in the CloudTrail **(past 90 days only)**
- You can determine the request made, source IP address, owner of request, date and additional details

## Cloudwatch
- Can monitor most of AWS as well as your applications that run on AWS
- CloudWatch with EC2 will monitor events every 5 minutes by default
    - You can have 1 miunte intervals by turning on detailed monitoring (chargeable)

## IAM Roles
- Roles are more secure than storing your access and secret keys on individual EC2 instances.
- Roles are easier to manage
- Roles can be assigned to EC2 instances after it is created using both console and command line
- Roles are universal - can be used in any region

## High Availability approaches
- Up to date AMIs are critical for rapid fail-over
- AMIs can be copied to other regions for safety or DR
- Horizontally scalable architectures are preferred because risk can be spread across multiple smaller machines versus one large machine
- Auto scaling and ELB work together to provide automated recovery by maintaining minimum instances
- Route53 health checks


## AWS Server Migration Service
- Is an agnet-less service which makes it easier and faster for you to migrate thousands of on-premises workloads to AWS
- Allows to automate, schedule and track incremental replications of live server volumes, making it easier for you to coordinate large-scale server migrations
- Replicates VMs to AWS, syncing volumes and creating periodic AMIs


## High performance computing (HPC)
- Create a large number of resources in almost no time. You only pay for the resources you use - and, once finished, you can destroy them
- Can be achieved by these services
1. Data transfer (get data into AWS)
    - *Snowball, SNowmobile* (terabytes/petabytes worth of data)
    - *AWS DataSync* to store on S3, EFS, FSx for Windows
    - *Direct Connect* (cloud service that established a dedicated network connection from your premises to AWS.)
2. Compute and networking
    - CPU or GPU optimized
    - EC2 fleets (Spot instances or fleets)
    - Placement groups (cluster placement groups)
    - Enhanced networking
        1. Uses single root I/O  (SR-IOV)virtualization to provide high-performance networking on supported instance tpypes. Provides higher I/O performance and lower CPU utilization.
        2. Use where you want good network connection
        3. Provies higher bandwidth, higher packet per second performance, consistently lower inter-instance latencies. There is additional charge for this service
        4. Can be enabled using:
            - *Elastic Network Adapter* - supports network speed of up to 100Gbps (Chose during exam)
            - *Virtual Function* interface, supports network speed up to 10 Gbps used for legacy systems
    - Elastic Network Adapters
    - Elastic Fabric Adapters
        1. Is a network device you can attach to your EC2 to accelerate HPC and ML applications
        2. Provides lower, more consistent latency and higher throughput than the TCP protocol.
        3. Can us OS-bypass, allows HPC and ML applications to bypass OS kernel and communicate directly with the EFA device. Only linux
3. Storage
    1. Instance-attached storage
        - EBS
        - Instance Store - scale to millions of IOPS, low latency
    2. Network storage
        - S3
        - EFS
        - FSx
4. Orchestration and automation
    - AWS Batch - batch computing jobs
    - ParallelCluster
