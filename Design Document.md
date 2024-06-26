# URL Shortening Service Design Document

## Goal

The goal is to generate a URL shortening service.

To shorten a URL, use a ?shorten query string; for example, <http://www.example.com/?shorten=http://some.url/etc/> will return a shortened URL <http://www.example.com/jhakdj>

To expand a URL simply request the shortened URL e.g. <http://www.example.com/jhakdj> and the system will redirect to the expanded URL.

To view the expanded URL without redirecting to it, simply add a + to the end of the URL, e.g. <http://www.example.com/xhgjgh+>

The solution will need to be highly scalable because it will need to handle at least 1 million requests per day, with peak traffic of 50 hits per second.

## Assumptions

* There is no need for a web frontend that consumes the service

## Technologies

### AWS Lambda

### DynamoDB

DynamoDB offers a fast and highly scalable solution for storing our urls and their respective short codes.

### AWS API Gateway

API Gateway allows for us to expose the URL shortening service as a RESTful API and integrates directly with AWS Lambda functions, our API Gateway setup will look like:

`POST ?shorten <shorten_url_lambda_function`

`GET /{shortened_url} <redirect_or_return_url_lamda_function>`

## Language

The Lambda functions will be written in Java, this ensures the team can get up and running quickly without having to be trained on a language that they are not as well equipped to use.

## Classes

### URLShortener

The `URLShortener` class should have a single `shorten(url)` method, that takes a url and returns a short code generated from the url.

// TODO: Explain the algorithm used to generate the short code.

### IURLStore

The `IURLStore` interface is an interface definition for interacting with storage, classes that implement it will be reponsible for inserting urls and their accompanying code into their storage medium.

### URLDynamoDBStore

The `URLDynamoDBStore` should implement the `IURLStore` interface, with implementations for `get` and `store` specific to DynamoDB.  

### URL Validator

The `URLValidator` should have a single `validate(url)` method that is resposible for determining if the passed in url is a valid url or not.

### URLRedirector

The URLRedirector class is responsible for redirecting the current request to a given url, in the constructor it should take the active request, with a method called redirect(url) that performs the redirect.

## Deployment

## Cost
