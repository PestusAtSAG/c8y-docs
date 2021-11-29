---
weight: 12
title: Managing users
helpcontent:
  - label: managing-users
    title: Managing users
    content: "The Cumulocity IoT user management allows you to manage the users within your tenant. You can create new users, assign usernames and passwords, store user details, or configure the login and security options.  


    Moreover you can select the global roles for a user, see also *Administration > Managing permissions* in the *User guide*."
---

The user management feature allows you to manage the users within your tenant. With this functionality you may:

- Create users.
- Assign usernames and set passwords.
- Store user details.
- Choose basic login options.
- Enable additional login security by using Two-Factor Authentication (TFA).

> **Info:** The user needs to have a role with the user management permission ADMIN or CREATE to be able to do so.

If your tenant is configured for using single sign-on (SSO) in {{< sag-cloud >}}, new users should be created under **My Cloud**, accessible through the application switcher in the upper right corner, so that they are able to use the single sign-on feature.

For users created via an external authorization server, updating the following settings in {{< product-c8y-iot >}} will have no effect (will be reset on the next user re-login):

* user info (login alias, email, first name, last name, telephone)
* global roles → configurable via SSO access mapping
* application access → configurable via SSO access mapping

Moreover, password reset in {{< product-c8y-iot >}} is disabled for users created through an external authentication server.

> **Info:** Users which are using single sign-on cannot change the password of users which are managed by the platform.


### Viewing users

To view all users in your tenant, click **Users** in the **Account** menu in the navigator.

![Expanded view](/images/users-guide/Administration/admin-users-list.png)

A user list will be displayed, providing the following information for each user:

* The username that is used to access the tenant.
* The name and email of the user, if set.
* The global roles assigned to the user.
* The [strength](/users-guide/getting-started/#login) of the password set for the user.

To filter the list, you can use the search field at the left of the top menu bar. For details on the search functionality, refer to [Searching](/users-guide/getting-started/#searching) in the *Introduction*.

Moreover you can filter by global roles. Select the desired roles from the dropdown list and click **Apply** to limit the users shown in the list to users with the selected roles.

Initially, the **User** page only shows the top-level users. To see all users in your account at once, click **Expand all** at the right of the top bar. This will expand all top-level users, showing their sub-users. Click **Collapse all** to just show the top-level users again. For details on user hierarchies, refer to [Managing user hierarchies](/users-guide/enterprise-tenant/#user-hierarchies).

<a name="creating-users"></a>
### To add a user

1. Click **Add user** at the right of the top menu bar.  

  >**Info:** If single sign-on is enabled for your tenant, a message will show up which reminds you that you are about to create a local user which will not be able to login via single sign-on. Create new users in **My Cloud** instead, accessible through the application switcher in the upper right corner, to enable them using the single sign-on feature.

2. At the left of the **New user** window, provide the following information to identify the user:

	<table>
	<thead>
	<colgroup>
	<col style="width: 20%;">
	<col style="width: 80%;">
	</colgroup>
	<tr>
	<th align="left">Field</th>
	<th align="left">Description</th>
	</tr>
	</thead>
	<tbody>
	<tr>
	<td align="left">Username</td>
	<td align="left">Serves as a user ID to identify the user at the system. Note that the username cannot be changed once the user has been created. This field is mandatory.</td>
	</tr>
	<tr>
	<td align="left">Login alias</td>
	<td align="left">In addition to the username, an optional alias can be provided to be used to log on. In contrast to username, this alias may be changed if required. User alias is not supported for devices.</td>
	</tr>
	<tr>
	<td align="left">Active</td>
	<td align="left">Enable/disable the user account here. If the user account is disabled the user cannot login.</td>
	</tr>
	<tr>
	<td align="left">E-mail</td>
	<td align="left">A valid email address. This is required to enable the user to reset the password. This field is mandatory.</td>
	</tr>
	<tr>
	<td align="left">First name</td>
	<td align="left">First name of the user. When the user is logged in, this name appears at the right of the top bar on the <strong>User</strong> button.</td>
	</tr>
	<tr>
	<td align="left">Last name</td>
	<td align="left">Last name of the user.</td>
	</tr>
	<tr>
	<td align="left">Telephone</td>
	<td align="left">A valid phone number. The phone number is required if the user is configured to use two-factor authentication.</td>
	</tr>
	<tr>
	<td align="left">Owner</td>
	<td align="left">Another user that manages ("owns") the new user. Select a user from the dropdown list and click <strong>Done</strong> to confirm. Refer to <a href="../../users-guide/enterprise-tenant#user-hierarchies">Managing user hierarchies</a> for details on user hierarchies.</td>
	</tr>
	<tr>
	<td align="left">Delegated by</td>
	<td align="left">Can be activated to delegate user hierarchies and permissions to the user. Refer to <a href="../../users-guide/enterprise-tenant#user-hierarchies">Managing user hierarchies</a> for details on delegation.</td>
	</tr>
	</tbody>
	</table>

3. Select the login options for the user.
	* 	**Two-factor authentication (SMS)**: If selected, the user will receive a verification code via SMS which is required to complete the authentication. The SMS will be sent to the phone number configured above. For details refer to [Two-factor authentication](/users-guide/administration/#tfa).
	* **User must reset password on next login**: If selected, you need to provide a password which the user must reset on the next login. Enter a password and confirm it. While entering the password, the strength of the password will be checked. See [To log into the {{< product-c8y-iot >}} platform](/users-guide/getting-started/#login) for further information on password strength.  
	* **Send password reset link as email**: If selected, the user will receive an email message with a link to set a password. The email will be sent to the email address configured above.

4. On the right of the page, select the global roles for the user. Details on global roles are described in [Managing Permissions](/users-guide/administration#managing-permissions).
5. Click **Save** to save your settings.

The new user will be added to the user list.

> **Info:** By default, manually created users always have the "Own&#95;User&#95;Management" permissions set to active.

### To edit a user

1. Click the menu icon at the right of the respective row and then click **Edit**. All fields except **Username** and **Send password reset link as email** can be changed. For details on the fields, see [To add a user](#creating-users).
2. Click **Change password** to change the password.
3. Click **Save** to apply your settings.

> **Info:** You need a role with user management permission to perform this option.

### To copy inventory roles

1. Click the menu icon at the right of the respective row and then click **Copy inventory roles from another user**.
2. In the resulting dialog box, select if you want to merge the roles to be copied with the existing user roles (the default) or if you want to replace the existing user roles.
3. Select the user from which you want to copy roles from the dropdown list.
4. Click **Copy**.

The inventory roles will be copied from the selected user.

> **Info:** You need a role with user management permission to perform this option.

### To delegate/undelegate user hierarchies

Click the menu icon at the right of the respective row and then click **Delegate** to delegate your user hierarchies and permissions to a user.

Click **Undelegate** to remove a delegation.

Refer to [Managing User Hierarchies](/users-guide/enterprise-tenant#user-hierarchies) for details on delegation.

> **Info:** You need a role with user management permission to perform this option.

### To disable/enable a user

Click the menu icon at the right of the respective row and then click **Disable** to disable an active user, or click **Enable** to enable a user that has been disabled.

> **Info:** You need a role with user management permission to perform this option.

### To delete a user

Click the menu icon at the right of the respective row and then click **Delete**.

> **Info:** You need a role with user management permission to perform this option.
