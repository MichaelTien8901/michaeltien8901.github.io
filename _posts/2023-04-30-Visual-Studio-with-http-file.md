---
layout: post
title: "Visual Studio Code with http file"
date: 2022-12-29
---

## [Bye Bye Swagger and postman](https://medium.com/@atakankorez/bye-bye-swagger-and-postman-built-in-rest-client-for-vs-2022-9be7df9322e9)

* The link described built-in REST client in Visual Studio. This http function is also available in VS Code with REST client(Huachao Mao).

Let’s talk about how to create an .http file and the rules for using this structure.

* Firstly, create a Web API project and add CRUD operations to the controller in the project.

* Then, add a new item to this project. It’s important to note that the extension for this item must be either .http or .rest.

* If you want to define an Http method, you must put `###` above this method. Thus, it will be understood by VS that the next line below is an Http method and a green Arrow will be added to its left.

* You can add any description you want by using the `#` sign on one line of the Http method.

```http
###
# this is the method return post data with id is 1 from jsonplaceholder
GET https://jsonplaceholder.typicode.com/posts/1

```

* You can use the example below to send information from the body in POST and PUT methods. An important point here is that there is at least one line space between the Content-type and the body object. Otherwise, you will get an error

```http
###
POST https://localhost:7034/api/products
Content-Type: application/json

{
    "id": 4,
    "name": "Product 4",
    "price": 45.00
}

###
PUT https://localhost:7034/api/products
Content-Type: application/json

{
    "id": 3,
    "name": "Product 3",
    "price": 60.00
}

```

* You can also define variables from the .http file. While defining a variable, the variable name starts with the @ sign, and no quotation marks are used in the values to be assigned, regardless of the type of the variable.

```http
@hostname = localhost
@port = 7034
@url = https://{{hostname}}:{{port}}/api/products
@id = 3

###
# Get all products
GET {{url}}

###
POST {{url}}
Content-Type: application/json

{
    "id": 4,
    "name": "Product 4",
    "price": 45.00
}

###
PUT {{url}}

{
    "id": {{id}},
    "name": "Product 4",
    "price": 60.00
}


###
DELETE {{url}}/{{id}}

```

* To use the methods contained in the .http file, VS 2022 must be run in debug mode. Then, the method is run with the green arrow key on the left side of each method. Since it works in debug mode, it can be debugged by setting breakpoints to the methods in the controller.

* Application [link](https://github.com/akorez/Built-In_RestClient_Example).