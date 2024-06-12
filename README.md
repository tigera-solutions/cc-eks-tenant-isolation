# AWS: EKS Security Bootcamp </br> Implementing zero-trust security for containers

## Welcome

In this EKS-focused workshop, you will work with AWS and Calico Cloud to learn how implement zero-trust security for workloads to reduce the attack surface of applications running on EKS clusters.  

Cloud-native applications require a modern approach based on the zero-trust principles of identity-based access, least privilege access, and proactively detecting threats and reducing the blast radius in case of a breach.

Calico Cloud enables fine-grained, zero-trust workload access controls between your microservices and external databases, cloud services, APIs, and other applications. It also prevents the lateral movement of threats with identity-aware segmentation that works across all of your workload environments, including hosts, VMs, Kubernetes components, and services.

You will come away from this workshop with an understanding of how others in your industry are securing and observing cloud-native applications in AWS, along with best practices that you can implement in your organization.

### Time Requirements

The estimated time to complete this workshop is 60-90 minutes.

### Target Audience

- Cloud Professionals
- DevSecOps Professional
- Site Reliability Engineers (SRE)
- Solutions Architects
- Anyone interested in Calico Cloud :)

### Learning Objectives

1. Learn how to deploy zero-trust workload access controls with namespace isolation recommendations
2. Extend firewall protection at the granular, workload level
3. Block lateral movement of APTs with identity-aware microsegmentation
4. Understand how to apply zero-trust security controls at application level.

## Workshop Environment Preparation

> [!WARNING]
> **For this workshop, you are expected to have access to a previously created EKS cluster.**

- Please, follow the instructions on the repository below if you don't have it ready:

  [Calico Cloud on EKS - Workshop Environment Preparation](https://github.com/tigera-solutions/eks-workshop-prep.git)

- We will run this workshop from the AWS CloudShell, as described in that repository.

- To start your cluster, we will scale the nodegroup up to 2 nodes using ```eksctl```. Reload the environment variables that were created in your AWS CloudShell first and then scale the nodegroup up.
  
- Ensure the nodegroup variable is populated into the ```workshopvars.env``` file:

   ```bash
   export NGNAME=$(eksctl get nodegroups --cluster $CLUSTERNAME --region $REGION | grep $CLUSTERNAME | awk -F ' ' '{print $2}') && \
   echo export NGNAME=$NGNAME >> ~/workshopvars.env
   ```

- Use the following command:

  ```bash
  source ~/workshopvars.env
  eksctl scale nodegroup $NGNAME \
  --cluster $CLUSTERNAME \
  --region $REGION \
  --nodes 2 \
  --nodes-max 2 \
  --nodes-min 2
  ```

## Modules

This workshop is organized in sequential modules. One module will build up on top of the previous module, so please, follow the order as proposed below.

Module 1 - [Connect the EKS cluster to Calico Cloud](modules/module-1-connect-calicocloud.md)  
Module 2 - [Zero-Trust Workload Access Control with Namespace Isolation Recommendation](modules/module-2-ztac-ns-isolation.md)  
Module 3 - [Workload Isolation with Microsegmentation](modules/module-3-wkload-isolation.md)  
Module 4 - [Ingress and Egress access control using NetworkSets](modules/module-4-network-sets.md)  
Module 5 - [Application Level Observability](modules/module-5-application-observability.md)  
Module 6 - [Clean up](modules/module-6-clean-up.md)  

## Useful links

- [Project Calico](https://www.tigera.io/project-calico/)
- [Calico Academy - Get Calico Certified!](https://academy.tigera.io/)
- [O’REILLY EBOOK: Kubernetes security and observability](https://www.tigera.io/lp/kubernetes-security-and-observability-ebook)
- [Calico Users - Slack](https://slack.projectcalico.org/)

**Follow us on social media:**

- [LinkedIn](https://www.linkedin.com/company/tigera/)
- [Twitter](https://twitter.com/tigeraio)
- [YouTube](https://www.youtube.com/channel/UC8uN3yhpeBeerGNwDiQbcgw/)
- [Slack](https://calicousers.slack.com/)
- [Github](https://github.com/tigera-solutions/)
- [Discuss](https://discuss.projectcalico.tigera.io/)

> [!NOTE]
> The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how Calico Cloud can be configured to build a functional solution. These examples are not intended for use in production environments.
