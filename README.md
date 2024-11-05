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


AWS Lambda and EBS Snapshot Management:


Overview of AWS Lambda:

> AWS Lambda is a serverless compute service that runs code in response to events and automatically manages the underlying compute resources.

> The default execution time for Lambda functions is 3 seconds, but it can be increased to 15 minutes as needed.


Cost Considerations:

> Keeping Lambda execution time short is beneficial as AWS charges based on execution time.

> If resources like EBS snapshots or volumes are not deleted after use, they will incur costs.


Permissions and Roles:

> When a service like Lambda interacts with other AWS services, it does so through an IAM role.

> To manage EBS snapshots, permissions such as describe snapshots and delete snapshots must be granted to the IAM role.


Creating and Attaching Policies:

> Custom policies can be created for specific actions like describing and deleting snapshots.

> Policies should be attached to the role executing the Lambda function to ensure it has the necessary permissions.


Snapshot Management Logic:

> The Lambda function can be designed to delete stale snapshots—those not associated with any active volume.

> A condition can be added to check if a snapshot is older than 30 days before deletion.


Event-Driven Execution:

> Lambda functions can be triggered manually or scheduled using CloudWatch Events to run at specified intervals.

> Be cautious with scheduled executions to avoid unnecessary costs.



Conclusion:

Managing EBS snapshots through AWS Lambda is an effective way to optimize cloud costs and maintain resource efficiency. The implementation involves setting up proper permissions, writing the Lambda function logic, and ensuring it runs in an event-driven manner.



