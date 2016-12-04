## Method Definitions

The request method is the primary source of request semantics. It indicates the purpose of request and what is expected by the client as a successful result.
The request method's semantics might be specialized by the semantics of some header.

The GET, HEAD and OPTIONS methods are said to be safe, because nothing will happen on the server as a result of the HTTP request. Semantically they are considered as read-only methods.

A request method is considered idempotent that can be called many times without different outcomes. PUT, DELETE, and safe request methods are idempotent.

Request methods can be defined as cacheable to indicate that responses are allowed to be stored for future reuse on the server. GET, HEAD and in some cases POST can be cached.
### Overview of HTTP methods
Methods | Idempotent | Safe | Resource type
--- | --- | --- | ---
**GET** | yes | yes | Entity / Collection
**HEAD** | yes | yes | Entity / Collection
**OPTIONS** | yes | yes | Entity / Collection
**POST** | no | no | Entity
**PUT** | yes | no | Entity
**PATCH** | no | no | Entity
**DELETE** | yes | no | Entity

The next section contains information about the main characteristics of each HTTP method. You can find more detailed information about using those methods in the [Resource Entity endpoins](<<anchor_placheolder>>) and [Collection resource endpoints](<<anchor_placheolder>>).

### <a name="GET"></a>GET
Use GET for the following:
* Get a representation of a resource

### <a name="HEAD"></a>HEAD
The HEAD method is identical to [GET](#GET) except that the server must not send a message body in the response.

Use HEAD for the following:
* Find out about a resource (e.g., determine its type) without getting it
* See if an object exists, by looking at the status code of the response
* Test if the resource has been modified, by looking at the ETag

### <a name="OPTIONS"></a>OPTIONS
Use OPTIONS for the following:
* To determine the options associated with a resource or the capabilities of a server, without implying a resource action

When the clien sends an OPTIONS request to the server the server responses with the list of supported methods via the [Allow](<<link_to_Allow>>) header.

<<example>>

### <a name="POST"></a>POST
Use POST for the following:
* To create a new resource
* To perform any unsafe or nonidempotent operation via calling a SOA endpoint
* To run queries with large inputs

### <a name="PUT"></a>PUT
Use PUT for the following:
* To create new resources only when client is able to assign URIs for the resources. To let clients assign URIs, the server needs to explain to clients how URIs on the server are organized, what kind of URIs are valid, and what kind are not
* To override an existing resource with a modified version of the one residing on the server

If a new resource is created, the server send back 201 (Created) response.
If an existing resource is modified, either the 200 (OK) or 204 (No Content) response codes should be sent to indicate successful completion of the request.
If the server does not support resource creation, return 404 (Not Found) status to the client.
#### Optimistic Concurrency Control
We must handle in some cases concurrent updates on a resource, but all resources are suited for concurrency control.
Once the ETag is received, it allows the client to perform the update. However, the changes will only apply if the ETag is still valid.
If the resource exists, take the following steps:
* First client must obtain an ETag for the update operation
* The client has to send the new representation of the resource along with the If-Match condicional header that contains the ETag
* If the supplied If-Match header do not match the actual ETag value of the resource on the server, the server send back 412 (Precondition Failed) response.
* If the supplied conditions match the server updates the resource, and sends back 200 (OK) or 204 (No Content) with the updated value of ETag header. The response also includes a Content-Location header.

### <a name="PATCH"></a>PATCH
Use PATCH for the following:
* To support partial updates in contrast with PUT that is defined for the complete update or replacement of a resource

The client can check whether te PATCH is supported via the Allow header of the OPTIONS response or the resource can include an Accept-Patch header with the supported media types for the PATCH method.
Be aware that the PATCH method sometimes blocked in some firewall connfiguration, therefore the resource should always support PUT method beside PATCH.
Similar to [PUT](#PUT) method PATCH can handle [Optimistic Concurrency Control](<<link_to_optimistic_concurrency_Control>>)

### <a name="DELETE"></a>DELETE
Use DELETE for the following:
* To remove a resource entity identified by the Request URI

A successful response should be 200 (OK) if the response includes an entity describing the status, 202 (Accepted) if the action has not yet been enacted, or 204 (No Content) if the action has been enacted but the response does not include an ent
If a DELETE method is successfully applied the server should send back:
* *202 (Accepted)*:  if the action will likely succeed but has not yet been enacted
* *204 (No Content)*:  if the action has been enacted and no further information is to be supplied
* *200 (OK)*: if the action has been enacted and theity.

Similar to [PUT](#PUT) method DELETE can handle [Optimistic Concurrency Control](<<link_to_optimistic_concurrency_Control>>)


## Resource entity
### GET
The client send a request to the server. In the response it gets back the resource entity and in the header its ETag.
The [ETag](<<link_to_etag>>) header provides the current entity-tag for the selected resource entity

A conditional GET allows a client to ask a server if a resource has changed. If it has not changed, it can assume it's current knowledge is up to date and send back 304 Not Modified status. If it has changed, the server will send the resource back alon with 200 OK status to the client.
The client uses the [If-None-Match](<<link_to_If-None-Match>>) header with the ETag as the value. It's asking the server "Return the resource if it's ETag does not match mine." Because an ETag changes when a resource has changed, this effectively asks the server to return the resource only if it has changed.

If the client submits an unconditional DELETE request, return error code 403 (Forbidden). If the supplied conditions do not match, return error code 412 (Precondition Failed). In either case, explain the reason for the failure to the client in the body of the representation.

If the clients submits a conditional DELETE request and the supplied conditions match, delete the resource.

<<example>>

### POST
#### Creating a new resource as a factory:
When using POST to create new resources, the server decides the URI for the newly created resource.
If the resources has been created on the server as a result of successfully processing a POST request, the server should send a 201 Created response along with the following headers:
* [Location](<<link_to_location_header>>) that provides an identifier for the resource created.
* [ETag](<<link_to_etag>>) for the resource

The respose body should contain the resource's representation.

<<example>>

If the resource already exists then the response is 409 (Conflict).

### PUT
The method requests that the state of the target resource be created or replaced with the state defined by the representation enclosed in the request message payload.
