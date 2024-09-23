---
title: "Introduction to Identity Management for Web and Mobile Apps Using AWS Cognito User Sign Ups"
datePublished: Tue Jul 04 2023 23:40:57 GMT+0000 (Coordinated Universal Time)
cuid: cljoxo8rj000409kwftanbarh
slug: introduction-to-identity-management-for-web-and-mobile-apps-using-aws-cognito-user-sign-ups
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1688513947880/31d3c8c2-dc21-444d-911a-f9c4bda6563e.png
tags: aws, aws-cognito, user-management, aws-community-builder, identity-management

---

AWS Cognito is a user authentication and management service that provides a secure way to manage user sign-ups and sign-ins for your web or mobile applications. It simplifies the development process for creating user authentication systems and provides secure authentication and authorization mechanisms for your application.

In this blog post, we will explore the concept of user pools in AWS Cognito and give step-by-step instructions for creating a user pool on AWS Cognito.

# Prerequisites

Before getting started, there are a few things you will need:

* AWS account. Follow the instructions [here](https://portal.aws.amazon.com/billing/signup) to create one if you need one.
    

# Creating a User Pool on AWS Cognito

To get started with AWS Cognito, create a user pool in the AWS Management Console.

An AWS Cognito user pool is a fully managed user directory that allows you to easily sign up and authenticate users for your applications. It provides features such as user registration, login, password management, and multi-factor authentication. User pools integrate seamlessly with other AWS services and can be customized to suit your specific authentication and authorization requirements.

Follow these steps to create a user pool:

* Login into the AWS Management Console as an [IAM user](https://docs.aws.amazon.com/singlesignon/latest/userguide/getting-started.html) and navigate to the [Cognito service](https://eu-west-1.console.aws.amazon.com/cognito/v2/home?region=eu-west-1). (Ensure that your user has the appropriate permissions to use Cognito.)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680620254273/a5d78750-4a96-4708-8836-89cee799bbf8.png align="center")
    
* Click **Create User Pool**. Select **Federated identity providers** to enable your users to sign in using service providers like Google and Facebook.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680620724714/2013e0b2-8dcb-467a-b669-ef210b45f269.png align="center")
    
* Choose the attributes in your user pool that are used to sign in. For this app, we require just an email from the user.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680621072270/7b2354f7-34db-4da2-9bd0-f327c2b330ea.png align="center")
    
    Configure security requirements. Set a password policy, multi-factor authentication (MFA) requirements, and user account recovery options. I am using Cognito's default password policy and optional MFA using Authenticator apps and SMS. Leave other settings as default.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680642716483/2b05fdcd-ca1f-4aa7-8d3b-f0e622403066.png align="center")
    
* Retain the default settings in **Configure sign-up experience**. The section contains settings such as user identity verification during sign-up and the required fields in the signup form.
    
* In the next section **Configure message delivery,** select *Send email with Cognito*.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680643340788/9d95e5de-2b1f-431c-be83-632c4dc53dfb.png align="center")
    
* Follow the steps to **Connect federated identity providers** in the next section. You can also choose to temporarily skip this section.
    
* Give the user pool a name. Select Use Cognito domain in the Domain section and insert a prefix for the URL. Enter an App Client Name and select Public Client under Application Type. Select **Don't generate client secret** and add a callback URL which can be localhost for now.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680644923673/79cb436f-6449-40f3-9315-8215c7d54c03.png align="center")
    
* Review and create.
    
* Open up the user pool that has been created. Scroll down to Edit Hosted UI customization and upload an app logo and CSS template.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680646095362/1e24257b-1237-4a08-8e51-e5d43de24776.png align="center")
    
    We can download the Cognito CSS template for use.
    
* Back on the user pool home page, scroll down to the App client list and click on the app client we just created.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680646397275/5cfa35dd-92f3-4d5c-b244-a0a61bfb9704.png align="center")
    
* Click on View Hosted UI. To open up the app sign up page.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1680646587336/bd728cf6-4767-4fe0-abce-a6b3ad0c0966.png align="center")
    

# Testing User Sign Ups

* Provide test information for a user using a verifiable email address and a valid password to sign up from the app sign-up page.
    
* Every user that signs up on the app can be viewed on the user pool page for this app on AWS Cognito.
    

# Next Steps

I hope this blog post properly explained the concept of user pools on AWS Cognito, how to create a user pool and sign up new users to the app.

As the next steps, we will dive deeper into connecting this user pool to the application backend built using Django REST Framework to create user records in the app database as well.