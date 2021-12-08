---
weight: 10
title: Overview
layout: redirect
---

The new Notifications 2.0 API allows applications or microservices to receive and process notifications generated by the use of the {{< product-c8y-iot >}} REST or MQTT API (device management and other platform API) in a reliable manner.
Currently, device management notifications equivalent to what is covered by real-time notifications are available, but delivered with at-least-once semantics over a new protocol.
In the future, further platform events will be included as 2.0 notifications.

This new Notifications 2.0 API supersedes the [Real-time notification API](https://{{< domain-c8y >}}/api/{{< c8y-current-version >}}/#tag/Real-time-notification-API), as it provides for stronger delivery semantics and ordering.
It is also intended to be simpler to use than the Bayeaux protocol used in the Real-time notification API.

In the {{< product-c8y-iot >}} 10.11 release, Notifications 2.0 needs to be enabled in the platform as it depends on the new Messaging Service that currently is optional.
See the *Messaging Service - Installation & operations guide* on how to do that.
Ingress via load balancers also needs to allow ingress for the new WebSocket endpoint that is enabled by default in {{< product-c8y-iot >}} 10.11, which might affect custom deployments that manage their own ingress.

To receive notifications over the 2.0 protocol, an application or microservice must subscribe to notifications, either for notifications about a particular managed object or in a wider context that is scoped to a tenant.

### Managed object context

For managed objects, an object's global identifier must be used to subscribe in the managed object context ("mo") in order to receive notifications.
There can be multiple subscriptions for the same subscribed object, with different filters or just to fan out to multiple interested consumer parties.

This subscribed object is known as the source object for the <kdb>notification/event</kdb> and it is referenced in the notifications delivered to subscribers.
Subscriptions are set up using the [subscription method](https://{{<domain-c8y>}}/api/{{< c8y-current-version >}}/#tag/Subscriptions) of the Notifications 2.0 API.
This API requires the calling user to be an authenticated {{< product-c8y-iot >}} user and to have the new ROLE_NOTIFICATION_2_ADMIN role.

When subscribing to notifications, a filter for notifications can be specified which determines the APIs (alarms, events, measurements, managed objects or any combination of these) to filter by.
It is also possible to filter by presence of a specific JSON fragment or "fragment type".
When matched, either the whole notification content is forwarded, or one or more fragments can be specified to be copied to the consumer.

In order to receive subscribed notifications, a consumer application or microservice must obtain an authorization token that provides proof that the holder is allowed to receive subscribed notifications.

This token is in the form of a string conforming to the JWT (JSON Web Token) standard that is obtained from the [token method](https://{{<domain-c8y>}}/api/{{< c8y-current-version >}}/#tag/Tokens) of the Notifications 2.0 API.
This API requires the calling user to be an authenticated {{< product-c8y-iot >}} user and to have the new ROLE_NOTIFICATION_2_ADMIN role.

See the [{{< openapi >}}](https://{{<domain-c8y>}}/api/{{< c8y-current-version >}}/#tag/Tokens) for both the Notification 2.0 Subscription and Token API.

Once subscribed, notifications are persisted and available to be consumed using a new WebSocket-based protocol.
This protocol implements a reliable delivery with at-least-once semantics.
The underlying Messaging Service will repeatedly attempt to deliver a notification until that notification is acknowledged as being received and processed by the consuming application or microservice.
Notification order is preserved from the point of view of a device sending in REST and MQTT API requests.
The protocol is text-based and described in detail in the next section.

### Tenant context

The tenant context ("tenant") is used for subscribing to and receiving notifications, in addition to the managed object context ("mo") mentioned above.
Creations of managed objects, which generate a new object identifier that can act as a source for notifications are reported in the tenant context.
This allows an application to discover a new managed object, which can then choose to subscribe to in the managed object context.
It is also possible to subscribe to all alarms that are generated in the tenant context.

See the [{{< openapi >}}](https://{{<domain-c8y>}}/api/{{< c8y-current-version >}}/#tag/Subscriptions) on how to subscribe to these notifications, additionally filtering the notification of interest.

For the protocol consumer, both managed object creations and alarms subscribed under the tenant context are reported in the same way.
There is no distinction between the two contexts for consumers, and notification ordering is maintained between the two contexts.

For Java developers, the API and the protocol have been wrapped up as an open Java API and a sample WebSocket client application.

There is a sample microservice available in the [cumulocity-examples repository](https://github.com/SoftwareAG/cumulocity-examples/tree/develop/hello-world-notification-microservice), so Java developers do not need to code to the following protocol specification directly.

### Token expiration

When creating a token, an expiration time must be given in minutes of validity from when the token was created.
This security feature limits the potential damage due to leaking of a token.
It requires tokens to be re-created or refreshed periodically.
This can be done by calling token create with the same parameters as originally. 
If the parameters used are not available they can be extracted from the token.
As the token string is a JWT (JSON Web Token), it can be decoded to extract the original information used to create the token by splitting it into 3 parts (on ".") and doing a base64 decode on the first substring.
This way, information like the subscription name can be extracted and the create token REST point can be called again, all on the client side.
The {{< product-c8y-iot >}} microservice Java SDK [TokenApi](https://github.com/SoftwareAG/cumulocity-clients-java/blob/develop/java-client/src/main/java/com/cumulocity/sdk/client/messaging/notifications/TokenApi.java) class contains a public refresh method which is implemented purely on the client side.

### Shared subscription

There can be multiple subscription on a managed object, each receiving filtered notifications as specified by their individual subscriptions.
In order to scale, shared subscriptions are required so that notifications are dispatched to one of a number of possible consumers that are part of the same logical subscriber or parallelised application.
This can be achieved by creating a token with a subscription and subscriber name for the scalable application but with an optional boolean "isShared" request parameter set to true in the token create request.

Notifications (messages) will be distributed among all connections to the Notification 2.0 web socket endpoint.
These all use a token for the same subscription and the subscriber and both the same subscription name, either by using the same shared token or using the same subscription name and subscriber when generating per instance tokens.
The subscription name defines the "topic" messages are published on, while the subscriber identifies the backend (north side) application that's usually used.
The application can consist of more than one instance running in parallel in the shared use case.

Notifications will be delivered in order with respect to the notification generating device.
The notifications will be delivered to the same application instance, except when a new instance or a failure changes the application topology.
Then it is necessary to re-distribute the devices to currently running instances.
In order to aid this assignment, the subscriber instance can specify a "consumer name" when connecting to the web socket endpoint.
The same token, or one that was generated in a similar way from subscription name and subscriber, is used as the token query string argument.
However, a different consumer name is passed as the "consumer" query string argument.
For example, instance 1 of the subscriber microservice could pass in `notification2/consumer?token=xyz&consumer=instance1` while instance 2 could use `notification2/consumer?token=XYZ&consumer=instance2`.
Currently, determining the instance ID of a microservice replica is not supported by the {{< product-c8y-iot >}} microservice API. An external application with named instances can be used instead.

### Volatile subscriptions

When subscribing, it is possible to pass in an optional boolean "volatile" query parameter set to true.
Note that there is no need to mark a subscription as shared - only the token is marked shared, but here it's both (token has isVolatile and isShared).
This changes the subscription to not persist notifications for the named subscription.
They are effectively only buffered in memory and can be discarded if they are not consumed quickly enough or on node failure.
Note that there can be both volatile and ordinary (non-volatile or persistent) subscriptions on a managed object with the same subscription name.
These count as separate subscriptions and can be consumed by a subscriber using a token with the corresponding `isVolatile` equals true or false.   

### Unsubscribing a subscriber

Once a subscription is made, notifications will be kept until consumed by all subscribers who have previously connected to the subscription.
For non-volatile subscriptions, this can result in notifications building up in storage if never consumed by the application.
These will be deleted if a tenant is deleted but otherwise can take up considerable space in permanent storage for high frequency notification sources.
It is therefore advisable to unsubscribe a subscriber that will never run again.
A separate REST endpoint is avaiable for this: <kbd>/notification2/unsubscibe</kbd>.
It has a non-optional query parameter "token".
The token is the same as you would use to connect to the web socket endpoint to consume notifications.
Note that there is no explicit subscribe subcriber using a token.
Instead this happens when you first connect a Web Socket with a token for the subscription name and subscriber.
To unsubscribe an application, however, is an explicit act using a similar or the original token token.
Unsubscribing should be infrequent, for example when deleting an application or during development when testing completes, 
as typically one wants messages to persist even when no consumer is running.
Only if no consumer will ever run again should an unsubscribe subscriber be necessary.

It is also possible to unsubscribe a subscriber on an open consumer web socket connection.
To do so, send "unsubscribe-subscriber" instead of a message acknowledgement identifier from your web socket client to the service.
The service will then unusbscribe the subscriber and close the connection.
It's not possible to check if the unsubscribe succeeded as the connection always closes so this way of unsubscribing is mostly for testing.