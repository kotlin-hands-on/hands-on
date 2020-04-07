# Manual testing HTTP endpoints

Now that we have all the endpoints ready, it's time to test our application. While 
we can use any browser to test GET verbs, we'll need another tool such as Postman, curl, or IntelliJ IDEA
to test other verbs. 

We'll be using [IntelliJ IDEA](https://www.jetbrains.com/idea).

## Creating a Customer HTTP test file

Let's create a new file in our project called `CustomerTest.http` and 
enter the following contents:

```
POST http://0.0.0.0:8080/customer
Content-Type: application/json

{
  "id": "100",
  "firstName": "Jane",
  "lastName": "Smith",
  "email": "jane.smith@company.com"
}


POST http://0.0.0.0:8080/customer
Content-Type: application/json

{
  "id": "200",
  "firstName": "John",
  "lastName": "Smith",
  "email": "john.smith@company.com"
}


POST http://0.0.0.0:8080/customer
Content-Type: application/json

{
  "id": "300",
  "firstName": "Mary",
  "lastName": "Smith",
  "email": "mary.smith@company.com"
}



GET http://0.0.0.0:8080/customer
Accept: application/json

GET http://0.0.0.0:8080/customer/200

GET http://0.0.0.0:8080/customer/500

DELETE http://0.0.0.0:8080/customer/100

DELETE http://0.0.0.0:8080/customer/500
```

IntelliJ IDEA now allows us to run each of these individually or all together. In our case
we want to run it individually. 

## Running our server 

Before we can run a request, we need to first run the server. Once again the easiest 
way to do this is to use IntelliJ IDEA and click on the `Run` icon in the gutter

![Run Server](./assets/run-app.png)

Once the server is up and running, we can execute each request by hitting Alt+Enter or the Run icon

![Run POST Request](./assets/run-post-request.png) 

If everything is correct, we should see the output in the Run tool window

![Run Output](./assets/run-output.png)

## Order endpoints

For the order endpoints we can follow the same procedure and define a new HTTP request
file

```
GET http://0.0.0.0:8080/order/2020-04-06-01
Content-Type: application/json


GET http://0.0.0.0:8080/order/2020-04-06-01/total
Content-Type: application/json
```



