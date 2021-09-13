# Notifications 2.0

The new Notification 2.0 API allows applications or microservices to receive and process notifications generated by use of the Cumulocity REST or MQTT API (device management and other platform API) in a reliable manner. Currently device management notifications equivalent to what is covered by Realtime Notifications is available, but delivered with at-least-once semantics over a new [reliable] protocol. In future, further platform events will be included as 2.0 notifications.

This new notification API supersedes Realtime Notifications, as it provides for reliable delivery semantics and ordering. It should hopefully also be simpler to use than the Bayeaux protocol used with Realtime notifications.

In 10.11, notifications 2.0 needs to be enabled in the platform as it depends on the new messaging service that currently is optional. Please see the operations manual on how to do that. Ingress via load balancers also needs to be enabled for a new web socket endpoint which might affect custom deployments that manage thier own ingress.

To receive notifications over the 2.0 protocol, an application or microservice must subscribe to notifications, either for notifications about particular managed objects or in a wider per-tenant context.

For managed objects an objects global identifier must be used to subscribe in the managed object ("mo") context. There can be multiple subscriptions for the same subscribed object, with different filters or just to fan out to multiple interested consumer parties. This subscribed object is known as the source object for the notification/event and is included in the notifications delivered to subscribers. Subscriptions are set up using the Notification2 subscription REST API. This API requires the calling user to be an authenticated Cumulocity user and to have the new ROLE_NOTIFICATION___ADMIN role.

In order to receive subscribed notifications a consumer application or microservice must obtain an authorization token that provides proof that the holder is allowed to receive subscribed notifications. This token is in the form of a string conforming to the JWT (Java Web Token) standard that is obtained from the new Notification 2.0 token API.  This API requires the calling user to be an authenticated Cumulocity user and to have the new ROLE_NOTIFICATION_2_ADMIN role.

Please see the OpenAPI documentation for both the Notification 2.0 subscription and token API.

Once subscribed, notifications are persisted and available to be consumed using a new Web Socket based protocol. This protocol implements a reliable delivery with at-least-once semantics. The underlying messaging service will repeatedly attempt to deliver a notification until that notification is acknowledged as being received and processed by the consuming application or microservice. Notification order is preserved from the point of view of a device sending in REST and MQTT API requests. We describe the new protocol below. 

The second context, other than managed object context, is the tenant context (tenant). Creations of managed objects, which generate a new object identifier that can act as a source for notifications are reported in the tenant context. This allows an application to disciover new managed object which it can then choose to subscribe to in the managed object context. It is also possible to subscribe to all alarms that are generated in the tenant context. Please see the API documentation of how to subscribe to these notifications. For the protocol, both managed object creations and alarms subscribed under the tenant context are reported in the same way. There is no distinction between the two contexts for consumers and notification ordering is maintained between the two contexts.

For Java developers, the API and the protocol have been wrapped up as an open Java API and a sample Web socket client application. There is a sample Java microservice so developers do not need to code to the following protocol specification directly.

## Notification 2.0 consumer protocol

The Cumulocity Notifications2 API uses a secure (web socket]) [https://en.wikipedia.org/wiki/WebSocket] to consume notifications generated by the Cumulocity API.

The new endpoint is accessible only using the external Cumulocity fully qualified domain name of your Cumulocity environment and the standard SSL port 443 using a secure web socket connection.

The [URL scheme](https://en.wikipedia.org/wiki/List_of_URI_schemes) therefore is "wss" and consumers use URLs starting with "wss://" followed by the fully qualified domain name of the Cumulocity environment, followed by a fixed URL path and a query string.

The URL path is "/notifications2/consumer/". There is only one required and one optional query string argument:

1. Required query string argument: token. The name of this query string parameter is "token" and the value must be a valid token in the form of a JWT token string as returned by a create token request to the Notification2 token REST API. Including the token as a query string parameter avoids having to set a HTTP header which can be an issue for some web socket clients or proxies.

2. Optional query string argument: consumer. The name of this optional query string parameter is "consumer" and the value is a non blank unique name for the consumer. While this parameter is optional it must be used for shared subscriptions (shared subscriptions are scaled out / parallel subscriptions, rather than exclusive subscriptions) if notifications for a particular device are to be delivered to the same named consumer instance in the scaled out set of consumers.

In summary, the urls used by consumers follow the following patterns:

'''
wss://your.cumulocity.environment.fullqualifieddomainname/notification2/consumer/?token=yourJwtTokenRequestedFromNotification2TokenService
'''
or
'''
wss://your.cumulocity.environment.fullqualifieddomainname/notification2/consumer/?token=yourJwtTokenRequestedFromNotification2TokenService&consumer=someUniqueNameForThisConsumer
'''

The web socket established with such an URL is a textual bi-directional connection using UTF-8 encoding. The web socket service sends a sequence of notifications to the consumer and the consumer sends back a short acknowledgement over this connection for each notification received. This acknowledgement is for a particular notification and the server will periodically re-sent a notification until it is acknowledged. This will cease once the server has received and processed an acknowledgement for a particular notification.

A notification (transmitted service to client) consists of a header and a body (similar to a HTTP request). The header is one or more (in practice at least 3) lines of text, separated by a '\n' (newline) character. The end of the header is demarcated by a double new line "\n\n".

The notification body follows the header. This also consists of UTF text - for example a JSON document. If the notification is binary data or includes binary data then it will be (base 64 encoded) [https://en.wikipedia.org/wiki/Base64].

The header lines for a notification are as follows (separated by \n' newlines):

1. Required message identifier for message acknowledgement. This opaque value is an encoded binary 64 bit value. It must be returned as part of the acknowledgement for the notification as described below.

2. The notification description on the second header line. This is a string describing what type of notification this is and its source.  Measurements ("measurements"), events ("events") and alarms ("alarms") are examples of notifications, as are inventory creates, updates and deletes ("managedObjects"). There is a direct correspondence with Realtime notifications which features similar notification descriptions. These are not enumerated here and are expected to increase in number in the future. For REST API notifications, they follow a 3 part format, separated by "/". More details on notification descriptions is given below.

3. An action string is the third header. Examples are CREATE, UPDATE and DELETE. More actions may be added in future. Together with the notification type they describe the logical event that generated the notification, such as a CREATE of an alarm or measurement.


Depending on the second header there may be further headers to follow but currently notifications only use the above three. In order to be future proof / forward compatible, we encourage consumer code to cope with more headers by parsing them out but ignoring them. Please see working code in the public microservices examples or elsewhere on how to do this relatively easily.

After the headers, there follows the notification body as UTF-8 text. Typically a JSON document is carried in this text.

### Notification description header

The second header line is the notification description string in the form of a '/' separated path. For API notifications descriptions have three parts: tenantId / type / sourceId 

1. tenantId - this the identifier for the tenant under which the notification was generated.

2. type - the platform type of notification generated. For example, event, measurement, alarm or managed object.

3. sourceId - the identifier of the "source" object that generated or is the subject of the notification. Source is a very loose term here, much as in "event sourcing" but generally indicates which managed object the notification is about.

Some examples are provided later below and backwards compatibility to realtime notifications is provided for, but please see other documentation, examples and experiment to get values for events that you are interested in.

### Notification acknowledgement

The first header line in each notification consists of an opaque, encoded binary identifier that must be returned as is in a reply to the notification2 service in a message acknowledgement. 

Please see working examples in the microservices examples on how to do this but it simply consists of sending the identifier back to the service in a self contained web socket client to server message text message. i.e. simply send back the first header without the training '\n'.


### Dealing with Notification duplication

Until a notification is acknowledged, the web socket service will attempt to re-deliver them. It is therefore desirable that a notification is acknowledged as soon as possible (so as to help avoid duplicates). However, this can be delayed until the notification has been successfully processed to make use of the "at-least-once" semantics that the Notification 2.0 service provides.

Simply process the message and only return the acknowledgement identifier when that processing completes successfully. If processing is longer than a minute or so, the service will resend the notification so the web socket client application must be prepared to deal with duplicates. Duplicates can also occur due to underlying network failures or perceived failures (slow transmissions) and subsequent failure masking re-transmission attempts.

A duplicate can be delivered out of order if several notifications are unacknowledged but only after follow-on notifications so should be relatively simple to deal with.

For example, in the logical sequence 1,2,3,4 notification 2 can be duplicated behind 3 or even 4 as in 1,2,3,2,4 or even several times, such as 1,2,3,4,2,3 ..2.

The notifications don't contain any unique identifier or timestamps to aid in de-duplication. Some events are easy to de-duplicate, such as inventory events where a unique source object is first CREATED and DELETED. But inventory UPDATES or logically sequenced events such as Alarms and Measurements require application specific sequencing. 

This can be achieved by including unique identifiers, sequence number or timestamps in the notification JSON (body) as required. Another alternative is to look up the Cumulocity database and see the current value, treating the notification as a signal only and ignoring the value carried.

As can be seen from the notification trace below, some notifications do carry timestamps. If the timestamps are not generated by the device client then they may only be loosely synchronized between notifications and therefore should be used carefully or not at all for de-duplication. 
### Parallel consumers

The protocol can deliver notifications to a scaled out subscriber. It is important to give each instance of the scaled out application a unique consumer identifier (passing on connect in the query string). Ideally this identifier should persist between re-starts of instances. With this, notifications from a particular device or soruce are delivered to the same instance of the scaled out application as long as that instance is running. This makes processing successive notifications about the same device (or source) simpler. 

## Trace of some notifications

'''
------------------------
header /tenant-a170/managedobjects/111
header CREATE
notification {"additionParents":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/additionParents"},"owner":"admin","childDevices":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/childDevices"},"childAssets":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/childAssets"},"creationTime":"2021-09-03T12:28:58.692Z","lastUpdated":"2021-09-03T12:28:58.692Z","childAdditions":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/childAdditions"},"name":"a switch","assetParents":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/assetParents"},"deviceParents":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/deviceParents"},"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111","id":"111","com_cumulocity_model_BinarySwitch":{"state":"ON"}}
------------------------
header /tenant-a170/measurements/111
header CREATE
notification {"self":"http://cumulocity.default.svc.cluster.local/measurement/measurements/117","time":"2021-09-03T12:29:01.664Z","id":"117","source":{"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111","id":"111"},"type":"c8y_TenantUsageStatisticsMeasurement","resourcesUsage":{"memory":0,"cpu":0,"usedBy":[]}}
------------------------
header /tenant-a170/events/111
header CREATE
notification {"creationTime":"2021-09-03T12:29:01.932Z","source":{"name":"a switch","self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111","id":"111"},"type":"com_cumulocity_model_DoorSensorEvent","self":"http://cumulocity.default.svc.cluster.local/event/events/118","time":"2021-09-03T12:29:01.664Z","id":"118","text":"Door sensor was triggered"}
------------------------
header /tenant-a170/eventsWithChildren/111
header CREATE
notification {"creationTime":"2021-09-03T12:29:01.932Z","source":{"name":"a switch","self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111","id":"111"},"type":"com_cumulocity_model_DoorSensorEvent","self":"http://cumulocity.default.svc.cluster.local/event/events/118","time":"2021-09-03T12:29:01.664Z","id":"118","text":"Door sensor was triggered"}
------------------------
header /tenant-a170/managedobjects/111
header UPDATE
notification {"additionParents":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/additionParents"},"owner":"admin","childDevices":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/childDevices"},"childAssets":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/childAssets"},"creationTime":"2021-09-03T12:28:58.692Z","lastUpdated":"2021-09-03T12:28:58.692Z","childAdditions":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/childAdditions"},"name":"a switch","assetParents":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/assetParents"},"deviceParents":{"references":[],"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111/deviceParents"},"self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111","id":"111","c8y_ActiveAlarmsStatus":{"major":1},"com_cumulocity_model_BinarySwitch":{"state":"ON"}}
------------------------
header /tenant-a170/alarms/111
header CREATE
notification {"severity":"MAJOR","creationTime":"2021-09-03T12:29:02.092Z","count":1,"history":{"auditRecords":[],"self":"http://cumulocity.default.svc.cluster.local/audit/auditRecords"},"source":{"name":"a switch","self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111","id":"111"},"type":"com_cumulocity_events_TamperEvent","self":"http://cumulocity.default.svc.cluster.local/alarm/alarms/119","time":"2021-09-03T12:29:01.664Z","id":"119","text":"Tamper sensor triggered","status":"ACTIVE","com_mycorp_MyProp":{"key1":"value1"}}
------------------------
header /tenant-a170/alarms/111
header CREATE
notification {"severity":"MAJOR","creationTime":"2021-09-03T12:29:02.092Z","count":1,"history":{"auditRecords":[],"self":"http://cumulocity.default.svc.cluster.local/audit/auditRecords"},"source":{"name":"a switch","self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111","id":"111"},"type":"com_cumulocity_events_TamperEvent","self":"http://cumulocity.default.svc.cluster.local/alarm/alarms/119","time":"2021-09-03T12:29:01.664Z","id":"119","text":"Tamper sensor triggered","status":"ACTIVE","com_mycorp_MyProp":{"key1":"value1"}}
------------------------
header /tenant-a170/alarmsWithChildren/111
header CREATE
notification {"severity":"MAJOR","creationTime":"2021-09-03T12:29:02.092Z","count":1,"history":{"auditRecords":[],"self":"http://cumulocity.default.svc.cluster.local/audit/auditRecords"},"source":{"name":"a switch","self":"http://cumulocity.default.svc.cluster.local/inventory/managedObjects/111","id":"111"},"type":"com_cumulocity_events_TamperEvent","self":"http://cumulocity.default.svc.cluster.local/alarm/alarms/119","time":"2021-09-03T12:29:01.664Z","id":"119","text":"Tamper sensor triggered","status":"ACTIVE","com_mycorp_MyProp":{"key1":"value1"}}
'''
