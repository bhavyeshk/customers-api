# Customer API


## Use Cases

The API must be designed to support the following consumer use cases at a minimum:
* A consumer may periodically (every 5 minutes) consume the API to enable it (the consumer) to maintain a copy of the provider API's customers (the API represents the system of record)
* A mobile application used by customer service representatives that uses the API to retrieve and update the customers details
* Simple extension of the API to support future resources such as orders and products


### Commentary on the interaction of use case 1 with the API

 * Given the customer dataset in nearly static data, where the changes are not regular, there are couple of approaches that have been taken to improve performance. It is assumed that the service provider can provide either of the following validators the response header.
	- Entity Tags (Etags)
	- Last-Modified HTTP Header
 * Once the consumer receives a response along with Etag value in the headers, the consumer would store them in local store, and all subsequent requests would involve passing “If-None-Matched” header that uses the ETag Value. If data hasn’t been modified, a 304 should be returned, and cached copy can be used in response, else a 200 is returned with new dataset and new value for Etag.
 * Other approach taken, is that the consumer stores the “Last-Modified” date-time returned in response, along with the response body in local cache. The consume can then pass “If-Modified-Since” header in subsequent requests with the stored date-time. If the response data has not been changed on server, 304 (Not Modified) should be returned, along with empty body, consumer can then use the local cache to return the saved response. Whereas if the response data is modified, server should throw 200 OK response along with new response and new “Last-Modified” header.

### Commentary on interaction of use case 2 with the API
 * Mobile application will have limited resources such as processing speed and bandwith. Following considerations can be taken:
	- In order to reduce network latencies, the API can allow compressing the Payload when response is verbose. An “Accept-Encoding” header has been introduced for GET /customers, where consumer can pass values for the compression algorithm understood by the server.
 	- Since customer service representatives will also be able to update customer details, in events of larger requests, the API also makes use of “Content-Encoding” header as part of PUT /customer/{id}.
	- Applying the rest architecture constraint of HATEAOS would be particularly useful for this scenario, as consumer can use related hypermedia links provided in response by server to retrieve or discover available actions and its resources. In this manner larger set of related information/data can be retrieved in multiple calls without the need to send a larger dataset.
	- Pagination can also be implemented to restrict the number of pages/results per page.
* Given the application will be used by customer service representatives, a policy should be applied for authorization, for example client id/secret authorization to restrict access to certain users, or OAuth2 authentication can be applied. 
 

### Commentary on how the API could be extended for use case 3
 * The API Design makes used of several resource types and traits which provides a template to be used across multiple resources without the need to duplicate or repeat code. 
 * There are two resource types defined in this API spec, “collection” and “collection-item”. The “collection” resourceType aims to represent the entire collection and defines two methods, GET and POST. The “collection-item” represents instance of collection, defining GET, PUT, and DELETE methods, indicating these operations can be made on an item in a collection. 
 * Once the resourceTypes/traits are defined and published to exchange, it can easily allow any resources within this API or other APIs to make use of these types/traits.

### Commentary on any 'interesting' design decisions you made (and alternative options considered)
 * Defined the traits within Library fragment, providing a consolidated list of all common traits available.
 * Traits relating to request/response were defined to extract patterns from method definitions that are common across resources, and thus eliminate further redundancies.
 * Provided client side caching options, with assumptions the server would respond with “Etag”, “Last-Modified”, “Cache-Control”, “Expires” headers.
 * Customer Search/Pagination can be built by adding query parameters.

