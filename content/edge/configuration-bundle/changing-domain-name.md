---
weight: 35
title: Changing the domain name
layout: redirect
---

After the installation, you can change the domain name of your Edge appliance using the GUI and REST APIs.

>**Important:** Before you change the domain name, see [Domain name validation for Edge license key generation](/edge/installation/#domain-name-validation-for-edge-license-key-generation).

### Changing the domain name using the GUI

1. Log in to the {{< management-tenant >}}.

	- Username: management/<*username*>
	- Password: password provided during the installation

2. Switch to the **Administration** application using the application switcher at the right of the top bar **<img class="Default" src="/images/icons/switcher-icon.png" alt="icon" style="display: inline; float: none">**.

3. Click **Edge** > **Domain/Certificate** in the navigator.

   Review the domain name and the SSL certificate.

4. Click **Edit** below **Domain name** to edit the domain name.

5. Provide the new domain name.

   If the existing license and SSL certificate files are compatible with the new domain name, you do not have to upload the license and certificate files.

   Click **Update**.

6. If the existing license is not compatible with the new domain name, provide the license file for the new domain name.

   If the existing license is compatible with the new domain name, you do not have to upload the license file.

7. If the existing certificate is not compatible with the new domain name, provide the SSL certificate and key files. If you do not have an SSL certificate, select **Generate self signed SSL certificate**.

   If the existing certificate is compatible with the new domain name, you do not have to upload the SSL certificate and key files.

8. Click **Save**.

### Changing the domain name using the REST APIs

To change the domain name, use the following endpoints:

- [GET /edge/configuration/domain](/edge/rest-api/#get-edgeconfigurationdomain)
- [POST /edge/configuration/domain](/edge/rest-api/#post-edgeconfigurationdomain)

### License and certificate compatibility

The example below describes a scenario where an existing license or certificate is compatible with the new domain name:

- If you have a license for the domain myown.iot.com, you can change the domain to myown.iot.com and any single level subdomain, for example sub.myown.iot.com

- If you have a certificate for the domain myown.iot.com, then you can only set the domain to myown.iot.com. If you have a wildcard certificate like *.myown.iot.com, then you must set the domain name to any single level subdomain of myown.iot.com, that is sub.myown.iot.com, but not myown.iot.com itself.

