---
title: "Schema and storage"
chapter: true
weight: 4
---

# Schema

## Technical Details 

A schema is a data model definition. EDI Data Platform's Data Model is defined in rich JSON objects. Each schema has ID and version:
<authority:source:entity-type:major-version.minor-version.patch-version. For example, <em>osdu:wks:master-data--Well:1.0.0</em>. Schema scopes can be INTERNAL or SHARED. The Schema service architecture is presented below.

![Schema service architecture](/images/schema.png)

## Lab

This lab explores schemas and records in OSDU system. You will search for schemas with a query and take a look at one of them.
Then you will add a new record for this schema using storage service, based on an existing one, and later check if it has been added correctly.

| Objective | Understand Schemas and Records |
| -- | --| 
| APIs | /api/search/v2/query/  |
| . | /api/schema-service/v1/schema/  |
| . | /api/storage/v2/records/  |
| Schema | WellActivityProgramType |
| Security |  Cognito using UserID & Password |

### Create a new Python file 
Open the Storage and Schema.py in your text editor <br>

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

### Authenticate against AWS Cognito

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
            AuthParameters={"USERNAME": user_name, "PASSWORD": password,"SECRET_HASH": secretHash},
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
        "data-partition-id":  osdu_partition,
    }

### Searching for schemas
We will search for a schema related to Welllog. To do that we compose a query on schema kind. We will also aggregate them by kind.

    # Searching for schemas
    query = {"kind": "osdu:wks:work-product-component--WellLog:1.1.0", "aggregateBy": "kind"}
    schema_search_result = requests.post(osdu_platform_url + "/api/search/v2/query", headers=headers, json=query)
    schema_search_result.json()["aggregations"]

### Retrieve schema structure
We can see two schemas matching our query. Let's pick the WellActivityProgramType and see how the schema looks like.

    schema_id = "osdu:wks:work-product-component--WellLog:1.1.0"
    schema_get = requests.get(osdu_platform_url + "/api/schema-service/v1/schema/" + schema_id, headers=headers)
    print(json.dumps(schema_get.json(), indent=4))

### Search for records
Let's look at one of the records stored for Well Log.

    query = {"kind": schema_id}
    example_record = requests.post(osdu_platform_url + "/api/search/v2/query", headers=headers, json=query)
    print(json.dumps(example_record.json()["results"][0], indent=4))

### Adding records
Now we can add our own record for the Well Log schema and send it to the Storage API.
    
    # Adding records
    record_data = [
        {
            "kind": "osdu:wks:work-product-component--WellLog:1.1.0",
            "acl": {
                "viewers": [
                f"data.default.viewers@{osdu_partition}.example.com"
                ],
                "owners": [
                f"data.default.owners@{osdu_partition}.example.com"
                ]
            },
            "legal": {
                "legaltags": [
                f"{osdu_partition}-public-usa-dataset"
                ],
                "otherRelevantDataCountries": [
                "US"
                ]
            },
            "data": {
                "SamplingInterval": 0.09999999999990905,
                "Name": "7552_p0505_1984_comp.las",
                "Datasets": [
                "osdu:dataset--File.Generic:8d13eff3ccdf43038748d3030b34f903"
                ],
                "ResourceSecurityClassification": "osdu:reference-data--ResourceSecurityClassification:RESTRICTED:",
                "SamplingStop": 3103.8,
                "Description": "Well Log created by" + username,
                "TopMeasuredDepth": 1814.2001,
                "SamplingStart": 1814.2001,
                "WellboreID": "osdu:master-data--Wellbore:7853:",
                "BottomMeasuredDepth": 3103.8,
                "Curves": [
                {
                    "NumberOfColumns": 1,
                    "TopDepth": 1814.2,
                    "Mnemonic": "DEPT",
                    "BaseDepth": 3103.8,
                    "CurveUnit": "osdu:reference-data--UnitOfMeasure:M:"
                },
                {
                    "NumberOfColumns": 1,
                    "TopDepth": 1814.2,
                    "Mnemonic": "GR",
                    "BaseDepth": 3103.8,
                    "CurveUnit": "osdu:reference-data--UnitOfMeasure:GAPI:"
                },
                {
                    "NumberOfColumns": 1,
                    "TopDepth": 1814.2,
                    "Mnemonic": "DT",
                    "BaseDepth": 3103.8,
                    "CurveUnit": "osdu:reference-data--UnitOfMeasure:US%2FF:"
                }
                ]
            } 
        }
    ]
    put_record_body = json.dumps(record_data)

    # Submit PUT request to put record into OSDU platform
    put_record = requests.put(osdu_platform_url + "/api/storage/v2/records", headers=headers, data=put_record_body)
    print(json.dumps(put_record.json(), indent=4))
    

### Retrieve using Search API
    query = {
        "kind": "osdu:wks:work-product-component--WellLog:1.1.0",
        "query": "id:\""+ put_record.json()["recordIds"][0] + "\""
    }
    result = requests.post(osdu_platform_url + "/api/search/v2/query", headers=headers, json=query) 
    print(json.dumps(result.json(), indent=4))

### Lab complete

Congratulations! You have completed this lab!

Now, let's explore Entitlements and Legal services.