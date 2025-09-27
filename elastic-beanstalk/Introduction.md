## Application

An Elastic Beanstalk application is a container for Elastic Beanstalk components, including *environments, versions,* and *environment configurations*.

### Create Environment
- An application that serves HTTP requests runs in a *web server environment* tier. A backend environment that pulls tasks from an Amazon Simple Queue Service (Amazon SQS) queue runs in a *worker environment* tier.
- Application name and upto 50 tags.
- Environment name(develop, test, prod), subdomain(can be auto-generated) and descirption. Cannot be changed later.
- Platform such as Java, Python etc with branch like Amazon Corretto and version like 4.6.5.
- Upload or use sample application code.
- EC2 confirguration like single instance, HA or custom

### Configure Service Access
- IAM role attached to IAM managed policies assumed as service role // Allows access to other AWS service resources that are required to create and manage environments.
    - Trust Policy: sts:AssumeRole
    - Permission Policy:                          AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy, AWSElasticBeanstalkEnhancedHealth
    - Tags
- EC2 Instance profile attached to IAM managed policies // create Role
    - Trusted Entity: AWS Service, Use Case: EC2, PermissionPolicy: Beanstalk Web tier, worker tier, multi-container docker
    - Trust Policy: sts:AssumeRole
    - Tags

### Setup networking, database and tags - optional
- VPC, Public IP etc
- If database is enable, it is bound to lifecycle of environment. other way is to snapshot database and restore at later stage.

### Configure instance traffic and scaling - optional
- option to select root volume, its size, iops, troughput, instance metadata service (currently IMDSv1 is deactivated, but you can toggle it)
- cloudwatch monitoring interval
- security groups to control traffic(leave as is)
- autoscaling group with env type(LB, single) and (min, max) instances, fleet composition, architecture to use, instance type, ami-id, scaling cooldown, Scaling triggers 
- Load Balancer Network settings: visibility(public) and subnets(select), type(App(select), network), (dedicated(select), shared), listeners(route client traffic to proccess, port, protocol), proccess(port, protocol, health check, stickiness), rules(priority based, default rule) for LB, S3 storage to store LB logs

### Configure updates, monitoring, and logging - optional
- health reporting(basic, enhanced(real-time)) with custom metrics for instance and environment
- updates can be managed(toggle), scheduled matinenace window, (minor(optional) + patch), instance replacement(toggle)
- email notifications.
- deployments: policy(rolling(time, health based) + (optional additional batch), immutable), batch size(%, fixed), min capacity, preferences
- Proxy server(Apache, Nginx), X-Ray Daemon, log rotation(s3), log streaming to cloudwatch
- Environment properties like plaintext or from SSM or ASM

### Creation Process Completed 
- Access app at domain link
- check cloudformation

## Tabs:

### Events Case 1 : for Single Instance
- Amazon S3 storage bucket for environment data
- Created security group
- Created EIP
- Created Autoscaling Group // doubtfull
- Added instance to environment
- Deployment completed.
- Health check

### Events Case 2: for HA
- Amazon S3 storage bucket for environment data
- Created security group
- Created Autoscaling Group and Policy
- Created CloudWatch alarm
- Created load balancer and listener
- Deployment completed.



### Health
- overall and instance health listing throughput, error and success code, latency, cpu usage

### Logs
- log files

### Monitoring
- Service metrics like environment health, cpu utilization, network in/out

### Alarms
- Alarms on metrics and notifications

### Managed Updates
- AWS regularly releases platform updates, you can configure your environment to automatically upgrade to the latest version of a platform during a scheduled maintenance window. Application remains in service during the update process with no reduction in capacity.

### Tags
- Name : env name
- elasticbeanstalk:environment-id : some-id
- elasticbeanstalk:environment-name : env-name

### Where is what
- EC2 -> Autoscaling Group -> Instance management -> instance health
- EC2 -> EIP (same as Public IP)
- EC2 -> EC2 instance profile (aws-elasticbeankstalk-ec2-role) 

## Upload new version
- Upload and deploy button

### Additional steps and notes for EC2 Confirguration as HA
- Select VPC ID
- choose subnets for availabilit zones such as us-east-2a, us-east-2b, us-east-2c
- Note: Target group security group allowing traffic from LB security group, port 80, TCP
- Note: LB SG allows port 80 inbound and outbound from anywhere

### Cloudformation events
- Load balancer target group
- Security Groups
- Load balancer and listener
- Autoscaling group, up and down policy
- Cloudwatch Alarm

### Deployment modes
- All at once
    - Pros: fastest, no cost, quick iteration
    - Cons: downtime
- Rolling
    - Pros: no downtime, no cost
    - Neutral: slower than AllAtOnce but still fast
    - Cons: reduced service capacity by batch size
- Rolling with additional batch
    - Pros: no downtime, no reduced capacity
    - Neutral: slower than rolling but still fast
    - Cons: additional cost of batch
- Immutable: temp ASG, moves instances from temp to old ASG, remove temp ASG
    - Pros: no downtime, quick rollback
    - Neutral: Great for prod
    - Cons: high cost as double capacity, longest deployment
- Blue/Green Deployment: new stage(green) env, route53, weighted policy, swap URLs
    - Pros: no downtime
    - Neutral:
    - Cons: manual, complicated
- Traffic Splitting: canary testing, temp ASG with same capacity, small traffic sent here for configurable time using ALB, new instances migrated to old ASG
    - Pros: automated rollback, no downtime, longest till now
    - Neutral:
    - Cons: temporary failure impact

### Deployment practice
- clone environment and deploy new version
- test the environment
- swap environment domain

### Lifecycle Policy
- max 1000 versions
     - time based
     - space based
- option not to delete src bundle

### Code configurations
- placed under.ebextension/ directory located in root folder
- YAML/JSON format and end with .config
- able to modify default settings: option_settings 
- able to add AWS resources

### Option Settings Use case
- inside config file, define database connectivity

### Migration Use case
- change the load balancer type with same config (can't use clone)
- deploy app
- perform CNAME swap or route 53 update

### Database Migration
- create database snapshot, Go to console and enable deletion protection
- create new env without database, point it to existing database
- perform CNAME swap/ Route53 update
- delete old env


