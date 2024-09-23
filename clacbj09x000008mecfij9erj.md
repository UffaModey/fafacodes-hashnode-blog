---
title: "Password Generator REST API with AWS Lambda and AWS API Gateway"
datePublished: Fri Nov 11 2022 09:50:26 GMT+0000 (Coordinated Universal Time)
cuid: clacbj09x000008mecfij9erj
slug: password-generator-rest-api-with-aws-lambda-and-aws-api-gateway
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1668123900041/_U7Yc31vI.jpg
tags: aws, python, aws-lambda, aws-apigateway, password-generator

---


# Introduction 👋 
Password Generator REST API leverages [AWS Lambda proxy integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html) and [AWS API Gateway](https://aws.amazon.com/api-gateway/) to generate a strong password (not less than 8 characters) on each API request. The API user may specify the length of the desired password. A Python function on AWS Lambda generates the password using a combination of upper case, lower case, numbers and special characters. 

## Project Demo 🎥
%[https://youtu.be/72WqPfd8Zu8]

# Objectives 😇
- Create a password-generating Lambda function for the API backend.
- Create and test a Password Generator API with Lambda proxy integration.

# Prerequisites 📌
- AWS IAM user account with management console access.
- Postman

# The Development 💻 💻
## Create Password Generator AWS Lambda Function
The Lambda function returns the JSON response for a request with a valid password length:

```
{
    "Your password is: " + password + " Thanks for using the Password Generator"
}
``` 
The Lambda function returns the JSON response for a request with an invalid password length:

```
{
    "Your password must have at least 8 characters"
}
``` 

- Open Lambda service on the [AWS Management Console](https://us-east-1.console.aws.amazon.com/lambda/home?region=us-east-1#/functions). Ensure that you are signed into your IAM user account. I am using US East (N. Virginia) us-east-1 region. Click on **Functions** on the left side panel and click on **Create Function**.
- Select **Author from scratch**. In **Basic information**, set the **Function name** as *PasswordGeneratorLambdaProxyIntegration*. Select *Python 3.9* as the function's language. 
![Lambda basic information](https://cdn.hashnode.com/res/hashnode/image/upload/v1668116585477/gSm56ng89.png align="left")
- In **Permissions**, open **Change default execution role** dropdown. Select **Create a new role from AWS policy templates**. Set the role name as **PasswordGeneratorLambdaBasicExecutionRole**.
- Click **Create Function**.

![Create Lamda Function](https://cdn.hashnode.com/res/hashnode/image/upload/v1668116902510/3tZg5QcZT.png align="left")
- Copy/paste the python code below into code editor
![update lambda code.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668117059616/D19oDBkIq.png align="left")

```
import json
import random
import string

def lambda_handler(event, context):
    
    print("Welcome to Password Generator")
    
    # Set the password length with constraints for the password length
    password_length = int(event['queryStringParameters']['length'])
    if password_length < 8:
        return {
        'statusCode': 200,
        'body': json.dumps('Your password must have at least 8 characters.')
    }
    else:
        # define the characters
        lower = string.ascii_lowercase
        upper = string.ascii_uppercase
        num = string.digits
        symbols = string.punctuation
        
        all_characters = lower + upper + num + symbols
        
        # generate a password string
        temp = random.sample(all_characters, password_length)
        password = "".join(temp)    

        return {
        'statusCode': 200,
        'body': json.dumps("Your password is: " + password + " Thanks for using the Password Generator")
    }
    
``` 
- Click **Deploy**

## Create Password Generator API
- Open the [AWS API Gateway](https://us-east-1.console.aws.amazon.com/apigateway/main/apis?region=us-east-1) console.
- Select **Create API**.  Select a type of API. For this project, we are using a public REST API. Select **Build**.
![choose API type](https://cdn.hashnode.com/res/hashnode/image/upload/v1668118900970/-Xxt02t31.png align="left")
- Set the API name as **PasswordGeneratorLambdaSimpleProxy** and then click **Create API**.
![create API](https://cdn.hashnode.com/res/hashnode/image/upload/v1668119138002/vIZfcoK8r.png align="left")
- In **Resources** on the left-hand panel, **Create Resource** for the root / of the API under **Actions**. Set the **Resource Name** as **generatepassword**.
![create resource](https://cdn.hashnode.com/res/hashnode/image/upload/v1668119435922/mCGNdwkkU.png align="left")
- Select **/generatepassword** from the resource list. Click **Actions**. Click **Create Method**. Select ANY and click on the checkmark. 
- Select **Use Lambda Proxy integration**. Type in characters in **Lambda Function** text box. This generates a dropdown of your Lambda functions.  Select **PasswordGeneratorLambdaProxyIntegration**.
![create method](https://cdn.hashnode.com/res/hashnode/image/upload/v1668119837172/O1wp_ikYB.png align="left")
- Click **Save**.
## API Testing on Postman
- Select **Deploy API** under **Actions**.
- Set the stage name as *test*.
![test stage](https://cdn.hashnode.com/res/hashnode/image/upload/v1668120194797/Rp9ra6oEq.png align="left")

- Click **Deploy**.

![invoke URL](https://cdn.hashnode.com/res/hashnode/image/upload/v1668120681464/Q87efNpGe.png align="left")
- Copy the invoke URL and paste it on Postman for testing. Include the name of the resource on the URL. Set the query parameters for the password length.

![password length error](https://cdn.hashnode.com/res/hashnode/image/upload/v1668159468174/pQYSgTyog.png align="left")

![password length valid](https://cdn.hashnode.com/res/hashnode/image/upload/v1668121961186/B-taG88Oc.png align="center")

# References 📙
- [Tutorial: Build a Hello World REST API with Lambda proxy integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-api-as-simple-proxy-for-lambda.html) 

# Conclusion 🔚
Test the project URL using the web browser as well. Set the appropriate query parameter for the password length. https://y5r9r96qdc.execute-api.us-east-1.amazonaws.com/test

Comments, questions or suggestions related to this tutorial? Please share them with me. I'll be delighted to hear from you.

Cheers! 🎊 🍻 🎊 🍻 🎊 🍻