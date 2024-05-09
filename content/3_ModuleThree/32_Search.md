---
title: "Search"
chapter: true
weight: 2 
---

# Search

## Technical Details 

The Search service is used to find any data that is stored within Energy Data Insights on AWS.

When data is registered with Energy Data Insights on AWS, metadata records are stored via the Storage service and an event is generated, which initiates the indexing of that metadata. Indexing is done using Elasticsearch. The Search service is a secure interface to an internal, fully managed Elasticsearch index. The architecture of the Search service is shown below. 

![Search service architecture](/images/search.png)

Search execution flow can be diagramed as follows.
![Search service execution flow](/images/Searchexecution.png)

Search services allow running various queries against the EDI:

– Full-text 

– Field ranges

– Field match

– Sort, Aggregations, Filters

– Geo-Spatial coordinates

– Limits & Pagination

The Search API can be used for queries and queries with cursor. In order the user performing the search needs to have membership in <em>service.earch.user</em> (the searched records’ ACL as Viewers).

## Query structure
The search service follows [Apache Lucene syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html):    

    {
        "kind": "opendes:wks:master-data--Well:1.0.0 ",
        "query": "Text query",
        "offset": 0,
        "limit": 1000,
        "sort": {"fields": ["id"],
            "order": ["ASC"]},
        "queryAsOwner": true,
        "spatialFilter": {
            "field": "data.SpatialLocation.Wgs84Coordinates",
            "byDistance": {
                "point": {
                    "longitude": 5.1580810546875,
                    "latitude": 52.859180945520829
                },
                "distance": 10000
            }
        },
        "trackTotalCount": true,
        "returnedFields": ["id","kind"],
        "aggregatedBy": "kind"
    }

Some of the main query parameters are:

- kind: The kind of the record to query 

- query: The query string in Lucene query string syntax

- offset: The starting offset from which to return results

- limit: The maximum number of results to return from the given offset. If no limit is provided, then it will return 10 items

- sort: Allows you to add one or more sorts on specific fields
queryAsOwner: If true, the result only contains the records that the user owns. If false, the result contains all records that the user is entitled to see. Default value is false

- spatialFilter: A spatial filter to apply

- returnedFields: The fields on which to project the results

In this lab you will make API calls to retrieve data from the OSDU platform.
| Objective | Search for wells in the OSDU Platform |
| -- | --| 
| APIs | /api/search/v2/query/ |
| Security |  Cognito using UserID & Password |

## Lab 

### Create a new Python file 
Open the Search.py in your text editor <br>

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

Code below will prompt you for a user name and  password. Enter the username and password emailed to you.

    # Insert EDI region here
    region = ""

    # Insert username
    user_name=input('Username:')

    # Insert password
    password=getpass.getpass("Password:")

    # Insert client Id
    clientId = ""

    # Insert client secret
    clientSecret=""

    # Insert partitiion Id
    osdu_partition = ""
    
    # Insert platform URL
    osdu_platform_url = ""

    # Authenticate against AWS Cognito
    cognito = boto3.client('cognito-idp', region_name = region)

    # Compute secret hash
    message = username + clientId
    dig = hmac.new(clientSecret.encode('UTF-8'), msg=message.encode('UTF-8'), digestmod=hashlib.sha256).digest()
    secretHash = base64.b64encode(dig).decode()

    # Attempt to authenticate with the identity provider
    access_token = ""
    try:
        auth_response = cognito.initiate_auth(
            AuthFlow="USER_PASSWORD_AUTH",
            AuthParameters={"USERNAME": username, "PASSWORD": password, "SECRET_HASH": secretHash},
            ClientId=clientId,
        )
        access_token = auth_response["AuthenticationResult"]["AccessToken"]
        print("Success! You can now use the OSDU Platform APIs")        
    except:    
        print("An error occurred in the authentication process")    

    # Headers
    headers = {
        "Authorization": "Bearer " + access_token,
        "Content-Type": "application/json",
        "data-partition-id": osdu_partition
    }

### EDI Search URL
The osdu_platform_url below is specific to the session

    osdu_search_url = osdu_platform_url + '/api/search/v2/query/'
    print(osdu_search_url)

### Query 
The query below follows [Lucene Query Syntax](https://lucene.apache.org/core/2_9_4/queryparsersyntax.html) and provides wildcard, boolean operators and fuzzy search. More [Query Syntax](https://community.opengroup.org/osdu/documentation/-/wikis/Releases/R2.0/OSDU-Query-Syntax).

Note that the query below uses version 0.2.0 of the schema and will return all wells in the platform.

The below query will return all Wellbores which are belong to the Well ID 8787.

    # EDI query
    query_get_all_wells = {
        "kind": "osdu:wks:master-data--Wellbore:1.0.0",
        "query":"data.WellID:\"osdu:master-data--Well:8787:\""
    }

### Perform Search
The query for the search(above) is passed through the json parameter

    result = requests.post(osdu_search_url, headers=headers, json=query_get_all_wells)

### Inspect the result
Upon inspection, note the "totalCount" returned, and which represents the number of wells in the OSDU platform. 

The query however only returned 10 wells. This is by default and can be changed using the "limit" parameter. 

More on using "limit" below. 

    # Print result
    print(json.dumps(result.json(), indent=4))

### Query w/ limits, offsets and subsets

*limit* controls the number of results returned. Default = 10. Max = 1000.

*offset* is used to support pagination.

With offset = 100 and limit = 100, the query below returns the second set of 100 wells.

*returnedFields* is used when you want only a specific set of fields 

    # Query w/ limits, offsets and subsets
    query_2 = {
            "kind": "osdu:wks:master-data--Well:1.0.0",  
            "limit": 100,
            "offset": 100,
            "returnedFields": ["data.Source"]
        }

### Perform the second search

    result_2 = requests.post(osdu_search_url, headers=headers, json=query_2)

### Inspect results
Note both the number of wells and the limited subset of fields returned
    
### Lets try few more search queries

(a) List all wellbores which are belongs to Well ID starting with 8
    
    query = {
        "kind": "osdu:wks:master-data--Wellbore:1.0.0",
        "query": "data.WellID:(Well AND 8???)",
        "sort": {
            "field": ["data.WellID"],
            "order": ["DESC"]
        }
    }
    result = requests.post(osdu_search_url, headers=headers, json=query)
    print(json.dumps(result.json(), indent=4))

(b) Get logs with bottom measured depth between 2000 and 3000
    
    query = {
        "kind": "osdu:wks:work-product-component--WellLog:1.0.0",
        "query": "data.BottomMeasuredDepth:[2000 TO 3000]"
    }
    result = requests.post(osdu_search_url, headers=headers, json=query)
    RenderJSON(result.json())

(c) Geo-Spatial query
    
    query = {
        "kind": "osdu:wks:master-data--Well:1.0.0",
        "spatialFilter": {
            "field": "data.SpatialLocation.Wgs84Coordinates", "byDistance": {
                "point": {
                    "longitude": 5.1580810546875,
                    "latitude": 52.859180945520826 
                    },
                "distance": 100000
                }
            }
        }
    result = requests.post(osdu_search_url, headers=headers, json=query)
    print(json.dumps(result.json(), indent=4))

### Lab complete

Congratulations! You have completed this lab!

Great, you have successfully authenticated against EDI platform on AWS. Now, let's explore Dataset service.