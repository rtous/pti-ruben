# Making a Web API with Node.js

## Introduction

A web service is a generic term for a software function that is accessible through HTTP. Traditional web services usually relied in support protocols for data exchange (e.g. SOAP) and service definition (WSDL). However, nowadays the paradigm has evolved to a simplified form, usually called web APIs. Web APIs normally rely only in plain HTTP (plus JSON for serializing the messages). Their design is usually influenced by the [REST architectural style](https://en.wikipedia.org/wiki/Representational_state_transfer), though the most part of existing web APIs do not really comply with REST principles. Nowadays, the most part of client-server systems (e.g. web applications and mobile apps) design their back end as a combination of web APIs.  

The goal of this session is to create a simple web API with the Node.js JavaScript runtime environment. We will not bother to follow the REST principles, so it will not be a trully RESTful API.  

Each group will have to:

1. Tutorial: Follow a brief tutorial about how to create a simple web API with Node.js.
2. Assignment (basic part): Complete the lab assignment consisting on developing a simple car rental web API. 
3. Extensions: Optionally complete one of the suggested extensions.
4. Write a .pdf report describing the steps taken to complete the assignment, including screenshots of the application output.

*NOTE: The tutorial can be followed even if you don't have previous knowledge about JavaScript/Node.js.*

*NOTE: The tutorial was tested with Node.js 14.10.1 and Express 4.17.*

## 1 Making a Web API with Node.js, a quick tutorial

### 1.1 Setup

[(help for those wanting to use their own computers (through Docker))](./../docker.md)

#### 1.1.1 Booting the machine 

Conventional room: Select a 64-bit Linux image and login with your credentials.

Operating Systems room: Select the latest Ubuntu imatge with credentials user=alumne and pwd:=sistemes

#### 1.1.2 Prerequisites

Let's start by updating the Ubuntu's Package Index:

    sudo apt-get update

You need 'curl' installed (check it typing 'curl -V' in the terminal). If, for some strange reason, it's not, just do:

    sudo apt-get install curl

##### Troubleshooting

If you encounter lock errors with apt-get commands try killing the blocking process:

	ps aux | grep apt
	sudo kill PROCNUM

If that didn't solve the error try changing the default Ubuntu package repository in /etc/apt/sources.list. Replace http://es.archive.ubuntu.com/ubuntu/ by http://en.archive.ubuntu.com/ubuntu/. You can use this command:

	sudo sed -i 's|http://es.|http://en.|g' /etc/apt/sources.list

#### 1.1.3 Install Node.js

Run the following command:

	curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -

Install Node.js:
	
	sudo apt-get install -y nodejs


#### 1.1.4 Setup a directory for your application and install the Express package

Create a directory to contain your application: 

    cd
    mkdir myapp
    cd myapp

Create a package.json file that will register the dependencies of your application (more information [here](https://docs.npmjs.com/files/package.json)). Run the following and press RETURN to accept all default values:

	npm init

Now install Express in the myapp directory and save it in the dependencies list:

	npm install express --save

The save flag will add Express as a dependency within the package.json file.

We can check which version of Express we installed by typing:

    npm list 

### 1.2 A simple web server
    
A web API is a specific type of web (HTTP-based) service. Let's start by programming a basic web server with Node.js:   

Edit a new file named "app.js" within $HOME/myapp with this content:

```js
const express = require('express')
const app = express()
const port = 8080

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`PTI HTTP Server listening at http://localhost:${port}`)
})
```

This example:

1. Creates an Express "application" with the express() function. 
2. With a call to the app.get method specifies that HTTP GET requests to the '/' path will be handled by a [callback function](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function) that sends 'Hello World!' back to the client who issued the request (e.g. a browser).
3. Starts a UNIX socket and listens for connections on the port 8080 

*NOTE: In JavaScript "=>" is more concise way for defining a function (more details [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions))"

*NOTE: In JavaScript "=>" is more concise way for defining a function (more details [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions))"

https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/Introduction

Let's launch the server:

    node app.js &

test in browser: http://localhost:8080

Kill the process before going through the next steps.
    
### 1.3 URL routing

https://expressjs.com/en/guide/routing.html
    
An web API exposes different functionalities. These functionalities are accessed through different HTTP methods (POST, GET, PUT, PATCH or DELETE) and URL routes or endpoints. We need a mechanism that let us map the requests received by the server to different functions in our code. 

Let's modify our app.js and add the following below the app.get('\'...
```js
app.get("/endpoint1", (req, res, next) => {
 res.send('Received request at /endpoint1')
});
```
Open http://localhost:8080/endpoint1 in your browser.

URL 
```js
app.get("/endpoint1/*", (req, res, next) => {
 res.send('Received request at /endpoint1 and the path is '+req.path) 
});
```
Open http://localhost:8080/endpoint1 in your browser.
   
### 1.4. JSON 

Typically an endpoint has to deal with more complex input and output parameters. This is usually solved by formatting the parameters (input and/or output) with JSON. 

#### 1.4.1 A JSON response

Let's modify our webserver.go to include a JSON response.

```js
app.get("/endpoint2", (req, res, next) => {
 res.json(["Tony","Lisa","Michael","Ginger","Food"]);
});
```
Rebuild, run and open http://localhost:8080/endpoint2 in your browser.

Let's try also to call the server with curl:

	curl -H "Content-Type: application/json" http://localhost:8080/endpoint2

#### 1.4.2 A JSON request

Let's now add a new endpoint that accepts JSON as input. First of all add the following struct:
```go
type RequestMessage struct {
    Field1 string
    Field2 string
}
```
And "io" and "io/ioutil" to the imports:
```go
import (
    	"fmt"
    	"log"
    	"net/http"
    	"github.com/gorilla/mux"
    	"encoding/json"
	"io"
	"io/ioutil"
	) 
```
Then add a new route:
```go
router.HandleFunc("/endpoint2/{param}", endpointFunc2JSONInput)
```
And its related code:
```go
func endpointFunc2JSONInput(w http.ResponseWriter, r *http.Request) {
    var requestMessage RequestMessage
    body, err := ioutil.ReadAll(io.LimitReader(r.Body, 1048576))
    if err != nil {
        panic(err)
    }
    if err := r.Body.Close(); err != nil {
        panic(err)
    }
    if err := json.Unmarshal(body, &requestMessage); err != nil {
        w.Header().Set("Content-Type", "application/json; charset=UTF-8")
        w.WriteHeader(422) // unprocessable entity
        if err := json.NewEncoder(w).Encode(err); err != nil {
            panic(err)
        }
    } else {
        fmt.Fprintln(w, "Successfully received request with Field1 =", requestMessage.Field1)
        fmt.Println(r.FormValue("queryparam1"))
    }
}
```

*This code involves some tricky error handling. [Go’s approach to error handling](https://blog.golang.org/error-handling-and-go) is one of its most controversial features. Go functions support multiple return values, and by convention, this ability is commonly used to return the function’s result along with an error variable. By convention, returning an error signals the caller there was a problem, and returning nil represents no error. The panic built-in function stops the execution of the function and, in a cascading way, of the chain of caller functions, thus stopping the program.*

Rebuild and run. In order to submit a JSON request we will use curl instead of the browser. Open a new terminal and type:

    curl -H "Content-Type: application/json" -d '{"Field1":"Value1", "Field2":"Value2"}' http://localhost:8080/endpoint2/1234?queryparam1=Value3

(while curl is enough for this session, for your project you could take a look at [POSTMAN](https://www.getpostman.com/))

**WARNING: The fields of the request and response structs MUST START WITH A CAPITAL LETTER.**

**WARNING: The fields of the request and response structs MUST ONLY INCLUDE ALPHANUMERIC CHARACTERS (AVOID UNDERSCORES, ETC.).**


## 2 Lab assignment 

### 2.1 Creating your own car rental web API (9 points)

#### 2.1.1 Description

As an example web API you will create a simple car rental web API. It will consist in two functionalities:

- Request a new rental: An endpoint to register a new rental order. Input fields will include the car maker, car model, number of days and number of units. The total price of the rental will be returned to the user along with the data of the requested rental.
 
- Request the list of all rentals: An endpoint that will return the list of all saved rental orders (in JSON format). 

In order to keep the rentals data (to be able to list them) you will need to save the data to the disk. A single text file where each line represents a rental will be enough (though not in a real scenario). ANNEX 1 and ANNEX 2 provide help for witing and reading comma-separated values to/from a CSV file.


### 2.2 Extension (1 point)

In order to obtain the maximum grade you can complete any of the following extensions: 

* Add to the report a 1-page explanation of the pros and cons of Go compared to another alternative (e.g. C++, Java, Rust, etc.).
* Dockerize your web application. Some help [here](./dockerize.md).

## 3.  Submission

You need to upload the following files to your BSCW's lab group folder before the next lab session:

* A tarball containing the source files.
* A .pdf with a report describing the steps taken to complete the assignment, including screenshots of the application output.   


## ANNEX 1. Writing comma-separated values to a CSV file

An easy way to save the list of rentals could be a text file with lines containing comma-separated values (CSV). One rental per line. This way you can save a new rental just adding a line at the end of the file.

	(need to add "encoding/csv" and "os" to imports)
```go
func writeToFile(w http.ResponseWriter, values []string) {
    file, err := os.OpenFile("rentals.csv", os.O_APPEND|os.O_WRONLY|os.O_CREATE, 0600)
    if err!=nil {
        json.NewEncoder(w).Encode(err)
        return
        }
    writer := csv.NewWriter(file)
    writer.Write(values)
    writer.Flush()
    file.Close()
}
```

Yoy may call your function from endpointFunc2JSONInput this way:

```go
writeToFile(w, []string{requestMessage.Field1, requestMessage.Field2})
```

If you don't specify a file path the file will be saved in the directory from which you launch the command. 

## ANNEX 2. Reading a CSV file

In order to read all the lines from a CSV file and to put them within a JSON response you can do:

```go
var rentalsArray []ResponseMessage
file, err := os.Open("rentals.csv", )
if err != nil {
    log.Fatal(err)
} 
reader := csv.NewReader(file)
for {
    line, err := reader.Read()
    if err != nil {
        if err == io.EOF {
            break
        }
        log.Fatal(err)
    }
    rentalsArray = append(rentalsArray, ResponseMessage{Field1: line[0], Field2: line[1]})
}
json.NewEncoder(w).Encode(rentalsArray)

```
     
