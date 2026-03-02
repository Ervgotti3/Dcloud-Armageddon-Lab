# Armageddon AWS Lab 3

## Technologies Used

- Amazon Web Services
- Terraform
- Python
- MySQL

## Lab Overview

<-- TODO Add Diagram-->

## Global Infrastructure for Cloudfront
![alt text](<Global Users.png>)
CloudFront serves as the single global entry point and is protected by AWS WAF. Traffic is routed to either the Tokyo or São Paulo Application Load Balancer based on cache behavior and origin configuration. No application servers or databases exist in the global layer.

## Tokyo Infrastructure
![alt text](Tokyo.png)
The Tokyo region architecture. The VPC contains public and private subnets, with the ALB in the public subnet and application servers and RDS deployed in private subnets. All patient medical data (PHI) is stored exclusively in Tokyo RDS. Connectivity to São Paulo occurs through a Transit Gateway attachment.

## Sao Paulo Infrastructure
![alt text](Sao_Paulo.png)

This lab builds on Labs 1 and 2 with a very important caveat...For this lab data can only exist in the Tokyo region. The Sao Paulo region can access data but is NOT allowed to persist any data. In order to satisfy this requirement, the RDS instance will only exist in the Tokyo region. In order for the instance in Sao Paulo to access data, we will configure a Transit Gateway (TGW). The TGW will allow the instance in the Sau Paulo region to connect to the RDS instance even though it lives in a private subnet in another region.

## Transit Gateway

To further increase security, we'll be making sure that access to the web application can only occur through CloudFront. We'll also use best practices like least privilege to ensure that instances do not have more access than they need.

### Sample Traffic FloW

<-- TODO Add Diagram-->

## Why This Lab is Important for Cloud Engineers

Companies are required to comply with various laws such as GDPR, HIPPA...As a cloud engineer, you will be required to create architecture that is compliant with laws. This lab shows that you understand data residency and can build an architecture that complies with laws that restrict where data can live and how data can be accessed.
