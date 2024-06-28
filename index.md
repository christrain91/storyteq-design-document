# URL Shortening Service Design Document

## Introduction

This design document outlines the design for a URL shortening service that allows users to shorten long URLs.

To shorten a url a request is made to the base url of the application with the query string `?shorten=http://url_to_shorten.com`, this will return the shortened url.

When a shortened URL is navigated to the response will redirect to the original URL, if there is a + on the end of shortened URL then just the original URL should be returned.

The service will need to be able to handle up to 1 million requests per day with peak traffic of 50 hits per second.

## Assumptions

* There is no need for a web frontend that consumes the service.
* There are no specific additional security requirements for data protection.

## Technologies

### AWS Services

* API Gateway - For handling HTTP requests.
* Lambda - For executing code without provisioning servers.
* DynamoDB - For storing the URLs and their generated short-form codes.

Using AWS Lambda functions will provide significant scalability benefits to the application. Lambda functions allow the backend to automatically scale in response to load ensuring high availability and performance without the need for manual intervention.

### Libraries

* Java SDK for AWS: To interact with AWS services.

## Data Persistence

The urls and their short-form codes will be stored in a DynamoDB table.

### Table Schema

```yml
Table Name: URLMappings
Primary Key: ShortCode (String)
Attributes:
  - OriginalURL (String)
  - CreatedAt (Timestamp)
```

### Example Record

| ShortCode | OriginalURL | CreatedAt |
| --------- | ----------- | --------- |
| uk34dm | <https://www.google.com> | 2024-06-26T13:27:51.413Z |

The main benefit of using DynamoDB is its scalability, it can store a vast amount of records with little degradation of performance.

## Language

The Lambda function will be written in Java, this ensures the team can get up and running quickly without having to be trained on a language that they are not as well equipped to use.

## Code Design

### Interfaces

#### ICodeGenerator

Methods:

```java
  public String generate ()
```

* Returns:
  * `String`: The generated short code

#### IURLStore

Methods:

```java
  public String get (String id)
```

* Parameters:
  * `String id`: The unique ID to get the url for

* Returns:
  * `String`: The url that was found with a matching ID

```java
  public String store (String id, String url)
```

* Parameters:
  * `String id`: The unique ID to store the url with
  * `String url`: The url to store

* Returns:
  * `String`: The id that was used to store the url

### Classes

### ShortCodeGenerator

The `ShortCodeGenerator` implements the `ICodeGenerator` interface.

Methods:

```java
  public String generate ()
```

* Returns:
  * `String`: The generated short code

#### Generating A Short Code

1. Define the character set, using the a-z, A-z and 0-9 characters.
2. Randomly select 6 characters
3. Join them together into a single string and return the result

### URLShortener

This class is responsible for the url shortening process, it takes a url generates a unique ID,  stores the ID and url in the database, then returns the shortened url.

Constructor:

```java
  public constructor (IURLStore store, ICodeGenerator codeGenerator)
```

* Parameters:
  * `IURLStore store`: The store for the urls and their short codes
  * `ICodeGenerator codeGenerator`: The generator to use to generate the short codes

Methods:

```java
  public String shorten (String url)  
```

* Parameters:
  * `String url`: The url to shorten

* Returns:
  * `String`: The fully formed short url

```java
  private String getUniqueShortCode ()
```

* Returns:
  * `String`: The unique short code

Note: This method utilizes the generator to produce a short code and subsequently checks if the generated code is already in use within the store. If the code is available, it is returned. If not, the method continues generating codes until a unique one is identified and returned.

### URLDynamoDBStore

The `URLDynamoDBStore` implements the `IURLStore` interface.

Methods:

```java
public String get (String id)
```

* Parameters:
  * `String id`: The short code of the url to retrieve

* Returns:
  * `String`: The full url for the passed link

```java
  public String store (String id, String url) 
```

* Parameters:
  * `String id`: The id to store the url against
  * `String url`: The url to store

* Returns:
  * `String`: The id that the url was stored at

### URLValidator

Methods:

```java
  public Boolean isValid (String url)
```

* Parameters:
  * `String url`: The url to validate

* Returns:
  * `Boolean`: A boolean to indicate if the url is valid

### URLRedirector

Constructor:

```java
  public constructor (IURLStore store) 
```

* Parameters:
  * `IURLStore store`: The store to use to get a full url from a short code

Methods:

```java
  public APIGatewayProxyResponseEvent redirectFromShortCode (String shortCode)
```

* Parameters:
  * `String shortCode`: The short code of the url you want to redirect to

* Returns:
  * `APIGatewayProxyResponseEvent`: The generated response to return from the Lambda function to perform a redirect to the full url.

### APIHandler

The API Handler class should implement `RequestHandler<APIGatewayProxyRequestEvent, APIGatewayProxyResponseEvent>` and is responsible for handling the incoming request.

Methods:

```java
  public APIGatewayProxyResponseEvent handleRequest (APIGatewayProxyRequestEvent request, Context context)
```

* Parameters:
  * `APIGatewayProxyRequestEvent request`: The request that has been made
  * `Context context`: The request context

* Returns:
  * `APIGatewayProxyResponseEvent`: The response to send

The handleRequest function should determine the type of request is it a request to shorten/redirect/get and invoke the necessary class methods to do so.

## Factories

### URLStoreFactory

This factory is responsible for creating an instance of the class we'll use for persisting the URLs in the the application, initially this will just be an instance of the URLDynamoDBStore, this approach allows us to switch database technologies simply by switching the class that we create an instance of.

Methods:

```java
  public IURLStore build ()
```

* Returns:
  * IURLStore: An instance of a class that implements the IURLStore interface

### CodeGeneratorFactory

This factory is responsible for creating an instance of a class that implements the ICodeGenerator interface, which will initially be an instance of the ShortCodeGenerator class. This approach allows us to switch to using a different method for generating short codes.

Methods:

```java
  public ICodeGenerator build ()
```

* Returns:
  * `ICodeGenerator`: An instance of a class that implements the ICodeGenerator interface

### URLShortenerFactory

This factory should create an instance of the URLShortener class passing in instances of classes built by the URLStoreFactory and CodeGeneratorFactory as parameters.

Methods:

```java
  public URLShortener build ()
```

* Returns:
  * `URLShortener`: An instance of the URLShortener

## Cost

Using the AWS Pricing Calculator the following price estimates have been produced. All prices are before tax.

### Breakdown

| Service Name | Upfront Cost | Monthly Cost |
| --------- | ----------- | --------- |
| Amazon API Gateway | 0.00 USD | 35.29 USD |
| AWS Lambda | 0.00 USD | 6.08 USD |
| Amazon DynamoDB | 0.00 USD | 4.83 USD |

This is based on 1 million daily requests, with 1/100 requests being shorten requests rather than a redirect, the difference being redirect does not require a write to the database. The Lambda function has been provisioned with 512mb of RAM.

### Totals

Total Upfront Cost: **0.00 USD**

Monthly Cost: **46.20 USD**

Yearly Cost **554.40 USD**
