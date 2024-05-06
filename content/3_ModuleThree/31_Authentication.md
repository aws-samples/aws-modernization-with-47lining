---
title: "Authentication"
chapter: true
weight: 1 
---

# Authentication

## Technical Details 

In this module, we will learn on how to use AWS Python SDK to geneate JWT bearer token which will be used to access OSDU APIs. Note that user creation is already done here, and the username and password should have been sent to your email prior to this workshop. The overall authentication flow is shown below.

![EDI Authentication](/images/authentication.jpg)

## Lab

### Create a new Python file 
Open the Authentication.py in your text editor <br>

### Import required libraries
Import the following required libraries into your Python code:

    import boto3
    import hmac
    import hashlib
    import base64

### Key Parameters
There are several key parameters that one needs in order to begin this authorization process like AWS cognito Client ID, User Name and Password.

Code below will prompt you for a user name and password. Enter the username and password emailed to you.
    
    # Insert EDI region here
    region = ""

    # Email that you used to sign up for this Workshop
    username = ""

    # Insert password for the above username
    password = ""

    # Insert client Id
    clientId=""

    # Insert client secret
    clientSecret=""

### Authenticate against AWS Cognito
    # Compute secret hash
    message = user_name + client_id
    dig = hmac.new(clientSecret.encode('UTF-8'), msg=message.encode('UTF-8'), digestmod=hashlib.sha256).digest()
    secretHash = base64.b64encode(dig).decode()

    # Authenticate with AWS Cognito
    client = boto3.client('cognito-idp', region_name = region)
    sdk_response = client.initiate_auth(
        AuthFlow='USER_PASSWORD_AUTH',
        AuthParameters={
            "USERNAME": username,
            "PASSWORD": password,
            "SECRET_HASH": secretHash
        },

        ClientId=clientId,
    )

    # Print responses
    print(sdk_response)
    access_token = sdk_response["AuthenticationResult"]["AccessToken"]
    refresh_token = sdk_response["AuthenticationResult"]['RefreshToken']
    print("Access Token: "+access_token+"\n")
    print("Refresh Token: "+refresh_token)    

### Lab complete

Congratulations! You have completed this lab! You have sucessfully authenticated against EDI platform on AWS.