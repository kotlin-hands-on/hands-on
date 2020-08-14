# Manually testing HTTP endpoints

Now that we have all the endpoints ready, it's time to test our application. While 
we can use any browser to test `GET` requests, we'll need a separate tool to test the other HTTP methods. Some options are `curl` or Postman – but if you're using [IntelliJ IDEA Ultimate Edition](https://www.jetbrains.com/idea/), you actually already have a client that supports `.http` files, allowing you to specify and execute requests – without even having to leave the IDE.

### Creating a customer HTTP test file

`.http` files are one way of specifying HTTP requests to be executed by different types of tools, including IntelliJ IDEA Ultimate Edition. Let's create a new file in the `test` directory of our project called `CustomerTest.http` and enter the following contents:

```kotlin
POST http://127.0.0.1:8080/customer
Content-Type: application/json

{
  "id": "100",
  "firstName": "Jane",
  "lastName": "Smith",
  "email": "jane.smith@company.com"
}


###
POST http://127.0.0.1:8080/customer
Content-Type: application/json

{
  "id": "200",
  "firstName": "John",
  "lastName": "Smith",
  "email": "john.smith@company.com"
}

###
POST http://127.0.0.1:8080/customer
Content-Type: application/json

{
  "id": "300",
  "firstName": "Mary",
  "lastName": "Smith",
  "email": "mary.smith@company.com"
}


###
GET http://127.0.0.1:8080/customer
Accept: application/json

###
GET http://127.0.0.1:8080/customer/200

###
GET http://127.0.0.1:8080/customer/500

###
DELETE http://127.0.0.1:8080/customer/100

###
DELETE http://127.0.0.1:8080/customer/500
```

Inside this file, we have now specified a bunch of HTTP requests, using all the supported HTTP methods of our API. IntelliJ IDEA now allows us to run each of these requests individually or all together. To really see what's going on, let's run them individually. But first, we need to make sure our API is actually reachable!

### Running our API server

Before we can run a request, we need to first start our API server. The easiest 
way to do this is to use IntelliJ IDEA and click on the Run icon in the gutter:

![Run Server](./assets/run-app.png)

Once the server is up and running, we can execute each request by pressing Alt+Enter or by using the Run icon in the gutter:

![Run POST Request](./assets/run-post-request.png) 

If everything is correct, we should see the output in the Run tool window:

![Run Output](./assets/run-output.png)

### Order endpoints

For the order endpoints we can follow the same procedure: we create a new file called `OrderTest.http` in the `test` directory of our project, and fill it with some HTTP requests:

```kotlin
GET http://127.0.0.1:8080/order/2020-04-06-01
Content-Type: application/json

###
GET http://127.0.0.1:8080/order/2020-04-06-01/total
Content-Type: application/json
```

Running these requests just as the ones before, we should see the expected output – detailed information about one order, and the total of the order respectively.
