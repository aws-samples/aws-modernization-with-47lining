---
title: "EDI Architecture"
chapter: true
weight: 3 # MODIFY THIS VALUE TO REFLECT THE ORDERING OF THE MODULES IF APPLICABLE
---

# EDI Architecture 

## Architecture Diagram
This architecture diagram shows how the various components of the EDI solution are deployed in your AWS account. <br>

![EDI Architecture](/images/edi_architecture.jpg)

Users interact through Load Balancer and Istio Gateway. This solution is deployed in an Amazon VPC, interacting with multiple AWS core services, such as AWS Certificate Manager (ACM), Amazon Route 53, and AWS Identity and Access Management (IAM) for security, management, and orchestration. OSDU Data Platform services run inside pods inside the Amazon Elastic Kubernetes Service (Amazon EKS) cluster. Services use several in-cluster components, such as Elasticsearch and MongoDB. Other AWS managed services, such as Amazon DynamoDB and Amazon Simple Storage Service (Amazon S3) are used as storage for metadata and raw data.

### Workshop Prerequisites
Next, let's go over the workshop prereqisities and setup your EDI account