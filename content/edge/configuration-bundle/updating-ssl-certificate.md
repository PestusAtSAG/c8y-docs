---
weight: 36
title: Updating the SSL certificate
layout: redirect
---

You must always have an SSL certificate for your domain name that is configured. If the validity of the certificate expires, you must upload a new certificate. You can upload the certificate using the GUI and REST APIs.

### Updating the SSL certificate using the GUI

1. Log in to the {{< management-tenant >}}.

	- Username: management/<*username*>
	- Password: password provided during the installation

2. Switch to the **Administration** application using the application switcher at the right of the top bar **<img class="Default" src="/images/icons/switcher-icon.png" alt="icon" style="display: inline; float: none">**.

3. Click **Edge** > **Domain/Certificate** in the navigator.

   Review the domain name and the SSL certificate.

4. Click **Edit** below **SSL Certificate** to upload the new SSL certificate file and the key file.

5. Provide the new SSL certificate file and the SSL certificate key file.

   If you do not have an SSL certificate, select **Generate self-signed certificate** to generate one.

5. Click **Save**.

### Updating the SSL certificate using the REST APIs

To upload the new SSL certificate and the key file, use the following endpoints:

- [GET /edge/configuration/certificate](/edge/rest-api/#get-edgeconfigurationcertificate)
- [POST /edge/configuration/certificate](/edge/rest-api/#post-edgeconfigurationcertificate)
