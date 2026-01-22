# AWS EC2 Scenario-Based Interview Questions & Answers

## 1. **High CPU Utilization Scenario**
**Question:** You receive an alert that an EC2 instance is consistently at 90% CPU utilization during business hours. How would you investigate and resolve this?

**Answer:**
1. **Immediate investigation:**
   - Check CloudWatch metrics for CPU utilization patterns
   - Use CloudWatch Logs Insights to analyze application logs
   - SSH into instance and run `top`, `htop`, or `vmstat` to identify processes
   - Check application-specific metrics (APM tools, application logs)

2. **Short-term fixes:**
   - Scale vertically: Change instance type to larger CPU capacity
   - Scale horizontally: Add more instances behind ELB/ALB
   - Implement Auto Scaling based on CPU metrics

3. **Long-term solutions:**
   - Optimize application code
   - Implement caching (ElastiCache)
   - Use asynchronous processing (SQS + Lambda)
   - Consider moving to containers (ECS/EKS) for better resource utilization

4. **Prevention:**
   - Set up CloudWatch alarms at 70-80% threshold
   - Implement proper Auto Scaling policies
   - Regular performance testing

## 2. **Instance Connectivity Issue**
**Question:** Developers cannot SSH into an EC2 instance. What steps would you take to troubleshoot?

**Answer:**
1. **Check instance state:**
   - Verify instance is in "running" state in AWS Console
   - Check System Status and Instance Status checks

2. **Security Group verification:**
   - Ensure port 22 is open to relevant IP ranges
   - Check Network ACLs for any deny rules

3. **Key pair and authentication:**
   - Verify correct key pair is being used
   - Check if instance has IAM role with proper permissions

4. **Network troubleshooting:**
   - Use VPC Reachability Analyzer
   - Check route tables for proper routing
   - Verify Internet Gateway attachment for public instances
   - For private instances: check NAT Gateway/VPC endpoints

5. **Instance-level checks:**
   - Use EC2 Instance Connect or SSM Session Manager as alternatives
   - Check instance logs via AWS Console (Get System Log)
   - Verify if OS-level firewall (iptables, firewalld) is blocking SSH

## 3. **Cost Optimization Scenario**
**Question:** Your EC2 costs have increased by 40% this month. How would you identify the cause and reduce costs?

**Answer:**
1. **Cost analysis:**
   - Use AWS Cost Explorer to identify service/resource breakdown
   - Check for underutilized instances (low CPU/memory usage)
   - Identify non-production instances running 24/7

2. **Right-sizing:**
   - Analyze CloudWatch metrics for CPU, memory, network
   - Use AWS Compute Optimizer recommendations
   - Downsize over-provisioned instances

3. **Purchasing optimization:**
   - Convert on-demand to Reserved Instances/Savings Plans
   - Use Spot Instances for fault-tolerant, stateless workloads
   - Implement Auto Scaling to match demand

4. **Architecture review:**
   - Consider serverless alternatives (Lambda) for burst workloads
   - Implement auto-start/stop schedules for dev/test instances
   - Use smaller instance types with containers

5. **Tagging and governance:**
   - Ensure all resources are tagged (environment, owner, project)
   - Implement AWS Budgets with alerts
   - Use Service Control Policies to enforce tagging

## 4. **Disaster Recovery Scenario**
**Question:** An Availability Zone goes down. How do you ensure your EC2-based application remains available?

**Answer:**
1. **Multi-AZ architecture:**
   - Deploy instances across multiple AZs
   - Use Auto Scaling groups spanning multiple AZs
   - Distribute traffic using ELB/ALB across AZs

2. **Data replication:**
   - Use EBS snapshots for volume backups
   - Implement RDS Multi-AZ for databases
   - Store shared data in S3 or EFS

3. **Recovery procedures:**
   - Have AMIs with latest application versions
   - Maintain CloudFormation/Terraform templates for infrastructure
   - Test failover procedures regularly

4. **Monitoring and automation:**
   - Set up CloudWatch alarms for instance/zone failures
   - Use Route 53 health checks for DNS failover
   - Automate recovery with AWS Systems Manager Automation

## 5. **Application Deployment Strategy**
**Question:** How would you deploy a new version of your application on EC2 with zero downtime?

**Answer:**
1. **Blue-Green deployment:**
   - Launch new instances with updated application (Green)
   - Route traffic gradually using Route 53 weighted routing or ALB
   - Monitor new deployment before decommissioning old

2. **Canary deployment:**
   - Deploy to small percentage of instances first
   - Gradually increase traffic to new version
   - Roll back if metrics show issues

3. **Using AWS services:**
   - **CodeDeploy:** For in-place or blue-green deployments
   - **Elastic Beanstalk:** For managed deployment with rollback
   - **ALB:** For weighted target group routing

4. **Infrastructure as Code:**
   - Update Launch Templates/Configurations
   - Create new Auto Scaling group with updated AMI
   - Shift traffic using ALB listener rules

5. **Validation:**
   - Implement health checks
   - Monitor application metrics during deployment
   - Automated rollback procedures on failure

## 6. **Security Breach Scenario**
**Question:** You suspect an EC2 instance has been compromised. What immediate actions would you take?

**Answer:**
1. **Containment:**
   - Isolate instance by modifying Security Groups (allow only your IP)
   - Take EBS snapshots for forensic analysis
   - Do NOT terminate instance immediately (preserve evidence)

2. **Investigation:**
   - Check CloudTrail logs for unauthorized API calls
   - Analyze VPC Flow Logs for suspicious traffic
   - Use GuardDuty findings
   - Check instance metadata service access logs

3. **Remediation:**
   - Rotate all credentials (IAM roles, application keys, SSH keys)
   - Launch new instance from clean AMI
   - Restore data from last known good backup
   - Apply all security patches

4. **Prevention:**
   - Implement IMDSv2 with hop limit
   - Use Security Hub for compliance monitoring
   - Regular vulnerability scanning
   - Minimal IAM roles with least privilege

## 7. **Performance Degradation Scenario**
**Question:** Users report slow response times from your application running on EC2. How do you diagnose?

**Answer:**
1. **End-to-end tracing:**
   - Check ELB/ALB metrics (latency, request count)
   - Use X-Ray for distributed tracing
   - Monitor application performance with CloudWatch custom metrics

2. **Resource analysis:**
   - Check EC2 metrics: CPU, memory, disk I/O, network
   - Monitor EBS volume performance (IOPS, throughput)
   - Check ENI network performance

3. **Dependency checks:**
   - Test database performance (RDS/ElastiCache metrics)
   - Check external API response times
   - Verify S3/DynamoDB latency

4. **Instance-level investigation:**
   - Use CloudWatch Agent for enhanced monitoring
   - Check swap usage, disk space
   - Analyze garbage collection logs (for Java apps)
   - Review application logs for errors or slow queries

## 8. **Auto Scaling Issues**
**Question:** Your Auto Scaling group isn't scaling out during high traffic, causing performance issues. How do you troubleshoot?

**Answer:**
1. **Verify Auto Scaling configuration:**
   - Check scaling policies and metrics/thresholds
   - Verify CloudWatch alarms are in "ALARM" state
   - Check cooldown periods and warm-up times

2. **Capacity issues:**
   - Verify instance limits in the region
   - Check if desired capacity exceeds max size
   - Verify Launch Template/Configuration

3. **Instance launch problems:**
   - Check Auto Scaling group activity history
   - Verify AMI availability
   - Check subnet/IP availability
   - Review Security Group and IAM role assignments

4. **Testing:**
   - Manually execute scaling policy
   - Test instance launch manually
   - Use predictive scaling for better planning

