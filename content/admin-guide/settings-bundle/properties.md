---
weight: 20
title: Properties library
layout: redirect
---

Click **Properties library** in the **Settings** menu, to add custom properties to inventory objects, alarms, events and tenants.

![Properties library](/images/users-guide/Administration/admin-settings-properties-library.png)

With custom properties, you can extend the data model of {{< product-c8y-iot >}} built-in objects. You may create the following custom values:

- Custom inventory properties are used to extend the inventory data model. They can be used in the "Asset table" and "Asset properties" widgets.
- Custom tenant properties are available during tenant creation. The custom properties can be edited under **Subtenants** in the **Custom properties** tab of each tenant. Additionally, these properties can be viewed and exported in the **Usage statistics**.
- Custom alarm and event properties can be used as custom fields which can be added to your reports and will be available in the **Export** page in the Cockpit application.

>**Info:** Custom properties are visible to all authenticated users of the tenant, regardless of their inventory role permission.

<a name="add-property"></a>
#### To add a custom property

1. Select the tab for the desired property and click **Add property**.

	![Add new property](/images/users-guide/Administration/admin-settings-property-add.png)

1. In the resulting dialog box, provide a unique name as identifier and a label for the property and select its data type from the dropdown list.

1. Additionally, select validation rules for the new property:

<table>
<colgroup>
<col width="20%">
<col width="80%">
</colgroup>
<thead>
<tr>
<th style="text-align:left">Checkbox</th>
<th style="text-align:left">Description</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left">Required</td>
<td style="text-align:left">If selected, the property needs to be provided, i.e. during alarm creation. Not available if the property type is "Boolean".</td>
</tr>
<tr>
<td style="text-align:left">Default Value</td>
<td style="text-align:left">Provide a default value to be automatically filled in the custom property field. Only available for properties with type "String".</td>
</tr>
<tr>
<td style="text-align:left">Minimum</td>
<td style="text-align:left">Enter a minimum integer value.</td>
</tr>
<tr>
<td style="text-align:left">Maximum</td>
<td style="text-align:left">Enter a maximum integer value.</td>
</tr>
<tr>
<td style="text-align:left">Minimum length</td>
<td style="text-align:left">Enter the minimum length required for the string.</td>
</tr>
<tr>
<td style="text-align:left">Maximum length</td>
<td style="text-align:left">Enter the maximum length required for the string.</td>
</tr>
<tr>
<td style="text-align:left">Regular expression</td>
<td style="text-align:left">Add a regular expression which will be required in order to fill the custom property field.</td>
</tr>
</tbody>
</table>

4. Click **Save** to create the new property.

#### To edit a custom property

1. Click on the name of a property in the list to open it.
2. Do your edits. For details on the fields see [To add a custom property](#add-property).
3. Click **Save** to save your settings.


#### To remove a custom property

1. Click on the name of a property in the list to open it.
2. Click **Remove** to delete the property.
