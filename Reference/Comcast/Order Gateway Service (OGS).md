
### Overview
- The intake and routing hub for advertising orders
- Upstream systems send orders to it (such as AutoScheduler)
	- #task what other systems send orders
- It validates, stores, then passes them to downstream systems
	- #task What do these systems do?
	- OneTIM
	- XG
	- EDS
- Architecture
	- Azure Functions App
	- Each stage is separate func with various triggers (HTTP, Service Bus queue, blob storage, Event Grid)
		- #task what are these triggers, what uses what, and why?

### How orders come in, two ways
- Small orders
	- `CreateOrder` HTTP POST carried entire order
	- stored
	- passes through `OrderGatewayValidateAdvertiser` and `OrderGatewaySaveOrder`
	- sent to OneTIM, XQ, and EDS queues
- Large orders
	- Caller POSTs metadata
		- OrderId
		- application ID
		- Advertiser ID
		- AgencyAccountNumber (optional)
	- Checks caller identity, validates body, then calls the manager
	- `SaveOrderTokenInformation`
		- writes audit row
		- creates the `OrderGroup`
		- no advertiser attached
	- returns  blob upload URL
	- caller uploads the order file
		- upload triggers SaveOrderDetail -> rest of pipeline

