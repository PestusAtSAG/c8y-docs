---
weight: 60
title: Test the connection to the cloud
layout: redirect
opensource: true
---

We provide a way of testing the connection from your device to a cloud provider.
You can call this connection check function by:

```shell
sudo tedge connect <cloud> --test
```

It returns exit code 0 if the connection check is successful and 1 otherwise.

This test is already performed as part of the `tedge connect <cloud>` command.

### What does the test do?

The connection test sends a message to the cloud and waits for a response.
The subsequent sections explain the cloud-specific behaviour.

#### For Cumulocity IoT

The test publishes a [SmartREST 2.0 static template message for device creation `100`](/reference/smartrest-two/#device-creation-100) to the topic `c8y/s/us`.
If the device-twin is already created in your {{< product-c8y-iot >}} account,
the device is supposed to receive `41,100,Device already existing` on the error topic `c8y/s/e`.

So, the test subscribes to the `c8y/s/e` topic and if it receives the expected message on the topic, the test is marked successful.

The connection test sends a maximum of two SmartREST 2.0 `100` requests.
This is because the first `100` request can be considered a successful device creation request if the device-twin does not exist in {{< product-c8y-iot >}} yet.

#### For Azure IoT Hub

The test subscribes to the topic `az/twin/res/`.
Then, it publishes an empty string to the topic `az/twin/GET/?$rid=1`.

If the connection check receives a message containing `200` (status success), the test is marked successful.

The connection test sends the empty string only once.
