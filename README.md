# AWS EBS, Snapshots, AMI & Launch Templates Scenario-Based Questions & Answers

## 1. **Data Loss Prevention & Recovery**
**Question:** A critical database running on EBS-backed EC2 instance was accidentally deleted by a developer. How would you recover the data with minimal downtime?

**Answer:**
1. **Immediate assessment:**
   - Determine when the deletion occurred
   - Check if the instance is EBS-backed (not instance store)
   - Verify latest available EBS snapshots

2. **Recovery options:**
   - **Option A (Using existing snapshot):**
     - Create new EBS volume from most recent snapshot
     - Stop the affected EC2 instance
     - Detach the corrupted volume, attach new volume
     - Modify file system mount if necessary
     - Start instance
   
   - **Option B (Create new instance):**
     - Create AMI from snapshot (if data volume is root volume)
     - Launch new EC2 instance from this AMI
     - Update DNS/ELB to point to new instance
   
   - **Option C (Volume swap):**
     - Create new volume from snapshot
     - Attach as secondary volume to running instance
     - Copy data to existing volume (if only partial data loss)

3. **Best practice implementation:**
   - **RPO/RTO alignment:** Implement more frequent snapshots if RPO is too large
   - **Automation:** Use Data Lifecycle Manager for automated snapshots
   - **Testing:** Regularly test recovery procedures
   - **Protection:** Enable delete protection on critical volumes

## 2. **EBS Performance Issues**
**Question:** Your database application on EC2 is experiencing slow I/O performance. The EBS volume is gp2 (general purpose SSD). How would you diagnose and resolve?

**Answer:**
1. **Diagnosis steps:**
   - Check CloudWatch metrics: `VolumeReadOps`, `VolumeWriteOps`, `VolumeQueueLength`, `BurstBalance`
   - For gp2: Monitor if burst credits exhausted (BurstBalance near 0)
   - Check if application I/O pattern changed (higher IOPS needs)
   - Use `iostat -x` on Linux to see await time and %util

2. **Resolution options:**
   - **Increase gp2 size:** gp2 provides baseline IOPS of 3 IOPS/GB (max 16,000)
   - **Migrate to gp3:** Newer generation, separate IOPS/throughput provisioning
   - **Switch to io1/io2:** For consistent high performance (>16,000 IOPS)
   - **Use provisioned IOPS:** If predictable high performance needed
   - **Optimize application:**
     - Implement database connection pooling
     - Add caching layer (ElastiCache)
     - Optimize queries and indexes

3. **Example calculation:**
   ```
   Current: 500GB gp2 = 1,500 baseline IOPS
   Need: 5,000 IOPS consistently
   Options:
   - Increase to 1,667GB gp2 (expensive)
   - Migrate to 500GB gp3 with 5,000 provisioned IOPS (cost-effective)
   - Use 334GB io2 with 5,000 provisioned IOPS
   ```

## 3. **Cost Optimization for EBS Snapshots**
**Question:** Your EBS snapshot costs have increased significantly. How would you optimize?

**Answer:**
1. **Snapshot analysis:**
   - Use AWS Cost Explorer to identify snapshot costs
   - Check snapshot age and retention policy
   - Identify unused/obsolete snapshots
   - Verify cross-region replication costs

2. **Optimization strategies:**
   - **Implement lifecycle policies:**
     ```bash
     # AWS CLI example
     aws dlm create-lifecycle-policy --policy-details file://policy.json
     ```
     ```json
     {
       "PolicyType": "EBS_SNAPSHOT_MANAGEMENT",
       "ResourceTypes": ["VOLUME"],
       "Schedules": [
         {
           "Name": "DailyBackups",
           "RetainRule": {"Count": 7}
         },
         {
           "Name": "WeeklyBackups",
           "RetainRule": {"Count": 4},
           "CreateRule": {"Interval": 7, "IntervalUnit": "DAYS"}
         }
       ]
     }
     ```
   
   - **Use incremental snapshots:** Only changed blocks stored
   - **Delete old/unused snapshots:** With proper approval workflow
   - **Consider alternative backups:** For less critical data, use AMI lifecycle policies
   - **Compression:** Some third-party tools offer compression

3. **Architecture improvements:**
   - Implement data tiering (hot vs cold data)
   - Use smaller volumes with specific data only
   - Consider EBS Snapshots Archive tier for long-term retention

## 4. **AMI Management & Compliance**
**Question:** Your security team requires all EC2 instances to use the latest AMI with security patches. How would you implement this?

**Answer:**
1. **Golden AMI pipeline:**
   - **Build pipeline:**
     ```
     Source AMI → Packer/EC2 Image Builder → Security scanning → 
     Vulnerability assessment → Approved Golden AMI → Shared across accounts
     ```
   - Use AWS EC2 Image Builder for automated AMI creation
   - Integrate with vulnerability scanners (Inspector, third-party)

2. **Automation implementation:**
   - Use Systems Manager State Manager to enforce AMI compliance
   - Implement Launch Templates referencing latest approved AMI
   - Use Parameter Store to store approved AMI IDs
   
   ```json
   // Launch Template versioning
   {
     "LaunchTemplateName": "app-server-lt",
     "VersionDescription": "Updated with patched AMI-12345",
     "LaunchTemplateData": {
       "ImageId": "ami-1234567890abcdef0",
       "InstanceType": "t3.large"
     }
   }
   ```

3. **Update strategy:**
   - Blue-green deployment for AMI updates
   - Canary deployments for testing new AMIs
   - Automated AMI deprecation after N days

4. **Monitoring and compliance:**
   - Use Config rules to detect non-compliant instances
   - Set up EventBridge rules for AMI update notifications
   - Regular patching schedule (e.g., every 2 weeks)

## 5. **Launch Templates for Auto Scaling**
**Question:** Your Auto Scaling group needs to deploy instances with different configurations for dev/staging/prod. How would you manage this using Launch Templates?

**Answer:**
1. **Launch Template structure:**
   ```json
   // Base template (common settings)
   {
     "NetworkInterfaces": [{"DeviceIndex": 0, "SubnetId": "subnet-123"}],
     "ImageId": "ami-base123",
     "InstanceType": "t3.medium",
     "KeyName": "app-key",
     "TagSpecifications": [{"ResourceType": "instance", "Tags": [...]}]
   }
   
   // Environment-specific overrides
   Dev:    InstanceType = t3.small,  EBS = gp2 100GB
   Stage:  InstanceType = t3.medium, EBS = gp3 200GB
   Prod:   InstanceType = t3.large,  EBS = io2 500GB
   ```

2. **Implementation approach:**
   - Use versioned Launch Templates for each environment
   - Store configuration in Parameter Store/Secrets Manager
   - Use IAM instance profiles for environment-specific permissions
   
   ```bash
   # Create template versions
   aws ec2 create-launch-template-version \
     --launch-template-name app-template \
     --version-description "Production v2" \
     --source-version 1 \
     --launch-template-data '{"InstanceType":"m5.large"}'
   ```

3. **Auto Scaling integration:**
   - Each environment has its own Auto Scaling group
   - Use mixed instances policy for flexibility
   - Implement instance refresh for rolling updates

   ```json
   {
     "AutoScalingGroupName": "prod-asg",
     "LaunchTemplate": {
       "LaunchTemplateName": "app-template",
       "Version": "$Latest"  // or specific version
     },
     "MixedInstancesPolicy": {
       "InstancesDistribution": {
         "OnDemandPercentageAboveBaseCapacity": 70
       }
     }
   }
   ```

## 6. **Cross-Region Disaster Recovery**
**Question:** How would you implement a cross-region DR strategy using EBS snapshots and AMIs?

**Answer:**
1. **Architecture design:**
   - Primary region: us-east-1
   - DR region: us-west-2
   - RPO: 1 hour, RTO: 4 hours

2. **Implementation steps:**
   - **Snapshot replication:**
     ```bash
     # Copy snapshot to DR region
     aws ec2 copy-snapshot \
       --source-region us-east-1 \
       --source-snapshot-id snap-123 \
       --region us-west-2 \
       --description "DR copy"
     ```
   
   - **Automation using Lambda:**
     - EventBridge scheduled rule triggers Lambda
     - Lambda creates snapshots, copies to DR region
     - Lambda creates AMI from latest DR snapshot
   
   - **AMI sharing:**
     ```bash
     # Share AMI with DR region account
     aws ec2 modify-image-attribute \
       --image-id ami-123 \
       --launch-permission "Add=[{UserId=dr-account-id}]"
     ```

3. **Recovery procedure:**
   - In DR event: Launch instances from latest AMI
   - Update Route 53 DNS failover routing
   - Restore databases from replicated snapshots
   - Test application functionality

4. **Cost optimization:**
   - Only replicate critical volumes
   - Use snapshot lifecycle policies in DR region
   - Consider cold storage for older DR snapshots

## 7. **Encryption & Security Compliance**
**Question:** A compliance audit requires all EBS volumes and snapshots to be encrypted. How would you implement and verify this?

**Answer:**
1. **Implementation strategy:**
   - **Default encryption:** Enable EBS encryption by default at account level
   ```bash
   aws ec2 enable-ebs-encryption-by-default
   ```
   
   - **Launch Template configuration:**
   ```json
   {
     "BlockDeviceMappings": [{
       "DeviceName": "/dev/sda1",
       "Ebs": {
         "Encrypted": true,
         "KmsKeyId": "alias/aws/ebs"
         // or custom KMS key
       }
     }]
   }
   ```
   
   - **Snapshot encryption:**
     - All snapshots of encrypted volumes are automatically encrypted
     - Copy unencrypted snapshots to encrypted:
     ```bash
     aws ec2 copy-snapshot --encrypted --kms-key-id alias/aws/ebs
     ```

2. **Verification and compliance:**
   - **AWS Config rules:** `ebs-snapshot-public-restorable-check`, `encrypted-volumes`
   - **Custom compliance checks:**
     ```python
     import boto3
     ec2 = boto3.client('ec2')
     
     # Check volume encryption
     volumes = ec2.describe_volumes()
     for vol in volumes['Volumes']:
         if not vol['Encrypted']:
             print(f"Unencrypted volume: {vol['VolumeId']}")
     ```

3. **Key management:**
   - Use AWS KMS for key management
   - Implement key rotation policies
   - Set up cross-account key sharing for multi-account strategies

## 8. **Storage Optimization Scenario**
**Question:** Your application has varying storage needs - some data is accessed frequently, some rarely. How would you optimize EBS costs?

**Answer:**
1. **EBS volume types strategy:**
   ```
   Hot data (frequent access):        io2/io1 (high IOPS needed)
   Warm data (regular access):        gp3 (balanced cost-performance)
   Cool data (infrequent access):     st1 (throughput optimized HDD)
   Cold data (rare access, backup):   sc1 (cold HDD)
   ```

2. **Implementation approach:**
   - **Multi-volume architecture:**
     - Root volume: gp3 (OS and applications)
     - Database volume: io2 (high performance)
     - Logs volume: st1 (sequential writes)
     - Archive volume: sc1 (old data)
   
   - **Lifecycle management:**
     - Use Data Lifecycle Manager to transition snapshots to archive
     - Implement Lambda to move data between volume types based on access patterns
   
   - **Monitoring and optimization:**
     ```bash
     # Monitor volume performance metrics
     aws cloudwatch get-metric-statistics \
       --namespace AWS/EBS \
       --metric-name VolumeReadBytes \
       --dimensions Name=VolumeId,Value=vol-123
     ```

3. **Cost comparison example:**
   ```
   1TB storage for 1 month:
   - io2 (16,000 IOPS): ~$1,000/month
   - gp3 (3,000 IOPS):  ~$100/month
   - st1:              ~$45/month
   - sc1:              ~$25/month
   ```

## Best Practices Summary:

### EBS:
- Always use gp3 over gp2 for new deployments
- Enable encryption by default
- Monitor performance metrics and burst balances
- Implement proper backup strategies

### Snapshots:
- Automate with Data Lifecycle Manager
- Test restore procedures regularly
- Consider archive tier for long-term retention
- Tag snapshots for cost allocation

### AMIs:
- Implement Golden AMI pipeline
- Version control for AMI management
- Regular patching schedule
- Cross-account sharing for enterprise

### Launch Templates:
- Use versioning for change management
- Reference AMIs via SSM Parameter Store
- Implement mixed instances policies
- Use for both Auto Scaling and single instances

### Interview Tips:
- Always mention trade-offs (cost vs performance vs availability)
- Reference specific AWS services by name
- Include CLI commands or infrastructure code examples
- Discuss monitoring and automation aspects
- Consider security and compliance requirements
