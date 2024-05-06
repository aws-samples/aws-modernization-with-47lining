---
title: "EDI Core APIs" 
chapter: true
weight: 3
---

# EDI Core APIs 

## OSDU Platform Components 

At a very high level OSDU Data Platform architecture can be visualized as follows:

![OSDU Data Platform Components](/images/osdu_data_components.jpg)

We will start exploring the CORE APIs of the platform, which include:

* Authentication - Get authorization token to access Data Platform
* Search - Search records in OSDU Data Platform
* Dataset - Store, retrieve, register files and metadata
* Schema - Create or update schemas and lifecycle
* Storage - Interact with the data at the record level
* Entitlements and Legal - Data security and compliance using groups and legal tags

{{% notice info %}}
<p style='text-align: left;'>
**REMOVE:** With the exception of _index.md, the module folders and filenames should be changed to better reflect their content, i.e. 1_Planning as the folder and 11_HowToBegin as the first submodule. Changing the "weight" value of the header is ultimately what reflects the order the modules are presented.
</p>
{{% /notice %}}

{{% notice warning %}}
The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how various AWS services can be architected to build a solution while demonstrating best practices along the way. These examples are not intended for use in production environments.
{{% /notice %}}

### Authentication Service
First, we explore the Authencation Service