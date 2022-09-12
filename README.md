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
  
