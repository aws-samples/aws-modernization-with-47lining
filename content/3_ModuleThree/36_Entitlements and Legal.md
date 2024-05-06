---
title: "Entitlements and Legal"
chapter: true
weight: 6 
---

# Entitlements and Legal

## Technical Details 

The Entitlements and Legal service is used to find any data that is stored within Energy Data Insights on AWS.

When data is registered with Energy Data Insights on AWS, metadata records are stored via the Storage service and an event is generated, which initiates the indexing of that metadata. Indexing is done using Elasticsearch. The Entitlements and Legal service is a secure interface to an internal, fully managed Elasticsearch index. The architecture of the Entitlements and Legal service is shown below. 

![Entitlements and Legal service architecture](/images/search.png)

This lab explores on groups and how to add a user into appropriate OSDU group using entitlements service. Until we assign a user into groups the user will not be able to access any of the OSDU services or records.

| Objective | Users and Groups |
| -- | --| 
| APIs | api/entitlements/v2/groups/ |
| Security |  Cognito using UserID & Password |

## Lab

### Create a new Python file 
Open the Entitlements.py in your text editor <br>

### Import required libraries
Import the following required libraries into your Python code:

    import boto3
    import hmac
    import requests
    import json
    import getpass
    import json
    import base64
    import hashlib

### Key Parameters
There are several key parameters that one needs in order to begin this authorization process like AWS cognito Client ID, User Name and Password.

    # Insert EDI region here
    region = ""

    # Insert username
    user_name=input('Username:')

    # Insert password
    password=getpass.getpass("Password:")

    # Insert client Id
    client_id = ""

    # Insert client secret
    clientSecret=""

    # Insert partitiion Id
    osdu_partition = ""

    # Insert platform URL
    osdu_platform_url = ""

    # Authenticate against AWS Cognito
    cognito = boto3.client('cognito-idp', region_name = region)

    # Compute secret hash
    message = user_name + client_id
    dig = hmac.new(clientSecret.encode('UTF-8'), msg=message.encode('UTF-8'), digestmod=hashlib.sha256).digest()
    secretHash = base64.b64encode(dig).decode()

    # Attempt to authenticate with the identity provider
    access_token = ""
    try:
        auth_response = cognito.initiate_auth(
            AuthFlow="USER_PASSWORD_AUTH",
            AuthParameters={"USERNAME": user_name, "PASSWORD": password, "SECRET_HASH": secretHash},
            ClientId=client_id,
        )
        access_token = auth_response["AuthenticationResult"]["AccessToken"]
        print("Success! You can now use the OSDU Platform APIs")    
    except:    
        print("An error occurred in the authentication process")

# Define headers
headers = {
    "Authorization": "Bearer " + access_token,
    "Content-Type": "application/json",
    "data-partition-id": osdu_partition
}

### OSDU Entitlements URL
The osdu_platform_url below is specific to the workshop.

    osdu_entitlement_url = osdu_platform_url + '/api/entitlements/v2'
    print(osdu_entitlement_url)

### List all groups
List all groups with your EDI instance.

    # List all groups
    group_result = requests.get(osdu_entitlement_url + '/groups', headers=headers)
    print(json.dumps(group_result.json(), indent=4))


### Get members for a group
Get a user member information for all members belonging to a specific group.

    # Get members for a group
    print("Get members for a group:"+group_result.json()["groups"][0]["email"])
    result = requests.get(osdu_entitlement_url + '/groups/' + group_result.json()["groups"][0]["email"] + '/members?limit=100', headers=headers)
    print(json.dumps(result.json(), indent=4))

### List groups in which you are assigned to
Get all groups which you are a member of.

    # List groups in which you are assigned to
    result = requests.get(osdu_entitlement_url + '/members/' + user_name + '/groups?type=NONE', headers=headers)
    print(json.dumps(result.json(), indent=4))

### Create a new group
Create a new group for testing.

    # Create a new group
    group_name="data.testing.user"+user_name.split('@')[0]
    print('My group name is: '+group_name)
    group_body = {
        "name": group_name,
        "description": "Meant for testing"
    }

    # Check if you are already part of this group from previous run
    existing_group = ""
    for group in result.json()["groups"]:
        if group["name"] == group_name:
            existing_group = group

    # Add new group if it does not already exist
    if(existing_group!=''): 
        print(f"You are already a member of group with name {group_name}.")
    else: 
        print('My group name is: '+group_name)
        group_body = {
            "name": group_name,
            "description": "Meant for testing"
        }
        group_result = requests.post(osdu_entitlement_url + '/groups', headers=headers, json=group_body)
        print ("Added new group:")
        print(json.dumps(group_result.json(), indent=4))

### Lab complete

Congratulations! You have completed this lab!