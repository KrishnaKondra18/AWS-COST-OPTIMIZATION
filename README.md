# AWS-COST-OPTIMIZATION
#pre-requisites:
1. Good knowledge on functioning and useflow of AWS services and what are the factors taken into consideration by the amazon for charges. 
2. Knowledge on python programming language enough to write Lambda functions.
3. Creation of IAM roles and policies. 

## Project - Identifying Unused EBS Snapshots 
This example outlines a Lambda function designed to locate and remove EBS snapshots that are no longer connected to active EC2 instances. This approach aims to optimize storage costs by eliminating unnecessary snapshots.

Here's a breakdown of the process:

1.The Lambda function retrieves all EBS snapshots belonging to the current account.

2.It then fetches a list of active EC2 instances, encompassing both running and stopped states.

3.For each snapshot, the function verifies if its corresponding volume (if present) is not attached to any active instance identified in step 2.

4.If an unassociated or "stale" snapshot is found, the function proceeds to delete it, achieving storage cost optimization.
