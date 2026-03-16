What does the acronym HTTP stand for?


HyperText Transfer Protocol


Hyperlink Transfer Process


HyperText Terminal Program


High Transfer Transmission Protocol

Incorrect Answer
HyperText Transfer Protocol is correct because it is the standardized foundation for data communication on the World Wide Web, designed specifically to facilitate the transfer of hypermedia documents like HTML.

Which HTTP method is primarily used to request data from a specified resource?


POST


GET


PUT


DELETE

Correct Answer
GET is correct because it is the standard method used by browsers to retrieve information from a server, acting as a request to "read" or "fetch" a specific resource without altering its state.

In the client-server model, which component is responsible for initiating the HTTP request?


The Client


The Database


The Server


The Firewall

Correct Answer
The Client is correct because in the HTTP request-response cycle, the client (such as a web browser) acts as the initiator, sending a request to the server, which then processes the request and sends back a response.

Which part of an HTTP response indicates whether the request was successful, redirected, or resulted in an error?


Request Header


Uniform Resource Locator


Status Code


Payload Body

Correct Answer
Status Code is correct because it is a three-digit integer returned by the server (e.g., 200 for OK, 404 for Not Found) that tells the client the outcome of the request.

What does the "404" status code signify in HTTP?


Internal Server Error


Unauthorized Access


Not Found


Forbidden

Correct Answer
Not Found is correct because the 4xx class of status codes indicates client-side errors, and specifically, 404 informs the client that the server cannot find the requested resource.


Which HTTP method is typically used to submit data to be processed to a specified resource, often resulting in a change in state on the server?


POST


HEAD


OPTIONS


TRACE

Correct Answer
POST is correct because it is designed to send data to the server for processing, such as submitting a web form or uploading a file, which often leads to the creation of a new resource or a state change.

Which of the following is a component of an HTTP Request message?


Response Body


Request Line


Server Signature


Status Message

Incorrect Answer
Request Line is correct because an HTTP request consists of three parts: the Request Line (method, URL, version), Headers, and an optional Body. The other choices are either parts of a response or unrelated metadata.

What is the primary purpose of HTTP Headers?


To define the layout of the webpage


To encrypt the data in transit


To provide metadata about the request or response


To execute JavaScript on the server

Correct Answer
To provide metadata about the request or response is correct because headers act as key-value pairs that carry extra information about the client's browser, the server type, the content type being sent, and authentication credentials.


Which HTTP status code class indicates a successful request?


1xx


2xx


3xx


5xx

Correct Answer
2xx is correct because status codes starting with 2 (such as 200 OK or 201 Created) explicitly signal that the client's request was successfully received, understood, and accepted by the server.


Which method is used to modify or update an existing resource completely on the server?


GET


POST


PUT


DELETE

Incorrect Answer
PUT is correct because it is specifically defined to replace the target resource with the request payload, making it the standard choice for full updates compared to POST, which is often used for general processing.


What is a URL in the context of HTTP?


A programming language used to build servers


The address used to locate a specific resource on the internet


An encryption protocol for HTTP


A method for deleting resources

Correct Answer
The address used to locate a specific resource on the internet is correct because a Uniform Resource Locator (URL) provides the location and access method for a resource on the web.

Which of the following best describes the "stateless" nature of HTTP?


The server does not retain information about previous requests from the same client


The server maintains a permanent connection with the client until the browser closes


Every request must be authenticated with a password


The server stores all client data in the URL

Correct Answer
The server does not retain information about previous requests from the same client is correct because HTTP is stateless by design, meaning each request is handled as an independent transaction with no knowledge of prior interactions.

Which HTTP method is specifically intended to delete a specified resource?


REMOVE


PUT


POST


DELETE

Correct Answer
DELETE is correct because it is the semantic method defined in the HTTP specification for requesting that the server remove the resource identified by the provided URL.

What does a "500" status code imply?


Client sent a bad request


Resource was moved permanently


Server encountered an unexpected condition


Access is restricted to administrators

Incorrect Answer
Server encountered an unexpected condition is correct because the 5xx class represents server-side errors, with 500 being a generic error message indicating that the server could not fulfill the request due to an issue on its end.


Which header field is used in an HTTP request to indicate the type of media format being sent or expected?


Authorization


Content-Type


User-Agent


Host

Correct Answer
Content-Type is correct because it informs the recipient about the media type of the resource (e.g., text/html, application/json), allowing the browser or server to process the data correctly.