---
weight: 20
title: Viewing audit logs
helpcontent: 
  - label: audit-logs
    title: Viewing audit logs
    content: "Audit logs show the operations that users have carried out.
      
      In order to easily search through logs, specify filter criteria in the top bar for type, date range or user and apply them."
---

Audit logs show the operations that users have carried out.

To view the audit log list, click **Audit logs** in the **Account** menu. For each log entry, the following information is provided:

<table>
<colgroup>
<col style="width: 15%;">
<col style="width: 85%;">
</colgroup>
<thead>
<tr>
<th align="left">Column</th>
<th align="left">Description</th>
</tr>
</thead>

<tbody>
<tr>
<td align="left">Server time</td>
<td align="left">Server time when the operation was processed.</td>
</tr>

<tr>
<td align="left">Change</td>
<td align="left">Type of operation, e.g. "Alarm created", "Smart rule deleted". Below it, the user who processed it is displayed.</td>
</tr>

<tr>
<td align="left">Description</td>
<td align="left">Provides further information depending on the operation, e.g. the device name, alarm text, operation status.</td>
</tr>

<tr>
<td align="left">Device time</td>
<td align="left">Device time when the operation was processed. This can differ from the server time.</td>
</tr>
</tbody>
</table>

Only the last 100 logs are visible. Click **Load more** at the bottom of the list to view more log entries.

![Audit logs](/images/users-guide/Administration/admin-audit-logs.png)

>**Info:** The audit log list is not automatically refreshed after a realtime update for operations. Click **Reload** at the right of the top menu bar to update the list to the latest operations.

### Filtering logs

In order to easily search through logs, you may filter logs for

 - the type, i.e. alarm, operation, Smart Rule,
 - a date range providing a "From" and/or a "To" date,
 - the user.

To apply filters, click the filter icon next to the filter fields. To discard filters, click the delete icon (only visible if filters are set).
