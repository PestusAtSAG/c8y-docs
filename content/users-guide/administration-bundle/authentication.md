---
weight: 71
title: Authentication
layout: redirect
helpcontent:
- label: authentication
  title: Authentication
  content: "Under **Login settings** you can specify your preferred login mode:


	* OAI-Secure - Recommended, since it provides high security, using authorization tokens to prove your identity (to the server).
	* Basic Auth - Should be chosen only for specific compatibility reasons, since it only provides basic security.
	* Single sign-on redirect - Can only be selected if SSO is configured. If selected, will remove Basic Auth and OAI-Secure login options.


	Under **TFA settings**, select the checkbox **Allow two-factor authentication** if you want to allow TFA in your tenant (only possible for administrators).


	Switch to the **Single sign-on** tab to configure single sign-on. For details, see *Administration > Changing settings > Configuring single-sign on* in the *User guide*."
---


Under **Authentication** in the **Settings** menu you can view and configure the login and security settings ([TFA](#tfa) or [single sign-on](#single-sign-on)).

![Authentication settings](/images/users-guide/Administration/admin-settings-authentication.png)

>**Requirements** You need the following permissions:
- To see the authentication menu entry: ADMIN rights for the permission category "Tenant management"

### Login settings

In the **Preferred login mode** field, you can select one of the following options:

* OAI-Secure - Recommended, since it provides high security, using authorization tokens to prove the identity of the user. Default login mode on creating new tenants. This mode is an enhancement of the previous OAuth Internal authentication (available prior to 10.13.0).
* Basic Auth - Should be chosen only for specific compatibility reasons, since it only provides basic security.
* Single sign-on redirect - Can be selected only if SSO is configured. If selected, will remove Basic Auth and OAI-Secure login options.

This login mode will be used by the platform's applications as the default method to authenticate users. Device authentication stays unchanged.

>**Important:** Each time you change the login mode you will be forced to log out. Other users will need to log out and log in again so that the change is applied.

In the field **Password validity limit**, you can limit the validity of user passwords by specifying the number of days after which users have to change their passwords. If you do not want to force your users to change passwords, use "0" for unlimited validity of passwords (default value).

>**Info:** The password validity limit is not imposed on users with a "devices" role. This prevents device passwords from expiring.

By default, users can use any password with eight characters or more. If you select **Enforce that all password are "strong" (green)**, users must provide strong passwords as described in [Getting Started > User options and settings > To change your password](/users-guide/getting-started/#to-change-your-password).

>**Info:** The password validity limit and the password strength may not be editable, if configured by the platform administrator.

<a name="basic-oauth-restrictions"></a>
### Basic Auth restrictions

Even if OAI-Secure authentication is configured for users, basic authentication remains available for devices and microservices using the platform. To provide a higher security level the basic authentication can be restricted.

Use the **Forbidden for web browsers** toggle to disallow the usage of basic authentication for web browsers. Moreover you can specify the following parameters:

* **Trusted user agents** - this list is empty by default. If some user agent is added, all the HTTP requests containing this entry in the `User-Agent` header and having a valid basic authentication date will be accepted.
* **Forbidden user agents** - this list is empty by default. If some user agent is added, all the HTTP requests containing this entry in the `User-Agent` header and using basic authentication will be rejected.

> **Info:** If the user agent is not found in the list of trusted or forbidden user agents then {{< product-c8y-iot >}} will try to verify if it is a web browser using an external library.

### OAI-Secure session configuration

OAI-Secure can work in two modes with significant differences:

#### Without a configuration related to the session (session configuration turned off)

When there is no configuration related to the session, OAI-Secure issues a JWT token with a certain lifetime. If the token expires then the user is forced to re-login because token refresh is not supported. This behavior is very inconvenient for the user if the token lifetime is short because the user is forced to re-login frequently.  

#### With the configuration of the session (session configuration turned on)

Using OAI-Secure with session configuration is more convenient and secure, and can be used to achieve a behavior which is similar to the authentication based on HTTP sessions.

The OAI-Secure token acts as a session identifier on the client site (web browser). Such a token identifier which is stored in the cookie can have a preconfigured short lifetime. Then, the {{< product-c8y-iot >}} platform is responsible for renewing the session identifier without a user interaction. It is sufficient that the user's action causes the web browser to send a request to {{< product-c8y-iot >}}. Then, {{< product-c8y-iot >}} can examine if the renewing of the session identifier should be executed and perform the operation if necessary. {{< product-c8y-iot >}} offers extensive configuration related to this behavior so that tenant administrators can adjust the configuration to their needs.

If the **Use session configuration** option is enabled, the following settings can be configured on tenant level by a tenant administrator:

<table>
<col width="200">
<col width="600">
<col width="200">
<thead>
<tr>
<th style="text-align:left">Field</th>
<th style="text-align:left">Description</th>
<th style="text-align:left">Default</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left">User agent validation required</td>
<td style="text-align:left">If turned on, the user agent sent in headers of consecutive requests in the scope of one session will be compared and a request with changed user agent will not be authorized.</td>
<td style="text-align:left">false</td>
</tr>
<tr>
<td style="text-align:left">Session absolute timeout</td>
<td style="text-align:left">Defines the maximum period of time that the user can use {{< product-c8y-iot >}} without having to re-authenticate.</td>
<td style="text-align:left">14 days</td>
</tr>
<tr>
<td style="text-align:left">Session renewal timeout</td>
<td style="text-align:left">Expected to be much shorter than the absolute timeout. Defines the time after which the {{< product-c8y-iot >}} tries to provide a new token (session identifier). The renewal may take place only when {{< product-c8y-iot >}} receives an HTTP request from a client with a non-expired token and the period of time between obtaining the token and the execution of the request is greater than the renewal timeout.</td>
<td style="text-align:left">1 day</td>
</tr>
<tr>
<td style="text-align:left">Maximum parallel sessions per user</td>
<td style="text-align:left">Defines the maximum number of sessions which can be started by each user (for example on different machines or browsers). When a user exceeds this limit, then the oldest session will be terminated and the user will be logged out on this particular device.</td>
<td style="text-align:left">5 sessions</td>
</tr>
<tr>
<td style="text-align:left">Token lifespan</td>
<td style="text-align:left">Defines the time for which a token is active. The user is able to  access the {{< product-c8y-iot >}} only with a valid token. This configuration option is always available, it does not depend on session configuration. See <a href="#token-settings" class="no-ajaxy">Token and cookie settings</a> below. </td>
<td style="text-align:left">2 days</td>
</tr>

</tbody>
</table>

>**Info:** The relation between the time parameters should be the following: renewal timeout < token timespan < absolute timeout. The recommended setting for the renewal timeout is approximately half of the token lifespan.

During the session token renewal the previous token is revoked and a new one is provided. The parameter `renewal token delay` defines the delay used to make this process smooth and not disturbing for the user. The old token is still valid for this period (1 minute by default). This way both tokens, old and new, are accepted by {{< product-c8y-iot >}}. This parameter is only configurable on platform level and cannot be modified by the tenant administrator.

<a name="token-settings"></a>
### Tokens and cookies

OAI-Secure is based on JWT stored in a browser cookie. The lifespan for both, token and cookie, is configurable by tenant options belonging to the category `oauth.internal`.

#### Token settings

The default token validity time is two weeks. This can be changed with tenant options:
 - category: `oauth.internal`;
 - key: `basic-token.lifespan.seconds`;

The minimum allowed value is 5 minutes.

#### Cookies settings

Cookies used to store a token in a browser have their own validity time that can be changed with tenant options:
- category: `oauth.internal`;
- key: `basic-user.cookie.lifespan.seconds`;

The default value is two weeks. It can also be set to any negative value so that the cookie will be deleted when the user closes the browser.

Refer to the [Tenant API](https://{{< domain-c8y >}}/api/{{< c8y-current-version >}}/#tag/Tenant-API) in the {{< openapi >}} for more details.

> **Info:** If external communication to the {{< management-tenant >}} has been blocked, then it is only possible to access the tenant in a secure way (for example via SSH tunnel). This means that you can just as well use basic authentication. Additionally, it is not possible to use single sign-on since the communication from the external authorization server is also blocked. Therefore, the authentication method is automatically set to "Basic authentication" if the {{< management-tenant >}} is configured to block external communication.


### Configuring single sign-on

{{< product-c8y-iot >}} provides single sign-on functionality, that allows a user to login with a single 3rd-party authorization server using the OAuth2 protocol, for example Azure Active Directory. Currently authorization code grant is supported only with access tokens in form of JWT.

> **Info:** This feature is built on top of cookies technology. To be able to use it, you must have cookies enabled in the settings of your browser.

This feature is enabled since {{< product-c8y-iot >}} version 10.4.6. For correct behavior any microservice needs to use the microservice SDK with version 10.4.6 or later.

Before switching to the single sign-on option it is mandatory that:

* The authorization server you use supports OAuth2 authorization code grant.
* The access token is issued as JWT and you know what goes into the token content.
* The JWT must consist of a unique user identifier, "iss" (issuer), "aud" (audience) and "exp" (expiration time) fields.
* The {{< product-c8y-iot >}} platform is in version 10.4.6 but preferably higher.
* All microservices are build with Microservice Java SDK 10.4.6 but preferably higher. For custom-built microservices, refer to [General aspects > Security](/microservice-sdk/concept/#security) in the *Microservice SDK guide*.
* For on premises installation the domain-based tenant resolution is configured properly.

>**Info:** In order to use the single sign-on feature for {{< enterprise-tenant >}}s, the enterprise domain must be set up as redirect URI in the basic configurations. If single sign-on providers have a list of allowed domains, the enterprise domain should be added to that list.


#### Configuration settings

To enable the feature, the administrator has to configure a connection with the authorization server. This is done in the Administration application.

Click the **Single sign-on** tab in the **Authentication** page.

At the top left, you can choose a template. The chosen option has an effect on the look of the panel. The default template is "Custom" which allows for a very detailed configuration with virtually any authorization server using OAuth2 authorization code grant. Other templates provide simplified views for well known and supported authorization servers. In the next steps there will first be a definition of how to use the "Custom" template followed by a view dedicated to Azure Active directory.

##### Custom template

![Request configuration](/images/users-guide/Administration/admin-sso-1.png)

As the OAuth protocol is based on the execution of HTTP requests and redirects, a generic request configuration is provided.

The first part of the **Single sign-on** page consists of the request configuration. Here you can configure the HTTP request address, request parameters, headers and body in case of token and refresh requests. The authorize method is executed as a GET, token and refresh method by POST requests.

>**Info:** Be aware that the body field of each request, after filling placeholders with values, is sent in the request 'as is'. This means it is not encoded by {{< product-c8y-iot >}}. Many authorization servers require values inside the body to be URL-encoded (x-form-urlencoded). This can be achieved by entering already encoded values in a body field.

Specifying a logout request is optional. It performs [front-channel single logout](https://openid.net/specs/openid-connect-frontchannel-1_0.html). If configured, the user is redirected to the defined authorization server logout URL after logging out from {{< product-c8y-iot >}}.

![OAuth configuration](/images/users-guide/Administration/admin-sso-logout-custom.png)

The **Basic** section of the **Single sign-on** page consists of the following configuration settings:

![OAuth configuration](/images/users-guide/Administration/admin-sso-2.png)

|Field|Description|
|:---|:---|
|Redirect URI|Redirect parameter. Can be used in request definitions as a ${redirectUri} placeholder
|Client ID|OAuth connection client ID. Can be used in request definitions as a ${clientId} placeholder
|Button name|Name displayed on the button on the **Login** page
|Issuer|OAuth token issuer
|Provider name|Name of the provider
|Visible on Login page|Indicates whether the login option is enabled or not.
|Audience|Expected aud parameter of JWT
|Group|(Deprecated in favor of dynamic access mapping since 9.20)The initial group assigned to the user on first login
|Applications|(Deprecated in favor of dynamic access mapping since 9.20)The initial applications assigned to the user on first login

Each time a user logs in, the content of the access token is verified and is a base for user access to the {{< product-c8y-iot >}} platform. The following section provides the mapping between JWT claims and access to the platform.

 ![OAuth configuration](/images/users-guide/Administration/admin-sso-7.png)

 In the example above, if a user tries to login a decoded JWT claims look like:

```json
{
...
"user": "john.wick",
...
}
```

The user will be granted access to the global role "business" and the default application "cockpit".

If no access mapping matches the user access token, the user will get an "access denied" message when trying to log in. This will also happen if there is no access mapping defined causing all users to be unable to log in using SSO.

New rules can be added by clicking **Add access mapping** at the bottom. An access mapping statement can consist of multiple checks like in the image below. You can add a rule to an existing statement by clicking **and**. Click the Minus button to remove a rule.

New roles are added to the user from every matching access mapping. If one access mapping statement assigns the role "admin" and a second one assigns the role "business" and both meet the defined conditions, then the user will be granted access to the global roles "business" and "admin"."

When using "=" as operator you may use wildcards in the **Value** field. The supported wildcard is asterisk (\*) and it matches zero or more characters. For example, if you enter "cur\*" this matches "cur", "curiosity", "cursor" and anything that starts with "cur". "f\*n" matches "fn", "fission", "falcon", and anything that begins with an "f" and ends with an "n".

In case the asterisk character should be matched literally it has to be escaped by adding a backslash (\\). For example, to match exactly the string "Lorem\*ipsum" the value must be "Lorem\\*ipsum".


 ![OAuth configuration](/images/users-guide/Administration/admin-sso-8.png)

In this case the following claim will match the condition:

 ```json
 {
 ...
 "user": {
    "type": "human"
 },
 "role": [
    "ADMIN"
 ],
 ...
 }
 ```

As you can see, there is an option to verify if a value exists in a list via the "in" operator. Values can also be embedded in other objects. In this case a dot in the key implies looking into an embedded object.

By default, dynamic access mapping assigns user roles, based on the access token, on every user login. This means, that it is not possible to change the user roles inside {{< product-c8y-iot >}} as these would be overridden on next user login. To change this behaviour, select the **Use dynamic access mapping only on user creation** checkbox at the bottom of the **Access mapping** section.

![OAuth configuration](/images/users-guide/Administration/admin-sso-dynamic-access-mapping.png)

When selected, dynamic access mapping will be used only when a new user logs in to fill in the initial roles. When a user already exists in {{< product-c8y-iot >}}, the roles will not be overridden nor updated. Selecting this option also enables admins to edit roles of SSO users in the user management. For details, refer to  [Administration > Managing permissions](/users-guide/administration/#assigning-global-roles) in the *User guide*.

When a user logs in with an access token, the username can be derived from a JWT claim. The claim name can be configured in the **User ID configuration** window.

 ![OAuth configuration](/images/users-guide/Administration/admin-sso-3.png)

 If the **Use constant value** checkbox is selected, a constant user ID is used for all users who log in to the {{< product-c8y-iot >}} platform via SSO. This means that all users who log in via SSO share the same user account in the {{< product-c8y-iot >}} platform. Usage of this option is not recommended.

Next, the **User data mappings** can be configured:

![OAuth configuration](/images/users-guide/Administration/admin-sso-user-data-mappings.png)

On user login, user data like first name, last name, email and phone number can also be derived from JWT claims. Each field represents the claim name that is used to retrieve the data from JWT. The user data mapping configuration is optional and as admin manager you can only use the required fields. If the configuration is empty or the claim name cannot be found in the JWT token then the values in the user data are set as empty.

Mapping for alias is not available because it is not used in the context of single sign-on login.

Each access token is signed by a signing certificate. Currently there are three options to configure the signing certificates.

1. By specifying the Azure AD certificate discovery address.

 ![OAuth configuration](/images/users-guide/Administration/admin-sso-4.png)

2. By specifying the ADFS manifest address (for ADFS 3.0).

 ![OAuth configuration](/images/users-guide/Administration/admin-sso-9.png)

3. By providing the public key of a certificate manually to {{< product-c8y-iot >}}. A certificate definition requires an algorithm information, public key value and validity period.

 ![OAuth configuration](/images/users-guide/Administration/admin-sso-5.png)

4. By specifying the JWKS (JSON Web Key Set) address.

 ![OAuth configuration](/images/users-guide/Administration/admin-sso-9.png)


 >**Info:** {{< product-c8y-iot >}} only supports certificates with RSA key, either as a ("n", "e") parameters pair or "x5c" certificate chain. Other key types (for example Elliptic-curves) are not supported.
##### Placeholders
Inside some fields you can use placeholders that are resolved by {{< product-c8y-iot >}} at runtime. Available placeholders are:

|Placeholder|Description|
|:---|:---|
|clientId|Value of the **Client ID** field
|redirectUri| Value of the **Redirect URI** field
|code|Code returned by the authorization server in response to authorization request
|refreshToken| Refresh token returned by the authorization server after token request

These placeholders can be used in authorization requests, token requests, refresh requests and logout request in the fields: URL, body, headers and request parameters

To use a placeholder in a field, put it inside two curly brackets preceded with a dollar sign:
![OAuth configuration](/images/users-guide/Administration/admin-sso-placeholder-standalone.png)

Placeholders can also be used as a part of text:
![OAuth configuration](/images/users-guide/Administration/admin-sso-placeholder-text.png)

Placeholders are not validated for correctness. Any not recognized or misspelled placeholder will be left in text unprocessed.

#### Integration with Azure AD

##### Azure AD configuration

The integration was successfully verified against Azure AD. The configuration steps are available in [https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-protocols-oauth-code](https://docs.microsoft.com/en-us/azure/active-directory/develop/v1-protocols-oauth-code).

When configuring your Azure AD, use your full domain address as redirect URI. For the purpose of this document we assume that it is "http://documentation.{{< domain-c8y >}}/tenant/oauth". The redirect URI must be set for a web application and not for a single-page application. There are no additional steps on Azure AD required.

##### Cumulocity IoT configuration

When the "Azure AD" template is selected the configuration panel will look similar to the following:

 ![OAuth configuration](/images/users-guide/Administration/admin-sso-aad-basic.png)
 ![OAuth configuration](/images/users-guide/Administration/admin-sso-aad-basic-1.png)

|Field|Description|
|:---|:---|
|Azure AD Address| Address of your Azure AD tenant
|Tenant| Azure AD tenant name
|Application ID| Application ID
|Redirect URI| Address of your {{< product-c8y-iot >}} tenant followed by /tenant/oauth
|Client secret| Azure AD client secret if applicable
|Button name| Button name
|Token issuer| Token issuer value in form of a HTTP address

Optionally single logout can be configured:

 ![OAuth configuration](/images/users-guide/Administration/admin-sso-logout-azure.png)

|Field|Description|
|:---|:---|
|Redirect after logout| Activates single logout by redirecting the user, after logout, to the authorization server logout endpoint
|Redirect URL| Address to redirect the user to after successful logout from the authorization server

The second part of the panel is the same as for the "Custom" template, where access mapping, user data mapping, user ID field selection and signature verification address are provided.

 ![OAuth configuration](/images/users-guide/Administration/admin-sso-aad-2.png)


##### Troubleshooting

It can be particularly helpful to inspect the content of the authorization token sent to the platform as some of its fields contain the information required for the correct configuration described above.

In Administration application, after clicking on **Accounts** > **Audit logs** you can filter by the category "Single sign-on" and look for entries "Json web token claims".

The contexts of the token will be presented in JSON format.

![Audit token content](/images/users-guide/Administration/admin-sso-audit-token.png)
