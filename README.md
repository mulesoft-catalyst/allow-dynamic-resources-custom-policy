# Allow Dynamic Resources using Custom Policy

# Introduction
API description formats such as [RAML](https://raml.org/developers/raml-200-tutorial) or [OAS](https://github.com/OAI/OpenAPI-Specification) provides developers with a standard, programming language-agnostic interface description for HTTP APIs, which allows both humans and computers to discover and understand the capabilities of a service without requiring access to source code, additional documentation, or inspection of network traffic. It also provides a way to validate the incoming HTTP request against a schema.
However, there are certain scenarios where schema validation can be challenging. One such scenario is dynamic resource path validation using OAS or RAML API specification. While defining the resources if a developer is not aware about resource heararchy. They want to express the hierarchy in the URI using a path as a parameter /resources/foo, /resources/foo/bar, and /resource/foo/bar/baz would all map to it with path parameter values of /foo, /foo/bar, and /foo/bar/baz, respectively. As per [RFC 6570](https://www.rfc-editor.org/rfc/rfc6570) this can be represented by /resources{+path}. Few API Gateway tools also allows their own extensions to define dynamic resource path using notation like {path:.+|}. 
While various notations can be used to define dynamic resources, it is currently not possible to validate these notations against the API specification using APIKIT router MuleSoft proxy validation module. This custom policy tries to solve this problem by allowing users to define resource path detail inside the custom policy perform resource validation.

# Proposed Solution
This custom policy tries to solve the prblem of dynamic resource path by using the power of regular expression. It allows users to input dynamic resources in custom policy input parameters using regular expression.
  Replacement logic 
  - Replace {path:.+|} or {+path} with *
  - Replace {any-uri-param} with w*
  - Replace any-uri-param with any-uri-param
  - Input all HTTP methods in Upper-case
  - URI Parameters are case sensitive

  Example:
  if your API spec has below URI paramaters and Methods

  - /cass/a-field/{field-id}:<br/>
      get:
  - /{+path}:<br/>
      get: 
  - /field/with-cass: <br/>
      post:


  Replace them by - 
  - GET:/cass/a-field/w*
  - GET:/*
  - POST:/field/with-cass
  
  The policy internally uses DW to match the incoming method and Uri path with user provided values in custom policy. If the match is not succesful it fails incoming request and throws error code 404.
  
# Publish policy to Exchange
The new custom policy sdk can be used for both Mule 4 and FlexGateway. This example is for Mule 4 but can be modified to be used with FlexGateway as well. This [link](https://docs.mulesoft.com/gateway/1.1/policies-custom-flex-getting-started) has official documentation on custom policy.
  
## Step 1 - Create policy asset in exchange
  Login to anypoint.mulesoft.com -> Exchange -> Publish New Asset
  
<img width="819" alt="step1-1" src="https://user-images.githubusercontent.com/44620039/189748395-1820e5c7-2996-40e4-beb7-4562718af84f.png">

## Step 2 - Provide the details
Fill in details of custom policy and publish to exchange
![step2](https://user-images.githubusercontent.com/44620039/189750636-f3a6ccd1-7d2f-4b21-a924-8fea42e2c391.jpg)

## Step 3 - Add Implementation
Click on implementation tab -> Add implementation button
![step3](https://user-images.githubusercontent.com/44620039/189751345-35322651-a400-406e-8272-6d4d41d618c8.jpg)

## Step 4 - Create policy jar
After you downloaded the code from GitHub go to the local folder which contains pom.xml and run below command 
```
mvn clean package
```
It will create custom policy binary under target folder

## Step 5 - Upload implementation binary
Select the custom binary file and meta file.
Provide name, asset id and asset version for implementation asset
<img width="451" alt="step4" src="https://user-images.githubusercontent.com/44620039/189753361-2b4e5ab4-79d3-438f-a341-e2d910047ea2.png">

# Apply policy in API Manager
Click on Policies tab under API Manager API instance where you want to apply the policy.
You will see Allow Dynamic Resources under custom section.
<img width="1242" alt="step5" src="https://user-images.githubusercontent.com/44620039/189754840-7d67786d-378f-4b65-a28f-84942d720599.png">

Input allowed method and resources combination with one pair of Method and Resource in one row. 
Use +Add button to keep adding more.
Click on Apply button once all methods and resources are added.
<img width="921" alt="step6" src="https://user-images.githubusercontent.com/44620039/189755679-9117bf6a-30e0-4d82-b3e6-05ffadc4cfe4.png">

In case the format is not correct the custom policy UI will throw error.
![step7](https://user-images.githubusercontent.com/44620039/189756202-98f3befb-43bf-421c-9ba4-e6e00dfb7501.png)

# Test Policy

Sample Request - 
```
curl -k -X PUT \
  https://demortfnginx.eastus.cloudapp.azure.com/test-dynamic-method/hello/abc \
  -H 'Content-Type: application/json' \
  -d '{
        "hello": "mule"
      }'
```

Sample Response:
```
{
  "error": "Resource Not Found - PUT:/hello/abc"
}
```
