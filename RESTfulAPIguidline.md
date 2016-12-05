# Project specific rules
## PATCH
Be aware that the PATCH method sometimes blocked in some firewall connfiguration, therefore the resource should always support [PUT](#PUT) method beside PATCH.
The PATCH method supports [JsonPatch](http://jsonpatch.com) format.

<<example>>

## PUT
The request has to send whether the If-Match header or the If-None-Match heder. Otherwise sthe server responses with 412 (Precondition Failed).

# Appendix
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
**POST** | no | no | Collection
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

## Resource entity
### GET
The client send a request to the server. In the response it gets back the resource entity and in the header its ETag.
The [ETag](<<link_to_etag>>) header provides the current entity-tag for the selected resource entity

A conditional GET allows a client to ask a server if a resource has changed. If it has not changed, it can assume it's current knowledge is up to date and send back 304 Not Modified status. If it has changed, the server will send the resource back alon with 200 OK status to the client.
The client uses the [If-None-Match](<<link_to_If-None-Match>>) header with the ETag as the value. It's asking the server "Return the resource if it's ETag does not match mine." Because an ETag changes when a resource has changed, this effectively asks the server to return the resource only if it has changed.

<<example>>

### <a name="PUT"></a>PUT
Use PUT for the following:
* To create new resources only when client is able to assign URIs for the resources. To let clients assign URIs, the server needs to explain to clients how URIs on the server are organized, what kind of URIs are valid, and what kind are not
* To override an existing resource with a modified version of the one residing on the server

#### Create new resource
The response status can be:
* *201 (Created)*: if a new resource is created
* *404 (Not Found)*: if the server does not support resource creation

#### Override an existing resource (Optimistic Concurrency Control)
We must handle in some cases concurrent updates on a resource, but all resources are suited for concurrency control.
Once the ETag is received, it allows the client to perform the update. However, the changes will only apply if the ETag is still valid.
If the resource exists, take the following steps:
* First client must obtain an ETag for the update operation
* The client has to send the new representation of the resource along with the If-Match condicional header that contains the ETag
The response should be:
* *412 (Precondition Failed)*: if the supplied If-Match header do not match the actual ETag value of the resource on the server
* *200 (OK) or 204 (No Content)*: if the supplied conditions match the server updates the resource, and sends back  with the updated value of ETag header. The response also includes a Content-Location header.

The method requests that the state of the target resource be created or replaced with the state defined by the representation enclosed in the request message payload.

### <a name="PATCH"></a>PATCH
Use PATCH for the following:
* To support partial updates in contrast with PUT that is defined for the complete update or replacement of a resource

The client can check whether te PATCH is supported via the Allow header of the OPTIONS response or the resource can include an Accept-Patch header with the supported media types for the PATCH method.

Similar to [PUT](#PUT) method PATCH can handle [Optimistic Concurrency Control](<<link_to_optimistic_concurrency_Control>>)

### <a name="DELETE"></a>DELETE
Use DELETE for the following:
* To remove a resource entity identified by the Request URI

Similar to [PUT](#PUT) method DELETE can handle [Optimistic Concurrency Control](<<link_to_optimistic_concurrency_Control>>)
The response should be:
* *200 (OK)*: if the response includes an entity describing the status
* *202 (Accepted)*: if the action will likely succeed but has not yet been enacted
* *204 (No Content)*: if the action has been enacted and no further information is to be supplied
* *403 (Forbidden)*: if the client submits an unconditional DELETE request
* *412 (Precondition Failed)*: if the supplied conditions do not match

If the request fails, the body should contain the explanation of the failure.

## Resource collection
### <a name="POST"></a>POST
Use POST for the following:
* To create a new resource
* To perform any unsafe or nonidempotent operation via calling a SOA endpoint
* To run queries with large inputs
Using POST to create new resources, the server decides the URI for the newly created resource.
If the resources has been created on the server as a result of successfully processing a POST request, the server should send a 201 Created response along with the following headers:
* [Location](<<link_to_location_header>>) that provides an identifier for the resource created.
* [ETag](<<link_to_etag>>) for the resource

The respose body should contain the resource's representation.

The response should be the following:
* *201 (Created)*: if a new resource is created
* *409 (Conflict)*: if the resource already exists

<<example>>
