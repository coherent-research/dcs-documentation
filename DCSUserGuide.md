# Introduction

The Coherent Research Data Collection Server (DCS) consists of four logical components: the actual server itself which runs as a Windows service (DCService), SQL Server with the DCS schema defined (DCS DB), a Web Application to configure the server and view the collected data (DCSWebApp) and a Web Services interface (DCSWebServices) to allow IDCs to communicate with it. This document discusses how to use the DCSWebApp. For more information on installing the components refer to the
DCS Installation Guide on the Coherent Research Support website
<a href="https://www.coherent-research.co.uk/support/extras/DcsInstallationGuide/" target="_blank">(go to DCS Installation Guide at Coherent Support)</a>.

# Concepts

A _Meter_ in DCS is an abstraction of an
individual physical entity from
which data can be collected, e.g. a gas meter,
electricity meter or
data logger. In DCS a _Meter_ contains one or more _Registers_,
each
of which measures a different quantity and
has its own _Unit_
(e.g. W, A, m<sup>3</sup>). Before a _Meter_
can be defined in DCS, an appropriate _Meter Type_
must be defined. A _Meter Type_ defines a particular configuration of meter,
i.e. the registers that the meter contains, scaling factors and the
plugin
used to fetch the data. Once a _Meter Type_
is defined, any number of _Meters_ of that type
can be created.

DCS comes with a number of _System Defined Meter Types_ for popular meters. The user is also able to define their own _Custom Meter Types_. When creating _Meters_ both _System Defined_ and _Custom Meter Types_ behave identically.

_Meters_ may be logically partitioned into a hierarchy
of _Meter Groups_ (see
[Managing Meter Groups](#managing-meters-and-virtual-meters-managing-meter-groups)). _IDCs_ may be logically partitioned into a hierarchy
of _IDC Groups_ (see [Managing IDC Groups](#managing-idcs-managing-idc-groups)).

A _Calibration Reading_ is usually derived by manually reading the display of the physical meter and recording the value and time.
When this calibration reading is entered into DCS it is used to calculate an offset between the value entered and the reading in the database at the corresponding time (linearly interpolated between half hour values).
The offset is then applied to all readings when displayed or downloaded.

For meters that don't support totals (e.g. some intelligent meters that only provide period data) a Calibration Reading may be used, along with the period data, to calculate register totals.
By its nature this implies that totals at period boundaries are estimated and there may be minor discrepancies between totals and period values.
See [Calibration Readings](#calibration-readings) for more details.

_Manual Readings_ are manually entered meter readings associated with a register that can be used/viewed instead of normal metered data for meters that do not support automatic data collection, or used to fill in gaps in metered data.

_Register Resets_ can be defined for registers when a meter counter has rolled over or been reset for some reason causing a discontinuity in the counter value.
The Register Reset enables DCS to handle the discontinuity correctly when charting, performing interpolation etc.
See [Register Resets](#register-resets) for more details.

# Security

In order to access DCS a user
must be issued with a user name and password which will be emailed to
the user by DCS when they have been created.

When accessing the
system the user will be asked to enter their username and password. At
this screen the user can enter their credentials, change their password
if desired and reset their password if they have forgotten it. When a
password is reset a new one will be sent automatically to the email
address registered for the user.

Users may also use credentials from another authenticatin provider via a Single-Sign-On system if DCS were configured to do so. This could be instead of a user name and password from DCS, or even the use of both methods interchangeably.

# Managing Meters and Virtual Meters

## Overview

It is possible to partition meters logically into a hierarchy of Groups. The hierarchy can be considered as a tree and meters are only assigned to the leaves of the tree. Meters can be moved from one group to another without effect.

There is no limit in DCS as to the number of groups or the nesting within groups and no prescribed way of deciding how to partition meters for a particular installation. Assigning a meter to a group is done when the Meter is added.

All operations for meters and virtual meters are accessed via the **Meters** page which can be selected from the main navigation bar.

The **Meters** page is generally divided into 2 parts: a group panel on the left and the main part on the right. The group panel contains the meter groups in a tree structure. All group actions are performed in the group panel. The group panel can be hidden by clicking on the **Hide groups** button to provide the user with more room for the main part of the page.

Alternatively, the user can click on the **Show flat list** button to display all meters and virtual meters in a flat list and use the filter bar to narrow the list down to the required meters and virtual meters. Clicking the **Show groups** button will display the group panel.

## Managing Meter Groups

### Adding a Group

- Go to the **Meters** page in the main navigation bar and ensure the group panel is visible.
- To create a new group use the **Add new group** button at the top of the group panel on the left of the screen. It is possible to create a top level group or a subgroup of the current group if one is selected.
- When a new group is created a group settings form will be shown.
- Enter the fields according to the table below.
- Click the **Save** button to confirm the creation of the group.

| Field     | Description                                                                                                                                                                                                                       |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name      | A human readable name for the Group. This can be any name whatsoever depending on how the user wants to partition the meters (i.e. by use, type, location etc).                                                                   |
| URL       | An optional absolute URL can be added if a link to extra information is required. This link will be presented when the user is viewing a group and when clicked will open in a new browser tab. This can be used for any purpose. |
| Link text | The text that will be displayed of the URL is used.                                                                                                                                                                               |

### Modifying a Group

- Each group in the group panel has a action menu to the right of the group name that contains an option to **Group settings**.
- Modify any details entered previously and click the **Save** button.

### Moving a Group

- Groups may dragged from one group to another to move them in the group panel. It is also possible to drag a group from underneath another group and drop it in the area immediately above the first group or below the last group to move the group to the top level.

### Deleting a Group

- A groups can be deleted by using the action menu to the right of the group name that contains an option to **Delete group**. This will only be available if the group is empty.

## Managing Meters

### Viewing Meters

- Select **Meters** in the main navigation bar and then select a group in the group panel to display a table of all meters (and virtual meter) in the group.
- To select an individual Meter click on the row in the Meter table.
- Alternatively click on the **Show flat list** button to display all meters and virtual meters in a flat list and use the filter bar to narrow the list down to the required meters and virtual meters.

### Adding a Meter

- Click the **Add new meter** button at the top right hand of the meter table which will display an empty meter settings form.
- Enter the fields according to the table below.
- Press the **Save** button to confirm the creation of the meter.

> When a meter is created only the registers designated as _Default_ in the _Meter Type_ will be created automatically. To add extra registers see [Add register](#managing-meters-and-virtual-meters-managing-meters-adding-a-register).

| Field                       | Description                                                                                                                                                                                                                                          |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name                        | A human readable name for the Meter. This can be any name whatsoever and is only to aid the user in identifying a meter and is not used by DCS to connect to the meter.                                                                              |
| Description                 | Used for user information only.                                                                                                                                                                                                                      |
| Meter Type                  | The type of meter to be created. Each meter must correspond to an existing meter type which defines the plugin used to interrogate the meter, register definitions etc.                                                                              |
| Remote address              | The form of the Remote Address depends on the Connection Method. See below for details about formatting the remote address for each Connection Method.                                                                                               |
| Device ID                   | This is usually only required for meters in a multi-drop configuration and corresponds to the substation/outstation address used to identify a meter within the group.                                                                               |
| Connection method           | The method DCS uses to communicate with the meter. The Connection Methods supported by DCS are **Modem,** **TCP/IP** and **IDC**.                                                                                                                    |
| Collection Schedule         | The schedule used when connecting to a meter for data collection. This is only valid if the Connection method is Modem or TCP/IP. NB: If the Connection method is TCP/IP and no Collection Schedule is specified the meter will be read each period. |
| Baud                        | Serial baud rate (if supported by Connection method)                                                                                                                                                                                                 |
| Data bits                   | Serial data word size in bits (if supported by Connection method)                                                                                                                                                                                    |
| Parity                      | Serial parity (if supported by Connection method)                                                                                                                                                                                                    |
| Stop bits                   | Serial stop bits (if supported by Connection method)                                                                                                                                                                                                 |
| Initialisation string       | Modem initialisation string. This is only valid for modems.                                                                                                                                                                                          |
| End of message              | Timeout value (in ms) defines the time after the reception of a character that is interpreted as an end of message by DCS (if supported by Connection method)                                                                                        |
| Ignore parity errors        | Ignore any parity errors in incoming data. This should only be used in exceptional cases when the parity bit is not received correctly (if supported by Connection method)                                                                           |
| Serial Number               | The meter's serial number.                                                                                                                                                                                                                           |
| MPN (Metering Point Number) | Used for user information only.                                                                                                                                                                                                                      |
| Password                    | The meter's password.                                                                                                                                                                                                                                |
| Service                     | Used for user information only.                                                                                                                                                                                                                      |
| Disabled                    | Can be used to indicate that a meter should be ignored for the purpose of data collection.                                                                                                                                                           |
| Adjust time                 | Specifies whether DCS will attempt to adjust the meter's date/time to synchronise it with the server running DCS.                                                                                                                                    |
| URL                         | Optional URL to allow the user to browse to extra information associated with this meter external to DCS (e.g. its location on a map).                                                                                                               |
| Link Text                   | Text displayed for the link.                                                                                                                                                                                                                         |
| Notes                       | Used for user information only.                                                                                                                                                                                                                      |

Extra fields may be present that are specific to the plugin used by the Meter Type selected. For more information on these fields see [Metered Data Plugins](#metering-plugins-metered-data-plugins

**Remote Address:**

For a **Modem** the Remote Address takes the form a telephone number.

For a **TCP/IP** connection the Remote Address takes the form of _IPAddress_:_PortNo_, e.g. _127.0.0.1:4000._

For an **IDC** the Remote Address takes forms of the MAC address (e.g. 00:17:BF:00:00:01) followed by an indicator as follows:

- Pulse (IDC v3) Inputs: 00:17:BF:00:00:01-Pn where n is the number of the pulse input (0-31).
- Pulse (IDC v4 or greater) Inputs: 00:17:BF:00:00:01-Pm.n where m is the Modbus address of the pulse counter, and where n is the number of the pulse input (0-7).
- Radio Inputs: 00:17:BF:00:00:01-Rn where n is the serial number of the radio transmitter (WITHOUT leading zeros).
- RS-485 connected meters: 00:17:BF:00:00:01-COMA
- RS-232 connected meters: 00:17:BF:00:00:01-COMB
- External Modbus connected serial (non-Modbus) meters: 00:17:BF:00:00:01-MOD (in this case the external Modbus is used as an RS-485 port)
- External Modbus connected Modbus devices: 00:17:BF:00:00:01-Mn where n is the Modbus device address (1-247)

### Modifying a Meter

- Select the meter from the meters table.
- Click the **Meter settings** button next to the meter name which will display they meter settings form.
- Modify any details entered previously and click the **Save** button.

### Cloning a Meter

- Select a group in the group panel to display a table of all meters in the group.
- A meter can be cloned by using the action menu on the right of the meter row and clicking **Clone meter**. This will create a new meter settings form filled out with the same values as the previous meter.
- Modify any fields required and then click the **Save** button to create the new meter.

### Moving a Meter

- Select a group in the group panel to display a table of all meters in the group.
- The Meter may now be dragged and dropped onto a different group in the group panel.

### Deleting a Meter

- Select a group in the group panel to display a table of all meters in the group.
- Delete a meter by using the action menu on the right of the meter row and clicking **Delete meter**.

> Note that when a meter is deleted all readings for that meter will be deleted permanently.

### Adding a Register

- Select the meter from the meters table.
- Click the **Registers** tab to view the registers currently defined for the meter.
- To add register click the **Add new register** button. This will display a list of all registers that are defined in the Meter Type but do not exist for the current meter. Select one or more registers and click **Save**.

Note that if all registers that are defined for the meter type have been created for the current meter the **Add new register** button will not be available.

### Deleting a Register

- Select the meter from the meters table.
- Registers can be deleted by using the action menu to the right of the register table.

> Note that all readings associated with the register will also be deleted.

### Modifying a Register

- Select the meter from the meters table.
- Click the **Registers** tab to view the registers currently defined for the meter.
- Click the register row to select it. This will display the register settings form.
- Change the fields according to the table below.
- Press the **Save** button to confirm the changed.

| Field        | Description                                                                                                                              |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Scale factor | An scale factor that will override the default scale factor from the Meter Type. This can be left blank to use the default scale factor. |
| Tariff       | An optional tariff associated with the register to be used when calculating Billing Data.                                                |

## Managing Virtual Meters

### Viewing Virtual Meters

- Select **Meters** in the main navigation bar and then select a group in the group panel to display a table of all virtual meters (and meters) in the group.
- To select an individual Virtual Meter click on the row in the Virtual Meter table.
- Alternatively click on the **Show flat list** button to display all meters and virtual meters in a flat list and use the filter bar to narrow the list down to the required meters and virtual meters.

### Adding a Virtual Meter

- Click the **Add new virtual meter** button at the top right hand of the virtual meter table which will display an empty virtual meter settings form.
- Enter the fields according to the table below.
- Press the **Save** button to confirm the creation of the virtual meter.
                      
Field | Description
------|------------            
Name | A human readable name for the Virtual Meter. It should be something concise and obvious, e.g. Main Building Electricity.
Description | Any text to describe the Virtual Meter further.
Expression | The mathematical function to define the time series data. This is expressed as a mathematical function of meter registers. Each meter register will be specified as a register variable which can be any alphanumeric string without spaces. See below for more details.
Unit | The unit for the calculated data.
Instantaneous | Specifies whether the virtual meter contains instantaneous quantity (e.g. W) or a cumulative quantity (e.g. Wh).
Decimal places | The number of significant decimal places for the calculated data. The value can be any integer &gt;= 0.
Notes | Any extra information that may be useful for the user.
URL | Optional URL to allow the user to browse to extra information associated with this meter external to DCS (e.g. its location on a map).
Link Text | Text displayed for the link.
Tariff | An optional tariff associated with the Virtual Meter to be used when calculating Billing Data.

**Expressions**  
Currently the expression can consist of the following:
numerical constants, +, -, ., /, \*, (, ) and register variables and the ternary operator (see below). E.g.

```
(ELECT1+ELECT2)*1.5 
```

```
(A*B+C*D)/100
```

Each register variable must correspond to a meter register. To associate a meter register with a register variable, select the register from the group panel and click the **Add Register +** button and enter the register variable name.

The ternary operator allows for conditional processing of a reading and takes the form of:

```text
CONDITION ? VALUE_IF_TRUE : VALUE_IF_FALSE
```  
where `CONDITION` must be a logical comparison such as `ELECT == 0 or ELECT >= 1` 
and `VALUE_IF_TRUE` and `VALUE_IF_FALSE` are expressions resulting in numeric values.

The comparision operators are: == (equals), &gt; (greater than), &gt;= (greater than or equal to), &lt; (less than), &lt;= (less than or equal to).
Logical operators can also be used in a `CONDITION`. The logical operators are &amp;&amp; (and) and || (or).

Examples:

To return value unless it is negative, in which case make it 0:

```text
ELECT1 >= 0 ? ELECT1 : 0
```

To make values below 1000 or above 3000 = 1000 and all others = 0:

```text
(ELECT1 < 1000 || ELECT1 > 3000) ? 0 : 1000
```

To returns the absolute value:

```text
ELECT1 >= 0 ? ELECT1 : -ELECT1
```

### Modifying a Virtual Meter

- Select the virtual meter from the virtual meters table.
- Click the **Virtual meter settings** button next to the virtual meter name which will display the virtual meter settings form.
- Modify any details entered previously and click the **Save** button.

### Cloning a Virtual Meter

- Select a group in the group panel to display a table of all virtual meters in the group.
- A virtual meter can be cloned by using the action menu on the right of the virtual meter row and clicking **Clone virtual meter**. This will create a new virtual meter settings form filled out with the same values as the previous virtual meter.
- Modify any fields required and then click the **Save** button to create the new virtual meter.

### Moving a Virtual Meter

- Select a group in the group panel to display a table of all virtual meters in the group.
- The Virtual Meter may now be dragged and dropped onto a different group in the group panel.

### Deleting a Virtual Meter

- Select a group in the group panel to display a table of all virtual meters in the group.
- Delete a virtual meter by using the action menu on the right of the virtual meter row and clicking **Delete virtual meter**.

# Accessing Metered Data

## Overview

Metered data for meters and virtual meters can be viewed or downloaded in a variety of ways. The readings can be viewed in a paged tabular format or as a chart and downloaded in a variety of formats.

## Viewing and Downloading Metered Data

- Select the meter or virtual meter and open the **Readings** tab.
- Normally the current day's readings will be displayed by half hour in chart form. The data can be show in either chart form or tabular form by clicking **Chart** or **Table** on the readings toolbar.
- In order to download the data, click the **Download to file** button and select the format from the list.
- For meters the readings data is per register and the register can be selected from a list on the readings toolbar.
- Readings data can be viewed by various time periods, i.e. half hour, hour, day, week, month, by selecting the period from a list.
- Readings data can be viewed for a specific date range by selecting the range from a list. For convenience there are a number of predefined ranges, e.g. today, last 7 days etc, or a specific day or date range can be selected from calendars.

There are a number of options that are used when displaying readings data that can be set in the **Options** panel

Field | Description
------|------------
Source | Identifies the source of the metered data to be viewed. The source can be Automatic (i.e. the normal data collected automatically by DCS), Manual (i.e. the Interpolated data derived purely from Manual Readings) and Merged. When Merged is selected metered data consists of the Automatic metered data (calibrated) with any gaps filled in by linear interpolation OR the interpolated manual reading value which ever interpolation has the least uncertainty. On charts readings derived from either type of interpolated reading are shown with a pale green background.
Calibrated | This is only available when the Source is Automatic. When selected the metered data is shown with the totals adjusted using the Calibration Readings for the register. When not selected the readings are shown without the adjustment.
Interpolated | This is only available when the Source is Automatic. When selected any missing gaps in the metered data will be filled in using linear interpolation (on charts interpolated readings are shown with a pale green background). If not selected missing readings will not be displayed (on charts missing readings are shown with a pale red background).
Time zone | The readings are normally shown in Standard Time (i.e. GMT in the UK) but it is also possible to display the readings in local time (which takes into account BST).
Decimals | The number of decimal places the readings will be rounded to when displayed (Only applicable to Tables)

The data currently being displayed can always be refreshed from the server by clicking the **Refresh from server** button. This is useful if the data may have changed due to, e.g. the meter being read in the background.

# Accessing Billing Data

## Overview

Each Register and Virtual Meter in DCS may optionally have a Tariff associated with it (see
[Managing Tariffs](#managing-tariffs) for how to manage tariffs in DCS). The tariff is used by DCS to calculate the cost associated with a Register or Virtual Meter for a given period of time based on the Metered Data. Note that Registers and Virtual Meters that correspond to Instantaneous values may NOT have Tariffs associated with them.

## Viewing Billing Data

To view the Billing Data for a Register or Virtual Meter:

- Select the meter or virtual meter and open the **\*Billing** tab.
- Normally the current day's billing data will be displayed.
- For meters the billing data is per register and the register can be selected from a list on the readings toolbar.
- Billing data can be viewed for a specific date range by selecting the range from a list. For convenience there are a number of predefined ranges, e.g. today, last 7 days etc, or a specific day or date range can be selected from calendars.

The data currently being displayed can always be refreshed from the server by clicking the **Refresh from server** button. This is useful if the data may have changed due to, e.g. the meter being read in the background.

# Importing Data

## Importing Metered Data

It is possible to import Metered Data into DCS. Imported Metered Data will appear in the system exactly
as data that has been collected automatically by DCS. The imported reading must have a (GMT) timestamp corresponding to a metering period boundary.
If a reading already exists in DCS for the same register at the same time as an imported reading, the reading in DCS will be overwritten.

To import readings:

- Select **Meters** page in the main navigation bar and go to **Meters Home**
- Open the **Date import** panel.
- Select Data Type as _Metered Data_.
- Select the file format from the list
- Select the file to be uploaded.
- Click the **Import** button.

To see the details of the data import plugins see the chapter [Metered Data Import](#data-importexport-plugins-data-import-plugins).

## Importing Manual Readings

It is possible to import Manual Readings into DCS. Imported Manual Readings Data can be used instead of, or as a supplement to, Metered Data. The imported reading must have a timestamp corresponding to a metering period boundary.
If a manual reading already exists in DCS for the same register at the same time as an imported reading, the reading in DCS will be overwritten.

To import readings:

- Select **Meters** page in the main navigation bar and go to **Meters Home**
- Open the **Date import** panel.
- Select Data Type as _Manual Readings_.
- Select the file format from the list
- Select the file to be uploaded.
- Click the **Import** button.

To see the details of the data import plugins see the chapter [Manual Readings Data Import](#data-importexport-plugins-data-import-plugins-manual-readings-data-import).

# Reports

## Overview

While DCS provides a simple means of viewing readings in tables or charts as described above, for more sophisticated data presentation individual reports can be set up.

A report consists of a Report Type (which is defined by its Report Type, See [Report Types](#report-types) for details of individual reports), its Duration (i.e. whether it covers a Period, Hour, Day, Week or Month) and optionally a list of items that are included in it. Items are Groups, Meters, Registers or Virtual Meters and a report can have any combination of items which will then be included in the report. If no items are included the report will include all items in the system. How each item, or each type of item, is handled by a particular report type is a function of the plugin.

Some reports which take more complex input data can have this data defined in a spreadsheet and have that uploaded as part of the report definition. The use of these files is specific to the type of report. See [Report Types](#report-types) for more details.

Reports can be viewed within DCS for any given date and they can optionally be subscribed to in which case they will be automatically emailed at the specified time.

A user may create any number of reports for their own use (Private Report) that will not be available to other users. Users with appropriate authority can create Public Reports that will be available to all users.

## Viewing a Report

- Select **Reports** in the main navigation bar and then select a group in the group panel to display a table of all reports in the group.
- Select the report and click on the **View** tab.
- It is now possible to generate a report for any specific date by clicking the **Generate** button.
- The user may also request that the report is immediately emailed to the email address associated with their account by clicking the **Email** button.
- Once a report is generated it is possible to download a PDF version of the report by clicking the **PDF** button.

Some reports contain attachments, e.g. reports that contain Metered Data as a CSV file. Once a report is generated a **Attachments** tab will be available where the attachments can be downloaded.

## Adding a Private Report

- Select **Reports** in the main navigation bar and then select the **Private reports** group in the group panel. This will display a table of all private reports.
- Click the **Add new report** button at the top right hand of the report table which will display an empty report settings form.
- Enter the fields according to the table below. Note that the
- Press the **Save** button to confirm the creation of the virtual meter.

| Field                           | Description                                                                                                                                                                                                                                                                                                                                                                                                      | When used                                             |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| Name                            | A string for the user to refer to the Report. It does not have any significance to the system.                                                                                                                                                                                                                                                                                                                   | Common                                                |
| Report Type                     | The type of report to be generated.                                                                                                                                                                                                                                                                                                                                                                              | Common                                                |
| Report Frequency                | The duration the report will cover. See below for details                                                                                                                                                                                                                                                                                                                                                        | Common                                                |
| Use local time for metered data | If ticked the timestamps for any metered data presented in the report will be in local time (e.g. British Summer Time during the British Summer). If not set all times will be in standard time (i.e. GMT time in the UK irrespective of time of year).                                                                                                                                                          | Only valid for reports that present Metered Data      |
| Interpolate metered data        | If ticked the metered data will be interpolated if readings are missing                                                                                                                                                                                                                                                                                                                                          | Only valid for reports that present Metered Data      |
| Definition file                 | An Microsoft Excel spreadsheet file (Excel 2007 XLSX format) that defines input parameters for the report generator. The use of this file is a function of the Report Type. To upload a definition file, click the Upload button. To download a copy of the definition file, click the Download button. To remove a definition file from the report specification click the Delete button next to the file name. | Only valid for reports that require a definition file |
| Readings period                 | The reading period used when presenting any Metered Data in the report. Possible values are: Half hour, Hour, Day, Week, Month                                                                                                                                                                                                                                                                                   | Only valid for reports that present Metered Data      |

** Report Frequency **

- Hourly: the report will cover one hour.
- Daily: the report will cover one day.
- Weekly: the report will cover one week from Monday to Sunday.
- Monthly: the report will cover one month from the 1st of the month.
- Per Period: this report will cover one metering period (normally 30 minutes)

## Adding a Public Report

- Select **Reports** in the main navigation bar and then select the **Public reports** group, or any of the groups below, in the group panel to display a table of all reports in the group.
- Proceed as for private reports as above.

## Specifying included items

Some reports require individual report items to be added (e.g. individual meters or meter groups).
In these cases there will be an **Included items** tab will contain a table of included items.

## Subscribing to a Report

The user may subscribe to a report thereby having it periodically emailed to the email address of their account.
To change your subscription settings for a report:

- Select the report from the reports table.
- Click the **Subscription** tab.
- Enter the fields according to the table below (note that some fields are not applicable for some types of reports and may not be present):

| Field                        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Subscribed                   | Tick to have the report automatically emailed.                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Email CC                     | A semi-colon separated list of email addresses that will be sent a copy of the email received by the user.                                                                                                                                                                                                                                                                                                                                                                                          |
| Don't send to me             | Tick to have the report emailed to the addresses in the Email CC field but not to yourself. This is useful if you create a report on behalf of others but do not wish to receive the report yourself. If the Email CC field is empty this setting will be ignored (i.e. you will still receive the report of you are subscribed to it).                                                                                                                                                             |
| Time of day                  | The time of day the report will be generated.                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Day                          | If the Duration of the report is Daily, then the report will be emailed every day at the time specified in the previous field. If the Duration of the report is Weekly, then the report will be emailed once a week at the time specified in the previous field on the day of the week specified in this field. If the Duration of the report is Monthly, then the report will be emailed once a month at the time specified in the previous field on the day of the month specified in this field. |
| Offset (minutes)             | If the Duration of the report is Per Period or Hourly the number of minutes are the end of the period or hour that the report will be generated.                                                                                                                                                                                                                                                                                                                                                    |
| Only send on alarm condition | Some types of reports have the concept of an alarm condition. For these reports it is possible to only have the report emailed if the alarm condition is met.                                                                                                                                                                                                                                                                                                                                       |

## Modifying a Report

- Select **Reports** in the main navigation bar and select the appropriate **Report** in the report tree.
- Click the **Edit** button and the Report data will become editable.
- Modify any details entered previously and press the **Save** button.

## Moving a Report

- Select a group in the group panel to display a table of all reports in the group.
- The Report may now be dragged and dropped onto a different group in the group panel.

> If a private report is moved to the Public Reports group it will become public. It is **not** possible to move a public report back into the Private Reports group to make it private.

## Deleting a Report

- Select a group in the group panel to display a table of all reports in the group.
- Delete a report by using the action menu on the right of the report row and clicking **Delete virtual meter**.

## Managing Report Groups

Public Reports (but not Private Reports) can be grouped in a similar way to Meters and Virtual Meters. The report group panel can contain any number of groups and while they behave in an identical way to groups in the meter group panel the structure of the groups is completely independent.

# Calibration Readings

## General

It is possible to add Calibration Readings to the system which are used to calibrate the meter readings for a particular register.
A calibration reading is usually derived by manually reading the display of the physical meter and recording the value and time.
When this calibration reading is entered into DCS it is used to calculate an offset between the value entered and the reading in the database at the corresponding time (linearly interpolated between half hour values).
The offset is then applied to all readings when displayed or downloaded. This is particularly useful for pulse inputs or radio pulses as it allows all readings to be offset at the time of the calibration reading so the DCS values can correspond to a physical meter's display.
It is also possible to view/download the readings from DCS uncalibrated (i.e. without the offset applied).

## View Calibration Readings

- Select **Meters** in the main navigation bar and then select a group in the group panel to display a table of all meters in the group.
- Select the meter from the meters table and go to the **Reading Modifications** tab and then open the **Calibration readings** panel.
- Select the register of the meter concerned to show the list of Calibration Readings for the register.

## Adding Calibration Readings

- Click the **Add new calibration reading** button at the top right hand of the calibration reading table which will display an empty settings form.
- Enter the fields according to the table below.
- Click **Save** to confirm the creation of the calibration reading.

| Field         | Description                                                                                                                                                                                                                                                                                                                                                 |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Timestamp     | The time (GMT) the calibration reading was made. This will be in the form YYYY-MM-DD HH:mm:ss, e.g. 2018-01-31 23:30:00. Seconds can be omitted.                                                                                                                                                                                                            |
| Reading value | The value of the register when the calibration reading was made. This must be a numerical value.                                                                                                                                                                                                                                                            |
| Valid from    | The date/time from which the calibration is used (GMT), e.g. a calibration may have been taken at sometime during January but a decision may be made to use the offset calculated from the calibration reading for all readings from the start of the year. This will be in the form YYYY-MM-DD HH:mm:ss, e.g. 2018-01-31 23:30:00. Seconds can be omitted. |
| Notes         | Used for user information only.                                                                                                                                                                                                                                                                                                                             |

## Modifying Calibration Readings

- Click the calibration reading row in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

## Deleting Calibration Readings

- Delete a calibration reading by using the action menu on the right of the table row and clicking **Delete calibration reading**.

# Register Resets

## General

Some meters contain registers that are implement as a counter to record an accumulated value.
These registers may, for some reason (e.g. hardware fault, replacement, rollover), have the counter change
to a new value that doesn't reflect the true consumption.
To handle these cases in DCS it is possible to create a _Register Reset_ for a register that will
allow DCS to handle the discontinuity in the readings.

A register reset is specified for a particular register and is assumed to have occurred at the start of a particular metering period.
When entering the Register Reset the operator can provide an estimated value of the total
value at the start of the period (i.e. a final start of period total value before the reset) and a period value for the period.

There are two normal cases where register resets are used:

**The "rollover" case:** the register simply reached a maximum value during a period and rolled over to zero

A register reset in this case will be created with a timestamp equal to the start of the period where the rollover occurred
(i.e. the period which had a high total value immediately before a period that had a very low total value).
The total value should correspond to the the value in the reading since this will be correct (since it occurred before the actual counter rolled over) so the operator can simply tick the _Use total from reading_ box.
The period value must be an estimation of the consumption for that period. It is up to the operator to estimate this value since it can not be precisely.

_Example_

A register with a counter with a maximum value of 65535 rolled over at 10:15:00 on 01/01/2015.

The readings in DCS appear as follows:

| Period value | Total at start | Period start     |
| ------------ | -------------- | ---------------- |
| 10           | 65520          | 01/01/2015 09:30 |
| -65525       | 65530          | 01/01/2015 10:00 |
| 10           | 5              | 01/01/2015 10:30 |
| 10           | 15             | 01/01/2015 11:00 |

In this case the consumption for the period starting at 10:00 is calculated as a negative value by DCS so a Register Reset is required.
A register reset should be created with a timestamp of 01/01/2015 10:00 since this is the start of the period where the reset occurred.
The total value should be the same as that real reading so _Use total from reading_ should be ticked.
The period value should be estimated to 10 (since the consumption before and after the period is 10 this seems a reasonable estimation).

** The "changed meter" case:** the meter was replaced and there are missing readings before the new readings start at a different counter value

A register reset in this case will be created with a timestamp equal to the start of the first new reading immediately after the gap.
The period value must be an estimation of the consumption for that period (it is up to the operator to estimate this value since it can not be precisely).
The total value should be an estimation of the total value at the start of the period based on the old value of the counter
(i.e. an estimation must be made of what the old counter value would be at the end of the gap).
It is up to the operator to estimate this value since it can not be precisely.

_Example_

A meter was replaced due to a hardware fault at 10:45 and there is a gap in the readings while this happened.
The final available value for the register in the old meter was 20000. Once the new meter was installed and started metering at 13:55 the register value has started again at zero.

The readings in DCS appear as follows:

| Period value     | Total at start   | Period start       |
| ---------------- | ---------------- | ------------------ |
| 10</td><td>19990 | 01/01/2015 09:30 |
| 10               | 20000            | 01/01/2015 10:00   |
| _0_              | _20010_          | _01/01/2015 10:30_ |
| _..._            | _..._            | _..._              |
| _0_              | _20010_          | _01/01/2015 13:30_ |
| 10               | _3_              | _01/01/2015 14:00_ |
| 10               | 13               | 01/01/2015 14:30   |

A register reset should be created with a timestamp of 01/01/2015 14:00 since this is the start of the period where the new meter is metering and therefore is
considered to be the point at which the reset occurred.
The total value should be an estimation of the total at the start of this period based on the old register and so should be set to, e.g.,
70 (since the last known value of the old register was 20010 at 10:30 and the consumption
is averaging 10 per period then this seems a reasonable estimation).
The period value should be estimated to 10 (since the consumption
is averaging 10 per period this seems a reasonable estimation).

To manage Register resets:

- Select **Meters** in the main navigation bar and then select **Register Resets** in the Meter Operation drop down menu now displayed in the toolbar.
- Select the register of the meter concerned and DCS will display a list of all Register Resets for the register on the main panel.

## View Register Resets

- Select **Meters** in the main navigation bar and then select a group in the group panel to display a table of all meters in the group.
- Select the meter from the meters table and go to the **Reading Modifications** tab and then open the **Register Resets** panel.
- Select the register of the meter concerned to show the list of Register Resets for the register.

## Adding Register Resets

- Click the **Add new register reset** button at the top right hand of the Register Reset table which will display an empty settings form.
- Enter the fields according to the table below.
- Click **Save** to confirm the creation of the register reset.

| Field             | Description                                                                                                                                                                                                                                                                                                                                    |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Timestamp         | The time at the start of the metering period that the reset occurred.. This will be in the form dd/MM/yyyy HH:mm. Note that if a time is entered that does not correspond to the start of a metering period it will be "rounded" down to the nearest valid time.                                                                               |
| Period value      | The estimated period value (i.e. consumption) of the register for the metering period when the reset occurred. This must be a numerical value.                                                                                                                                                                                                 |
| Total Value       | The estimated total value of the register immediately before the reset occurred. . This must be a numerical value.                                                                                                                                                                                                                             |
| Use Reading Value | If set the Total Value above will be ignored and the total value from the metered data will be used instead. This would be the case where the metered data contains a correct total value at the start of the metering period.The estimated total value of the register immediately before the reset occurred. This must be a numerical value. |
| Notes             | Used for user information only.                                                                                                                                                                                                                                                                                                                |

## Modifying Register Resets

- Click the register reset row in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

## Deleting Register Resets

- Delete a register reset by using the action menu on the right of the table row and clicking **Delete register resets**.

# Invalid Reading Ranges

## General

An invalid reading range can be specified for a register which will effectively remove the readings in the system for the specified time range. It is as if the readings were deleted from DCS but they can be _undeleted_ at anytime by deleting the Invalid Reading Range.

This is useful when a range of readings that have been collected by DCS are known to be faulty for some reason, e.g. a meter hardware fault. By combining an Invalid Reading Range with reading interpolation when viewing Metered Data DCS will ignore the faulty readings and interpolate over the timespan.

## View Invalid Reading Ranges

- Select **Meters** in the main navigation bar and then select a group in the group panel to display a table of all meters in the group.
- Select the meter from the meters table and go to the **Reading Modifications** tab and then open the **Invalid Reading Ranges** panel.
- Select the register of the meter concerned to show the list of Invalid Reading Ranges for the register.

## Adding Invalid Reading Ranges

- Click the **Add new invalid reading Range** button at the top right hand of the invalid reading range table which will display an empty settings form.
- Enter the fields according to the table below.
- Click **Save** to confirm the creation of the calibration reading.

| Field      | Description                                                                                                                                                                                                                                    |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Start time | The time (GMT) that specifies the start of the invalid range in the form YYYY-MM-DD HH:mm:ss, e.g. 2018-01-31 23:30:00. Seconds can be omitted. The start time is inclusive, i.e. all readings with a timestamp >= start time will be invalid. |
| End time   | The time (GMT) that specifies the end of the invalid range in the form YYYY-MM-DD HH:mm:ss, e.g. 2018-01-31 23:30:00. Seconds can be omitted. The end time is exclusive, i.e. all readings with a timestamp < end time will be invalid.        |
| Reason     | A mandatory explanation of why the readings are invalid.                                                                                                                                                                                       |

> Both start time and end time are required.

## Modifying Invalid Reading Ranges

- Click the invalid reading ranges row in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

## Deleting Invalid Reading Ranges

- Delete an invalid reading range by using the action menu on the right of the table row and clicking **Delete invalid reading range**.

# Managing Meter Types

## Viewing Meter Types

- Select **Mete Types** in the main navigation bar to display a table of all Custom and System Defined meter types.
- To select an individual Meter Type click on the row in the Meter Types table.

## Adding a Meter Type

- Click the **Add new meter type** button at the top right hand of the meter type table which will display an empty meter type settings form.
- Enter the fields according to the table below.
- Press the **Save** button to confirm the creation of the virtual meter.

| Field                 | Description                                                                                                                                                                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Meter class           | The class of meter see below for more details.                                                                                                                                                                                                 |
| Name                  | A human readable name for the Meter type.                                                                                                                                                                                                      |
| Description           | Used for user information only.                                                                                                                                                                                                                |
| Metered data plugin   | If the meter type corresponds to an intelligent meter, then select the Metered Data Plugin from the drop down list. If the meter type corresponds to an IDC pulse input, radio input and Modbus meter this field should be left as **None**.   |
| Real time data plugin | If the meter type corresponds to an intelligent meter, then select the Real Time Data Plugin from the drop down list. If the meter type corresponds to an IDC pulse input, radio input and Modbus meter this field should be left as **None**. |

**Meter class**
The possible values for Meter Class are

| Value                         | Usage                                                                                                                                                                                                            |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Intelligent meter with plugin | This is generally used for meters that store their own half hour data and are read by DCS using a plugin that implements the meter protocol. When this value is selected a Metered Data plugin will be required. |
| Pulse input                   | This is used when the meter corresponds to a pulse counter (or radio pulse input) that DCS connects to via and IDC.                                                                                              |
| Modbus meter                  | This is used when the meter corresponds to a Modbus meter that DCS connects to via an IDC. When this value is selected the register types will require Modbus specific settings.                                 |
| Generic                       | This option is only for backwards compatibility and is not required for new Meter Types.                                                                                                                         |

## Modifying a Meter Type

- Click the meter row in the table to select it and click the **Meter type settings** button. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

> If the definition of a _Meter Type_ is changed
> this will be reflected in **all** existing meters of
> that type, and all
> meters subsequently defined.

## Deleting a Meter Type

- Delete a meter type by using the action menu on the right of the table row and clicking **Delete meter type**.

> A Meter Type cannot be deleted if meters exist of this type.

## Viewing Register Types in a Meter Type

- Click the meter row in the table to select it. This will display a table of Register types.

## Adding a Register Type to a Meter Type

- Click the **Add new register type** button at the top right hand of the register type table which will display an empty settings form.
- Enter the fields according to the table below.
- Click **Save** to confirm the creation of the calibration reading.

Field | Description
------|------------ 
Name | A human readable name for the Register Type. 
Address | See below
Metered data plugin | If the meter type corresponds to an intelligent meter then select the Metered Data Plugin from the drop down list (see the chapter on **Metered Data Plugins** for more information on supported plugins.) If the meter type corresponds to an IDC pulse input, radio input and Modbus meter this field should be left as **None**.
Scale factor | A number which every reading will be multiplied by before it is stored in the database.
Unit | Unit for the readings.
Is instantaneous | Specifies whether the register is recording an instantaneous quantity (e.g. W) or a cumulative quantity (e.g. KWh).

> An instantaneous Register will store values recorded at the given time (e.g. W), while a non-instantaneous or standard Register will assume this is a cumulative quantity (e.g. KWh) and also give a "Period Value" being the difference between this reading and the next representing the accumulated total for the given period.

**Address format for IDC Inputs:**

- Pulse Input: The address is always 0
- For Radio Inputs: The address is always 0
- For Modbus meters: The address is the Modbus register address and these additional parameters must be configured (see relevant meter documentation):
 - The type of register which can be Holding Register or an Input Register
  - The length in 16 bit words (valid values are 16, 32, 48 and 64 bit word sizes)
  - The encoding which is shown below. Note that not all combinations of length and encoding are supported. Refer to the relevant meter documentation for register formats.
- For meters that use a plugin, the address is used by the Meter Plugin to address the register within the meter and its meaning is specific to the plugin.
 
**Modbus register encoding**

- Unsigned integer: Big/Little endian
- Unsigned integer (Modulo-10000): Big/Little endian
- Unsigned integer (YST format)
- Signed integer (2s complement): Big/Little endian
- Signed integer (sign/magnitude): Big/Little endian
- Floating Point to IEEE 754 (32 bit only): Big/Little endian
- Binary-coded decimal (BCD)
        
## Removing a Register Type from a Meter Type

- Delete a register type by using the action menu on the right of the table row and clicking **Delete register type**.

> if a register type is removed from an existing meter type the change will be reflected in
> all existing Meters (in
> other words registers will be removed from existing meter
> definitions including all metered data for those registers). Changes will, of course, also be reflected when new meters are defined.

## Modifying a Register Type</a>

- Click the register type row in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

> If the definition of a _Register Type_ is changed, this will be reflected in **all** existing meters of
> that type, and all meters subsequently defined.

# Managing IDCs

## Overview

DCS may use Coherent Research Internet Data Collectors (IDCs) to communicate with meters. DCS also allows users to view and manage these devices.

It is possible to partition IDCs logically into a hierarchy of groups in exactly the same way as meters and virtual meters. The hierarchy can be considered as a tree and meters are only assigned to the leaves of the tree. Meters can be moved from one group to another without effect.

## Viewing IDCs

- Select **IDCs** in the main navigation bar and then select a group in the group panel to display a table of all IDCs in the group.
- To select an individual IDC click on the row in the IDC table.
- Alternatively click on the **Show flat list** button to display all IDCSin a flat list and use the filter bar to narrow the list down to the required IDCs.

## Adding IDCs

IDCs are automatically added to DCS when they first connect. One IDC Group will be defined as the default group and IDCs will automatically be assigned to that group (see [Managing IDC Groups](#managing-idcs-managing-idc-groups)).

## Viewing IDC status
- Select the IDC and open the **Device Info** tab. This will show information about the IDC device and its current status. 
- The status can be updated by clicking the  **Refresh** button.
- The user may store information as Notes which, when added or edited, can be saved with the **Save** button. These notes do not effect the use of the IDCs in any way and can be modified even if the IDC is offline.
        
## Reading the IDC settings

It is possible to read the settings stored in the IDC device itself and modify these if required.

- Select the IDC and click the **IDC settings from remote device** button.

> This sets up a communications session with the remote device to read the settings and may take a few seconds. Also, be aware that if there is already a session in progress to the IDC to, e.g., read a meter it will not be possible to read the settings.

## Modifying the IDC settings

- Read the IDC **Settings** and modify them as required according to the table below.
- Click the **Write to IDC** button to write the settings back to the remote device.

> Note that this result in the IDC device being rebooted. This sets up a communications session with the remote device to read the settings and may take a few seconds. Also, be aware that if there is already a session in progress to the IDC to, e.g., read a meter it will not be possible to read or write the settings until this is completed.
        
Field | Description
------|------------ 
Name | A descriptive name for the IDC. This is for the administrator's benefit only. Maximum 20 characters.
Server address | The address of DCSWebServices. This can either be a fully qualified domain name such as www.coherent-research.co.uk or an IP address. Only used if the TCP listening port is set to 0.
Server port | The HTTP listening port for DCSWebServices (normally 80). 
Service directory | The directory that forms the complete URL of the DCS endpoint. This will normally be IdcWebServices.
DHCP | Determines if DHCP is used to obtain network settings.
IP Address | The static IP address of the IDC (only valid if DHCP is No).
Gateway address | The gateway IP address  (only valid if DHCP is No).
Netmask | The subnet mask for the IDC  (only valid if DHCP is No).
DNS address | The IP address of the DNS server (only valid if DHCP is No).
Proxy | Determines if a HTTP proxy server should be used for connections to DCS.
Proxy server address | The IP address for the proxy server (only valid if Proxy  is Yes).
Proxy server port | The listening  port for the proxy server (only valid if Proxy  is Yes).        
Modbus serial settings | The serial port settings for the external Modbus port when communicating with Modbus devices in the form "b, dps" where b is the baud rate, d is the number of databits (7, 8), p is the parity (O, E, N) and s is the number of stop bits (1, 2). E.g. 1200, 7E1.
Delay1 | Time delay (in ms) inserted between receiving a modbus message from a modbus address and sending to the same address. 0-10000, default 0.
Delay2 | Time delay (in ms) inserted between receiving a modbus message from a modbus address and sending to another address. 0-10000, default 0.
Delay3 | An extra duration, in milliseconds, added to the timeout used by the IDC when polling modbus meters. Normally this value can be zero as the IDC will use an appropriate timeout. However, for some meters that are slow to process Modbus requests an extra duration can be added. 0-10000, default 0.
Modbus retries | The number of times the IDC will retry to read a Modbus meter. 0-10, default 0.
Modbus/TCP port | If specified the IDC will act as a Modbus over TCP/IP gateway. Default 0. If zero the feature is disabled. The normal value when the feature is used in 502.
Trace On | Turn on/off UDP tracing for the IDC.
Trace Server Address | The address for the Trace Server. This can either be a fully qualified domain name such as www.coherent-research.co.uk or an IP address.
Trace Server Port | The UDP listening port for the Trace Server.

> Note that it is possible to change all IDC settings via DCS, including
> those which determine how the IDC connects to DCS, so care must be
> taken when changing these.

## Resetting IDCs

It is possible to issue a command to an individual IDC to reset itself. This is normally not required but may be useful for diagnostic purposes.

- Select a group in the group panel to display a table of all IDCs in the group.
- An IDC can be reset by using the action menu on the right of the IDC row and clicking **Reset device**.

## Managing IDC Groups

IDCs can be grouped in a similar way to Meters and Virtual Meters. The report group panel can contain any number of groups and while they behave in an identical way to groups in the meter group panel the structure of the groups is completely independent.

IDC Groups have a field _Default_ which can optionally be set on one Group. If this is set all new IDCs will automatically be allocated to this group when they first connect to DCS.

## Managing Modbus Devices

### Overview

Modbus devices fall into 2 categories: managed Modbus devices and unmanaged Modbus devices. Managed Modbus devices are devices produced by Coherent Research which can be
fully managed via DCS. These devices include Coherent Research Pulse Counters and Coherent Research Radio Receivers. All other Modbus devices are considered unmanaged devices.
It is not possible to add, delete, edit or setup unmanaged Modbus devices (these devices are created/deleted automatically when a meter is defined that references them) but the status of these devices is
check and it is possible to test that the communications to the device is working.

### View the Modbus Devices

- Select the IDC and open the **Modbus devices** tab. This will show a table of all the Modbus device defined for the IDC device and its current status.
- To select an individual Modbus device click on the row in the device table.

### Adding a Managed Modbus Device

- Click the Add new Modbus device button at the top right hand of the Modbus device table which will display an empty settings form
- Enter the fields according to the table below.
- Press the **Save** button to confirm the creation of the new device.

| Field         | Description                                                                                           |
| ------------- | ----------------------------------------------------------------------------------------------------- |
| Description   | A string for the user to describe the Modbus Device. It does not have any significance to the system. |
| Type          | The type of managed Modbus device. This can be either Pulse Counter or Radio Receiver.                |
| Address       | The Modbus address of the device.                                                                     |
| Serial number | The serial number of the managed Modbus device. This is in the form of 8 alphanumeric characters.     |

### Editing a Modbus Device

- Click the Modbus device row to select it. This will display the Modbus device settings form.
- Modify any details entered previously and click the **Save** button.

### Testing a Modbus Device

DCS will show the status of the Modbus device the last time the IDC tried to communicate with it. When commissioning Modbus devices it is often useful to
check the connection immediately. DCS provides the ability to send a message to any modbus device (managed or non-managed) to ensure that it is responding.

- Select the IDC and open the **Modbus devices** tab to show the table of Modbus devices.
- Test the device by using the action menu on the right of the Modbus device row and clicking **Test comms**.

### Setting up a Modbus Device

When a managed modbus device is first connected to the system its Modbus address and serial settings can be
automatically set from DCS. This can be done without knowing the current address or serial settings of the device but it does require that the serial number is known.
DCS will set the device to the address specified when the Modbus devices was added to DCS and the serial settings to those defined for the Modbus in the IDC.

- Select the IDC and open the **Modbus devices** tab to show the table of Modbus devices.
- Test the device by using the action menu on the right of the Modbus device row and clicking **Setup device**.

### Deleting a Modbus Device

- Select the IDC and open the **Modbus devices** tab to show the table of Modbus devices.
- Delete the device by using the action menu on the right of the Modbus device row and clicking **Delete device**.

### Editing Modbus Pulse Counter Settings

Coherent Research Pulse Counters have settings that can be modified via DCS.

- Select the IDC and open the **Modbus devices** tab to show the table of Modbus devices.
- Delete the device by using the action menu on the right of the Modbus device row and clicking **Edit pulse settings**.

| Field                    | Description                                                                                                                                                                                                                                                                       |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Minimum pulse width      | The minimum time in milliseconds that the pulse counter will recognise a contact as a valid pulse. Any contact less than this duration will be ignored. This value can be set individually for each pulse input.                                                                  |
| Stuck bit detection time | The time in milliseconds above which the pulse counter will declare the pulse input stuck. An input that is declared stuck will still function normally but a flag will be set indicating that a stuck bit was detected. This value can be set individually for each pulse input. |
| Bit stuck                | This settings shows the current status of the stuck bit indicator for each input. Note that the flags can not be set individual but all flags are cleared by writing the settings back to the pulse counter.                                                                      |

## View Internal IDC logs

It is possible to view 2 internal IDC logs: the Event log and the Radio Message log. The Event log contains all events from the IDC along with all
events from the managed Modbus devices.

- Select the IDC and open the **Log files** tab.
- Click on **Fetch event log** to read the and display the event log from the IDC.
- Click on **Fetch radio log** to read the and display the radio log from the IDC.

> IDCs that do not have any radio receivers will return an empty radio log.

# Interrogating Meters

## Normal Interrogation

While DCS is designed to interrogate meters automatically and store the data in its database, there are occasions when it is necessary
to interrogate a meter manually for a given time period, e.g.
when a meter is initially added to DCS it may be necessary to read its historical data,
or if it has not been possible to interrogate a meter automatically for some time due to a technical fault it may be necessary to interrogate the meter manually to fill in the gap.

- Select the meter and open the **Interrogation** tab.
- Select **Data collection** from the list and pick the date range you wish to interrogate the meter over and click the **Start** button.
- DCS now fetches the data from the meter within the specified date range and overwrites this to the database.
- While the interrogation is occurring, the main panel displays the trace of information exchanged between DCS and the meter.
- The interrogation will complete automatically when all the data is read. Alternatively the session can be terminated at any time by clicking the **Cancel** button.

> Normal interrogation is supported for intelligent meters.

## Real Time Interrogation

It is also possible to read real time data from a meter.
When a real time interrogation is requested, a connection is established to the meter as normal
and will be maintained until it is manually stopped by the user. While connected, DCS will continually read a predefined set of registers and display them.

Select the meter and open the **Interrogation** tab.

- Select **Real time** from the list and click the **Start** button.

> Realtime interrogation is supported for intelligent meters which have a Real Time Data Plugin defined in their Meter Type, or for Modbus Meters or Modbus Pulse Counters.

## Communications Test

In is also possible to test the communications with the meter by selecting the Interrogation Type **Comms Test**.
This will attempt to connect to the meter and login but not read any historical or real time data from the meter.

> A Comms Test is supported for intelligent meters and for Modbus Meters or Modbus Pulse Counters.

# Queued Interrogations

## Overview

A Queued Interrogation allows a user to order DCS to interrogate a meter, or set of meters, in the background without the user needing to sit through the actual interrogation session. This can be useful during commissioning for example.

There are 2 types of queued interrogations:

### Data collection

When selecting **Data collection** for a queued interrogation DCS will interrogate the meter exactly as if it was doing a normal automatic data collection. In other words, all outstanding metered data will be read up to the current time.

### Comms test

When selecting **Comms test** for a queued interrogation DCS will interrogate the meter exactly as if it was doing a Manual Interrogation with the Comms test interrogation type, i.e. it will simply ensure that it can contact the meter and log on to it but it will not collect any metered data.

It is possible to initialise a queued interrogation on a meter basis or for a whole group.

## Queuing an interrogation for a meter

- Select **Meters** in the main navigation bar and then select a group in the group panel to display a table of all meters (and virtual meter) in the group.
- Order a queued data collection communication test interrogation for an individual meter by using the action menu on the right of the meter row and clicking **Queue data collection** or a **Queue communication test** respectively.

## Queuing an interrogation for all meters in a group

- Select **Meters** in the main navigation bar and then select a group in the group panel.
- Order a queued data collection communication test interrogation for all meters in the group by using the action menu on the right of the group name and clicking **Queue data collection** or a **Queue communication test** respectively.

## Viewing interrogation results

When one or more interrogations are queued they will be added to the list and DCS will perform the interrogation in the background. To track the progress of queued interrogations and view results go to the Meters home page:

- Select Meters in the main navigation bar and click **Meters home** and open the **Queued Interrogation** panel.
- The user can
  click the **Refresh** button to update the current status.

> Completed interrogations will be automatically removed from the list after a predefined time. The time after which completed interrogations are automatically removed from the list can be configured in the DCSService.exe.config appSettings with setting **MaxAgeOfCompletedQueuedInterrogations_m** and is specified in minutes (default value 24\*60).

## Application settings

A number of application settings can be used to modify the behaviour of the Queued Interrogation function in the DCService.exe.config file:

Setting name | Meaning | Default value 
--|--|--|--
QueuedInterrogationDialupMaxRetryCount | Number of retries that will be attempted if a dial up queued interrogation fails. | 2 
QueuedInterrogationTcpMaxRetryCount | Number of retries that will be attempted if a TCP/IP queued interrogation fails. | 2 
QueuedInterrogationIdcMaxRetryCount | Number of retries that will be attempted if an IDC queued interrogation fails. | 2 
QueuedInterrogationDialupRetryGaps_s | The minimum gap in seconds between retries for dialup queued interrogations | 30 
QueuedInterrogationTcpRetryGaps_s | The minimum gap in seconds between retries for TCP/IP queued interrogations | 30 
QueuedInterrogationIdcRetryGaps_s | The minimum gap in seconds between retries for IDC queued interrogations | 30 

See the installation guide for details about modifying application settings.

## Caveats and limitations

- If a queued interrogation is not successful it marked as failed in the queued interrogation list. It is possible to see the reason an interrogation failed in the event log if required.
- DCS will combine queued interrogations with the same remote address/IDC into a single connection providing they are queued at the same time, i.e. they all belong to the same group, or they are all part of the same filtered list.
- If multiple queued interrogations are requested to the same remote address/IDC individually DCS will not try and combine these requests
  into a single connection. Since multiple queued interrogations are performed simultaneously this may mean that some of the interrogations will fail because a connection can not be established.

## Prerequisites

The Queued Interrogations functionality requires the following modification to the DCService.exe.config file:

```xml
 <system.runtime.remoting>
    <application name="DCService">
      <service>
        <wellknown type="Coherent.DCS.RemoteInterface.RemoteMeterReader, RemoteMeterReader"
		           objectUri="Coherent.DCS.RemoteInterface.RemoteMeterReader" mode="Singleton" />
        <wellknown type="Coherent.DCS.RemoteInterface.RemoteConfigurer, RemoteMeterReader"
		           objectUri="Coherent.DCS.RemoteInterface.RemoteConfigurer" mode="Singleton" />
        <wellknown type="Coherent.DCS.RemoteInterface.RemoteRealTimeMeterReader, RemoteMeterReader"
		           objectUri="Coherent.DCS.RemoteInterface.RemoteRealTimeMeterReader" mode="Singleton" />
		<!-- New line below -->
        <wellknown type="Coherent.DCS.RemoteInterface.RemoteInterrogationQueuer, RemoteMeterReader"
		           objectUri="Coherent.DCS.RemoteInterface.RemoteInterrogationQueuer" mode="Singleton" />
		<!-- End of modification -->
      </service>
```

After any change to the DCService.exe.config file DCService must be restarted.

# Managing Tariffs

## Overview

A tariff is used to determine how to calculate the cost associated with a set of readings. Any Register or Virtual Meter in DCS can have a Tariff associated with it allowing Billing Data to be generated for a give time span. Any number of Tariffs can be defined in DCS. It is possible for more than one Register or Virtual Meter to use the same tariff.

## Viewing Tariffs

- Select **Admin > Metering settings** in the main navigation bar and then open the **Tariffs** panel to display a table of all tariffs.

## Adding a Tariff

- Click the **Add new tariff** button at the top right hand of the tariff table which will display an empty tariff settings form.
- Enter the fields according to the table below.
- Click **Save**.

| Field              | Description                                                                                                                                                                                                                                                                                                     |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name               | A human readable name for the Tariff. It should be something concise and obvious.                                                                                                                                                                                                                               |
| Standing Charge    | The fixed cost (in pounds and pence) per month. This may be 0.                                                                                                                                                                                                                                                  |
| Times are local    | Determines whether Standard or Local time is used when calculating billing data. When 'Times are local' is selected the times in the Daily Tariff Bands are in Local Time and the time span used when calculating billing data are in Local Time. When 'Times are local' is NOT selected Standard Time is used. |
| Daily tariff bands | Each tariff must have one or more time bands during the day that specify the cost associated with that time of day. Each tariff time band consists of the following:                                                                                                                                            |

**Tariff band definition**

Field | Description
------|------------ 
Name |  A short human readable name, e.g. Day or Night.
Start time | The time of day (GMT) the band starts in the format HH:MM (in 24hr time).
Unit cost | The cost (in pounds and pence) per unit.
Day of week | The days of the week the tariff is valid for. This can be used to have, e.g., different tariffs on the weekends as opposed to weekdays.

> Note that the end time for a tariff time band is always considered the start of the next time band (in start time order). This means that no 2 tariff time bands can have the same start time. The cost per unit can be in fractions of pence, e.g. if a supplier charges 12.34 p/kWh this can be enterd as 0.1234. Tariff time band names do not need to be unique within a tariff. Repeated time bands will be aggregated when creating the billing data. This allows rates to be split during the day.

E.g. if a supplier charge 1p/unit during the night and 2p per unit from 8am to 8pm this could be entered as:

```
Night, 00:00, 0.01
Day, 08:00, 0.02
Night, 20:00, 0.01
```

When the total cost is calculated it will be presented as 2 bands Night and Day.

## Modifying a Tariff

- Click the tariff in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

## Deleting a Tariff

- Delete a tariff by using the action menu on the right of the table row and clicking **Delete tariff**.

# Managing Modems

## Overview

It is possible to use _Modems_ to connect
to certain types of meters. When a meter is defined with a Connection
Method of Modem, then DCS will scan all defined modems and use the first
available one (providing the Remote Address Filter allows the meter's
Remote Address to be dialled, see below).

There is no limit in DCS as to the number of Modems which
can be
supported (although the underlying operating system will probably
provide some limitation such as the number of serial ports it can handle).

## Viewing modems

- Select **Admin > Communications devices** in the main navigation bar and then open the **Modems** panel to display a table of all modems.

## Adding a Modem

- Click the **Add new modem** button at the top right hand of the modem table which will display an empty modem settings form.
- Enter the fields according to the table below.
- Click **Save**.

| Field                 | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Name                  | A string for the user to refer to the Modem. It does not have any significance to the system, device names, port names etc.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Baud                  | Serial baud rate.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Data bits             | Serial data word size in bits.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Parity                | Serial parity.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Stop bits             | Serial stop bits.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Port                  | The serial port for the modem.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Initialisation string | Modem initialisation string. This string will be prepended with AT and sent to the modem before each call.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| End of message        | Timeout value (in ms) defines the time after the reception of a character that is interpreted as an end of message by DCS.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Remote Address Filter | The string takes the form of a semi-colon separated list of subfilters. Each subfilter can be an allowed number, e.g. '01273888444', or number series, e.g. '01273\*', or a disallowed number, e.g. '-01273888444', or number series, e.g. '-01273\*'. When a meter interrogation is performed DCS will select the next idle modem. If the Remote Address Filter is set for a particular modem, it will only be used if the meter's remote address is allowed by the filter. Examples: to allow all numbers except those starting with 07: '-07\*', to allow all numbers starting with 08 except those starting with 081: '08\*; -081\*'. No filter is the equivalent of allowing all numbers. |
| Dialling prefix       | A string which will be prepended to the meter's remote address when the modem establishes a connection.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

## Modifying a Modem

- Click the modem in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

## Deleting a Modem

- Delete a modem by using the action menu on the right of the table row and clicking **Delete modem**.  


# Managing Collection Schedules

## Overview

Collection Schedules are used to define when a modem with the appropriate connection method is interrogated for data collection. A meter which uses a modem for data collection will be interrogated once per day according to a specific collection schedule. A meter which uses TCP/IP for data collection can be interrogated every period or once per day according to a specific collection schedule.
A schedule defines a window of time that a meter can be called and the number of times DCS can try to call it. In order to interrogate such a meter, at least one Collection Schedule must be defined.
When installed, DCS has a default Collection Schedule defined which will most likely be suitable for most installations.

## Viewing Collection Schedules

- Select **Admin > Metering settings** in the main navigation bar and then open the **Collection schedules** panel to display a table of all modems.

## Adding a Collection Schedule

- Click the **Add new collection schedule** button at the top right hand of the collection schedule table which will display an empty collection schedule settings form.
- Enter the fields according to the table below.
- Click **Save**.

| Field          | Description                                                                                                                                  |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Name           | A string for the user to refer to the Collection Schedule. It does not have any significance to the system.                                  |
| Time of day    | The start of the window that the collection may take place. (Note that this does not mean that the meter will be interrogated at this time). |
| Window Size    | The size of the window of time (in minutes) that the meter can be interrogated.                                                              |
| Maximum tries. | The number of times DCS may try to connect to the meter when a connection fails.                                                             |

## Modifying a Collection Schedule

- Click the collection schedule in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

## Deleting a Collection Schedule

- Delete a collection schedule by using the action menu on the right of the table row and clicking **Delete collection schedule**.  


> It will not be possible to delete a Collection Schedule if it is being used by a Meter.

# Managing Units for Metered Data

## Overview

All meter registers and virtual meters can have a unit specified. In DCS a unit is simply a label and does not have any other meaning. DCS has a number of utilities based units predefined but a user may add, delete or modify these as required.

## Viewing units

- Select **Admin > Metering settings** in the main navigation bar and then open the **Units** panel to display a table of all units.

## Adding a Unit

- Click the **Add new unit** button at the top right hand of the unit table which will display an empty settings form.
- Enter the unit name.
- Click **Save**.

## Modifying a Unit

- Click the unit in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

## Deleting a Unit

- Delete a unit by using the action menu on the right of the table row and clicking **Delete unit**.

> A unit cannot be deleted if it is used by a register type or virtual meter.

# Managing Users

## User Roles

Role | Capabilities | Usage
-----|--------------|------
Administrator | Adminstrator are permitted to perform all tasks in DCS | Limit the number of Administrators to a small number of trusted, capable users.
Operators | An operator can create, edit and delete meters, virtual meters or groups and manage IDCs unless they are restricted to certain groups by an administrator (see [User Restricted Groups](#user-restricted-groups)). An operator may view Meter Types but not create, delete or modify them. They may not view the **Admins** page. They may also view and create their own reports and view all public reports. | This role is designed for users who will use all aspects of the system be may be restricted to certain groups of meters or IDCs.
Viewer | The viewer may view virtually all parts of the system but they have stricly read only access. Viewers can not create, edit and delete meters, virtual meters or groups. Viewers can be restricted to certain groups by an administrator (see [User Restricted Groups](#user-restricted-groups)). Viewer may view but not edit Meter Types, IDCs and Administration functions with the exception of the Users table. They may also view and create their own reports and view all public reports. | This role is usually just used for demonstation purposes.
Guest | Guests are essentially only allowed to view Metered Data. They can view any of the groups, meters (meter passwords and remote addresses will be hidden) or virtual meters unless they are restricted to certain groups by an administrator (see [User Restricted Groups](#user-restricted-groups)); however they may not create, modify or delete any of them. They may not interrogate meters or import any data. They can not interact with IDCs, view the event log or access any administrative funtions. They may view and create their own reports and view all public reports. | Use this role if the user needs to be restricted to a subset of meters and should only be authorised to view Metered Data  

## Viewing users

- Select **Admin > User admin** in the main navigation bar and then open the **User accounts** panel to display a table of all users.

## Adding a User

Field | Description
------|------------ 
User name | A unique string which may only be made up of the following characters: a..z, A..Z, 0..9, -, ., _, @, +, and *SPACE*.
Role | Administrator, Operator, Viewer, Guest
Email | The user's email address
SSO Identity | Optional field for the user's fully qualified identity on the external Single-Sign-On provider if available.
Enforce SSO | If an SSO Identity is provided, ticking this box will cause DCS to exclusively use this method to authenticate the user, i.e. the user must use SSO to login and will not be able to use a user name and password from DCS.
Expiration | An optional future date that can be used to specify an expiration date for the account. Once this date has passed the user will not be able to sign in. Normally this field should be empty.
Data access start | An optional date that can be used to specify the start of a date range for which the user can view metered data. Normally this field should be empty.
Data access start | An optional date that can be used to specify the end of a date range for which the user can view metered data. Normally this field should be empty.
Meter restriction profile | An optional profile that can be applied to the user to restrict them to a subset of meter groups
IDC restriction profile | An optional profile that can be applied to the user to restrict them to a subset of IDC groups

## Modifying a User

- Click the user in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.

> A user is uniquely defined by their user name and as such it is not possible to change the it. In order to change a user's name the user must be deleted and added with the new name.

## Deleting a User

- Delete a user by using the action menu on the right of the table row and clicking **Delete user**.

## Meter Restriction Profiles

A Meter Restiction Profile is a nested list of Denied/Granted Meter Groups, Meters and/or Virtual Meters. A few examples best illustrates the use of a restriction profiles.

** Example 1: A set of users is only granted access meters in a given group **

A Meter Restriction Profile can be created which has the default access set to **Denied** which means they can not access any groups by default. A meter group can then be added which has the access set to **Granted**. This will mean that user will only be able to access this granted group.

** Example 2: A set of users is granted access all meters except those in a given group **

A Meter Restriction Profile can be created which has the default access set to **Granted** which means they can access any groups by default. A meter group can then be added which has the access set to **Denied**. This will mean that user will be able to access all groups except this denied group.

** Example 3: A set of uers is only granted access meters in a given group, except a certain meter **

A Meter Restriction Profile can be created which has the default access set to **Denied** which means they can not access any groups by default. A meter group can then be added which has the access set to **Granted**. This will mean that user will only be able to access this granted group. To restrict access to a certain meter this can be added with its access set to **Denied**.

### Viewing Meter Restriction Profiles

- Select **Admin > User admin** in the main navigation bar and then open the **Meter restriction profiles** panel to display a table of all profiles.

### Adding Meter Restriction Profiles

- Click the **Add new meter restriction profile** button at the top right hand of the table which will display an empty settings form.
- Enter the fields according to the table below.
- To add new meter group, meters and/or virtual meters to the profile click the **Add profile item** button and set the access as required.
- Click **Save**.

Field | Description
------|------------ 
Name |  A string for the user to refer to the profile. It does not have any significance to the system.
Default access | Specifies whether the default profile access is **Granted** (i.e. by default a user with this profile can access everything) or **Denied** (i.e. by default a user with this profile cannot access anything)
User default | If selected new users (except Administrators) will be given this profile automatically when the account is created. At most only one profile can have this selected.

### Modifying Meter Restriction Profiles

- Click the Meter Restriction Profile in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.
- Meter Groups, Meters or Virtual Meters can be added or removed, or their access can be changed.

### Deleting a Meter Restriction Profiles

- Delete a Meter Restriction Profiles by using the action menu on the right of the table row and clicking **Delete profile**.

## IDC Restriction Profiles

A IDC Restiction Profile is a nested list of Denied/Granted IDC Groups. A few examples best illustrates the use of a restriction profiles.

** Example 1: A set of users is only granted access IDCs in a given group **

A IDC Restriction Profile can be created which has the default access set to  **Denied** which means they can not access any groups by default. A IDC group can then be added which has the access set to **Granted**. This will mean that user will only be able to access this granted group.

** Example 2: A set of users is granted access all IDCs except those in a given group **

A IDC Restriction Profile can be created which has the default access set to  **Granted** which means they can access any groups by default. A IDC group can then be added which has the access set to **Denied**. This will mean that user will be able to access all groups except this denied group.

### Viewing IDC Restriction Profiles

- Select **Admin > User admin** in the main navigation bar and then open the **IDC restriction profiles** panel to display a table of all profiles.

### Adding IDC Restriction Profiles

- Click the **Add new IDC restriction profile** button at the top right hand of the table which will display an empty settings form.
- Enter the fields according to the table below.
- To add new IDC group to the profile click the **Add profile item** button and set the access as required.
- Click **Save**.

Field | Description
------|------------ 
Name |  A string for the user to refer to the profile. It does not have any significance to the system.
Default access | Specifies whether the default profile access is **Granted** (i.e. by default a user with this profile can access everything) or **Denied** (i.e. by default a user with this profile cannot access anything)
User default | If selected new users (except Administrators) will be given this profile automatically when the account is created. At most only one profile can have this selected.

### Modifying IDC Restriction Profiles

- Click the IDC Restriction Profile in the table to select it. This will display the settings form.
- Modify any details entered previously and click the **Save** button.
- IDC Groups can be added or removed, or their access can be changed.

### Deleting a IDC Restriction Profiles

- Delete a IDC Restriction Profiles by using the action menu on the right of the table row and clicking **Delete profile**.

# Miscellaneous DCS settings

A number of other server specific settings can be modified by administrators.

## Alarm Handling

DCS may be configured to send alarms via email to a defined list of email addresses. To edit the list of email addresses:

- Select **Admin > Server admin** in the main navigation bar and then open the **Alarm handling** panel.
- Modify the recipients list. The list must consist of a semi-colon separated list of valid email addresses.
- Specify if the alarms should be sent in a condensed aggregated list, or a flat list.
- Click the **Save** button.

## IDC Updates

IDCs have the ability to automatically update their software from a DCS server.
To configure the software update feature:

- Select **Admin > Communications devices** in the main navigation bar and then open the **IDC firmware updates** panel.
- Modify the settings to control how IDC firmware updates are rolled out to IDCs and managed Modbus devices
- Click the **Save** button.

## Meter Time Adjustment

When a meter is to have its time adjusted (defined per meter), then this will only occur if the time variation is within specified limits. These limits can be modified as follows:

- Select **Admin > Metering settings** in the main navigation bar and then open the **Meter time adjustment** panel.
- Modify the minimum and maximum times.
- Click the **Save** button.

# Using the DCS Event Log

The DCS has an event log which records significant events that
occur while
the server is running. Each time the DCS is started or stopped, an entry
is made
to the event log, as well as each successful or unsuccessful meter
reading. In
addition any other error conditions are logged. There are three types
of events
in DCS: Information, Warning and Alarm. Information events are normal
events
such as successful meter readings. Warning events occur when something
which
may require user intervention has occurred, e.g. it was not possible to
initialise a modem or a communication error that has not resulted in a
missed
reading. Alarm events are serious problems such as missed readings, or
threshold violations. These may optionally trigger an email
notification (see
above).

To view the Event log, click **Event Log** on
the main navigation bar,
select the time span interest, select which types of events to view (it is possible to view
all
events, warning and alarms, or just alarms) and optionally enter a source filter click
the **Fetch** button.

# Metering Plugins

## Metered Data Plugins

### ABB B23/B24 Meter

This plugin will read registers from the ABB B23/B4 M-Bus electricity meter and store the data as half-hour data.

The following register addresses are used:

| Register      | Address |
| ------------- | ------- |
| Active Energy | 1       |
| Power         | 2       |
| Voltage L1-N  | 12      |
| Voltage L2-N  | 13      |
| Voltage L3-N  | 14      |
| Current L1    | 16      |
| Current L2    | 17      |
| Current L3    | 18      |
| Power Factor  | 19      |
| Frequency     | 20      |

Note that this meter is currently only supported in DCS via IDCs using M-bus over RS232.

Note also that this meter does not allow logged data to be collected via the IDC
and as such only the current register values are polled every half hour.
The polling is not performed at any precise time
(i.e. it is not performed precisely on the hour/half hour)
so the half-hour data is generated by DCS using linearly interpolation.

When defining a meter of this type the meter will be treated as a
serial meter connected to an IDC by RS232 and therefore the Connection Method will
be set to **IDC** and
the remote address will have the form 00:17:BF:XX:XX:XX-COMB.

To use M-Bus primary addressing the **DeviceId** must be set to the M-Bus address of the meter (if no DeviceId is set 0 will be used). This value is between 0 and 250.
To use M-Bus secondary addressing the DeviceId must be set to the 8 digit meter identity. This may or may not be the same as the meter serial number.

The Serial Communication Settings must also be set (**Initialisations string** should be empty and
**End of message (ms)** can also be left empty).

### Cewe Prometer

This plugin
will read the half-hour data from a Cewe Prometer..

The register address
defined in the Meter Type corresponds to the following:

| Register                     | Address |
| ---------------------------- | ------- |
| Active Energy (import)       | 0       |
| Active Energy (export)       | 1       |
| Reactive Energy (import)     | 2       |
| Reactive Energy (export)     | 3       |
| Reactive Energy (inductive)  | 4       |
| Reactive Energy (capacitive) | 5       |
| Reactive Energy Q1           | 6       |
| Reactive Energy Q2           | 7       |
| Reactive Energy Q3           | 8       |
| Reactive Energy Q4           | 9       |
| Apparent Energy (import)     | 10      |
| Apparent Energy (export)     | 11      |

Serial numbers for the Cewe Prometers alphanumeric strings, e.g. CW123456.

When used in a multi-drop configuration the device ID must be specified for the meter, which takes the form of the serial number of the meter, e.g. CW123456.

### Cyble Gas and Water Meter

This plugin will read registers from the Itron Cyble Meter and store the data as half-hour data.

The following register addresses are used:

| Register             | Address |
| -------------------- | ------- |
| Volume (m3)          | 1       |
| Backflow volume (m3) | 2       |

Note that this meter is currently only supported in DCS via IDCs using M-bus over RS232.

Note also that this meter does not allow logged data to be collected via the IDC
and as such only the current register values are polled every half hour.
The polling is not performed at any precise time
(i.e. it is not performed precisely on the hour/half hour)
so the half-hour data is generated by DCS using linearly interpolation.

When defining a meter of this type the meter will be treated as a
serial meter connected to an IDC by RS232 and therefore the Connection Method will
be set to **IDC** and
the remote address will have the form 00:17:BF:XX:XX:XX-COMB.

To use M-Bus primary addressing the **DeviceId** must be set to the M-Bus address of the meter (if no DeviceId is set 0 will be used). This value is between 0 and 250.
To use M-Bus secondary addressing the DeviceId must be set to the 8 digit meter identity. This may or may not be the same as the meter serial number.

The Serial Communication Settings must also be set (**Initialisations string** should be empty and
**End of message (ms)** can also be left empty).

### EDMI Atlas Meter

This plugin
will read the half-hour data from an EDMI Atlas Mk7 or Mk10 meter.

The device id for the meter **must** be specified and corresponds to the meter's serial number.

The password for the meter must also include the user name in the form "USER,PASSWORD". This field is case sensitive.

The register addresses
defined in the Meter Type corresponds to the values used to specify the channels in the EDMI meter's survey data.

| Register                       | Address                    |
| ------------------------------ | -------------------------- |
| Active Energy Import (kWh)     | 19 (corresponds to hex 13) |
| Active Energy Export (kWh)     | 35 (corresponds to hex 23) |
| Reactive Energy Import (kvarh) | 23 (corresponds to hex 17) |
| Reactive Energy Export (kvarh) | 39 (corresponds to hex 27) |
| Apparent Energy Import (kVAh)  | 27 (corresponds to hex 1B) |
| Apparent Energy Export (kVAh)  | 43 (corresponds to hex 2B) |

### Elster AS230 Meter

This plugin
will read the half-hour data from any Elster
AS230 meter. It is configured in the same way as the Elster 1700 meter.

### Elster A1140 Meter

This plugin
will read the half-hour data from any Elster
A1140 meter. It is configured in the same way as the Elster 1700 meter.

### Elster A1700 Meter

This plugin
will read the half-hour data from any Elster
A1700 meter (or ABB Vision).

The register address
defined
in the Meter Type corresponds to the following:

| Register   | Address |
| ---------- | ------- |
| kWh Import | 1       |
| kWh Export | 2       |
| kvarh Q1   | 3       |
| kvarh Q2   | 4       |
| kvarh Q3   | 5       |
| kvarh Q4   | 6       |
| kVAh       | 7       |
| Custom 1   | 8       |
| Custom 2   | 9       |
| Custom 3   | 10      |
| External 1 | 11      |
| External 2 | 12      |
| External 3 | 13      |
| External 4 | 14      |

Serial numbers for A1700 meters are up to 16 character
alphanumeric strings,
e.g. K04M01234

When used in a multi-drop configuration the device ID must be specified for the meter, which takes the form of a decimal integer.

### Elster PPM Meter

This plugin
will read the half-hour data from any Elster
PPM meter.  
The register address
defined
in the Meter Type corresponds to the following: 1=kWh Import, 2=
kWhExport, 3=kvarh Import, 4=kvarh Export.
For the Elster
PPM meter, the device ID can be any decimal integer and is
mandatory (default value is 1).

### Indigo Plus Meter

This plugin
will read the half-hour data from any
Indigo Plus (Actaris)
meter.

This following settings can be applied to a meter using this plugin:

| Name | Description                             | Values                                                                                                                  |
| ---- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| BRI  | The Flag protocol Bearer Rate Indicator | An integer which corresponds to the baud rate for the session: 0 = 300, 1 = 600, 2 = 1200, 3 = 2400, 4 = 4800, 5 = 9600 |

NB: Normally the BRI will be determined automatically based on the value sent from the meter but can be overriden here if required.

When defining a _Meter Type_ which uses this
plugin, the following needs to be taken into consideration:

This plugin reads the half hour data from an Indigo Plus
meter by reading the meter's Interval Recording Data. The
register address defined in the _Meter Type_
corresponds to the following:
0=kWh Import, 1=kWh Export, 2=kvarh&nbsp;
Import, 3=kvarh
Export, 4=kvarhQ1, 5=kvarh
Q2, 6=kvarh Q3, 7=kvarh Q4, 8=kVAh, 9=AUX1 and
A=AUX2.

Serial numbers for Indigo Plus
meters are 10
character alphanumeric strings, e.g. S01M000009

When used in a multi-drop configuration the device ID must be specified for the meter, which takes the form of a decimal integer.

### Iskra 37x Electricity Meter

This plugin
will read the half-hour data from any Iskra ME37x/MT37x meter.

The register address
defined
in the Meter Type corresponds to the following: 1=Active energy import, 3=Reactive energy import.

Passwords for Iskra meters are mandatory and must be 8 characters in length.

There are 2 modes supported for the Iskra meter and they are controlled by Manufacturer Specific Settings:

| Name | Description                                                                                                                                                                              | Values                                                                                    |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Mode | Specifies the protocol mode used to interrogate the meter. Flag mode is used when connecting to the meter via the optical port otherwise DLMS is used (e.g. when connecting via a modem) | _Flag_ or _DLMS_. Note that Flag is default and is assumed if this settings doesn't exist |

When used in a multi-drop configuration the device ID must be specified for the meter, which takes the form of a decimal integer. This is only applicable to Flag mode.

### Itron CF Heat Meter

This plugin will read registers from the Itron CF family of heat meters and store the data as half-hour data.

The following register addresses are used:

| Register                   | Address |
| -------------------------- | ------- |
| Energy (kWh)               | 1       |
| Power (kW)                 | 2       |
| Flow temperature (C)       | 3       |
| Return temperature (C)     | 4       |
| Temperature difference (C) | 5       |
| Volume (m3)                | 6       |
| Flow rate (m3/hr)          | 7       |

Note that this meter is currently only supported in DCS via IDCs using M-bus over RS232.

Note also that this meter does not allow logged data to be collected via the IDC and as such only the current register values are polled every half hour. The polling is not performed at any precise time (i.e. it is not performed precisely on the hour/half hour) so the half-hour data is generated by DCS using linearly interpolation.

When defining a meter of this type the meter will be treated as a serial meter connected to an IDC by RS232 and therefore the Connection Method will be set to **IDC** and the remote address will have the form 00:17:BF:XX:XX:XX-COMB.

To use M-Bus primary addressing the **DeviceId** must be set to the M-Bus address of the meter (if no DeviceId is set 0 will be used). This value is between 0 and 250.
To use M-Bus secondary addressing the DeviceId must be set to the 8 digit meter identity. This may or may not be the same as the meter serial number.

The Serial Communication Settings must also be set (**Initialisations string** should be empty and **End of message (ms)** can also be left empty).

### Katflow 100 M-Bus Meter

This plugin will read registers from the Katronics Katflow 100 M-Bus meter and store the data as half-hour data.

The following register addresses are used:

| Register           | Address |
| ------------------ | ------- |
| Volume             | 6       |
| Flow rate          | 7       |
| Energy             | 1       |
| Flow temperature   | 3       |
| Return temperature | 4       |
| Power              | 2       |
| Signal stength     | 7       |

Note that this meter is currently only supported in DCS via IDCs using M-bus over RS232.

Note also that this meter does not allow logged data to be collected via the IDC
and as such only the current register values are polled every half hour.
The polling is not performed at any precise time
(i.e. it is not performed precisely on the hour/half hour)
so the half-hour data is generated by DCS using linearly interpolation.

When defining a meter of this type the meter will be treated as a
serial meter connected to an IDC by RS232 and therefore the Connection Method will
be set to **IDC** and
the remote address will have the form 00:17:BF:XX:XX:XX-COMB.

To use M-Bus primary addressing the **DeviceId** must be set to the M-Bus address of the meter (if no DeviceId is set 0 will be used). This value is between 0 and 250.
To use M-Bus secondary addressing the DeviceId must be set to the 8 digit meter identity. This may or may not be the same as the meter serial number.

The Serial Communication Settings must also be set (**Initialisations string** should be empty and
**End of message (ms)** can also be left empty).

### Kamstrup Multical Meter

This plugin will read registers from the Kamstrup Multical 602/801 meter and store the data as half-hour data.

Note that this meter is currently only supported in DCS via GPRS (i.e. TCP/IP).
Ensure that the TCP port in the meter's remote address is set to 1025.

Note also that this meter does not allow logged data to be collected via GPRS and as such only the current register values are polled
and the half-hour data is generated by DCS using linearly interpolation.

In the normal case the meters are polled every half hour (but not at any precise time and hence the values at the hour or half hour are interpolated).
In the case where the cost of data transmission is more important than the resolution of the data it is possible to allow the meter to be polled once per day.
This is done by setting the Collection Schedule for the meter (i.e. if the Collection Schedule is "None" the meter will be polled every half hour, if it is set then the meter is polled once per day according to that schedule. )

### Landis and Gyr T550 Heat Meter

This plugin will read registers from the Landis and Gyr T550 Heat meter and store the data as half-hour data.

The following register addresses are used:

| Register                   | Address |
| -------------------------- | ------- |
| Energy (kWh)               | 1       |
| Power (kW)                 | 2       |
| Flow temperature (C)       | 3       |
| Return temperature (C)     | 4       |
| Temperature difference (C) | 5       |
| Volume (m3)                | 6       |
| Flow rate (m3/hr)          | 7       |
| Cooling energy (kWh)       | 9       |
| Heating energy (kWh)       | 10      |

Note that this meter is currently only supported in DCS via IDCs using M-bus over RS232.

Note also that this meter does not allow logged data to be collected via the IDC and as such only the current register values are polled every half hour. The polling is not performed at any precise time (i.e. it is not performed precisely on the hour/half hour) so the half-hour data is generated by DCS using linearly interpolation.

When defining a meter of this type the meter will be treated as a serial meter connected to an IDC by RS232 and therefore the Connection Method will be set to **IDC** and the remote address will have the form 00:17:BF:XX:XX:XX-COMB.

To use M-Bus primary addressing the **DeviceId** must be set to the M-Bus address of the meter (if no DeviceId is set 0 will be used). This value is between 0 and 250.
To use M-Bus secondary addressing the DeviceId must be set to the 8 digit meter identity. This may or may not be the same as the meter serial number.

The Serial Communication Settings must also be set (**Initialisations string** should be empty and **End of message (ms)** can also be left empty).

### PRI Premier Meter PAKNET

This plugin
will read the half-hour data from any PRI Premier meter using the
PAKNET protocol.

If a password is specified in this field, it will be used as the meter's
Level 1 password. If another level of password is required
this can be specified using the settings field below.

This plugin may be configured with the following
settings:

| Name    | Description                                                                                                                                                                                                                                                                                                                                                                                                  | Values                                                                                                                                                                                                                                                  |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GDCount | The number of times a Global Disable message is sent out on a multidrop connection (a connection is assumed to be multidrop if a Device Id is defined for the meter). If this setting is not defined the Global Disable message will be sent once. It is only recommended to set this to a higher value if it is not possible to reliably get a response to a Specific Enable message on a particular chain. | An integer between 1 and 5. The default value is 1                                                                                                                                                                                                      |
| Pwdn    | Level n password for the meter.                                                                                                                                                                                                                                                                                                                                                                              | An integer corresponding to the level n password where n is 0..4                                                                                                                                                                                        |
| KeyRef  | The Key Reference for password encryption.                                                                                                                                                                                                                                                                                                                                                                   | A four digit integer corresponding to the key reference to the High Key                                                                                                                                                                                 |
| LowKey  | The Low Key for password encryption.                                                                                                                                                                                                                                                                                                                                                                         | Up to an 8 digit integer corresponding to the Low key                                                                                                                                                                                                   |
| HighKey | The High Key magic number                                                                                                                                                                                                                                                                                                                                                                                    | A 48 character string corresponding to the High Key hexadecimal value. Note that this is usually configured as part of the DCS installation and is not required. It can be entered here if a particular high key has not been configured for the system |

The
register address defined in the Meter Type corresponds to the decimal
value of the PACT Energy Type e.g. Active energy,
import fundamental (kWh) will have an address of 130.

Serial numbers for PRI meters are 8 character alphanumeric
strings, e.g.
R0400219.

When used in a multi-drop configuration the device ID must be specified for the meter, which is the same as the meter's serial number.

When using the PAKNET protocol over a modem connection it is recommended that the End of message timeout in the Serial Communications Settings is set to 50ms.

### PRI Premier Meter

This plugin
will read the half-hour data from any PRI
PACT meter using the PACT protocol.

All other details are the same as for the PAKNET version of the plugin.

> The [PRI Premier Meter PAKNET](#metering-plugins-metered-data-plugins-pri-premier-meter-paknet) plugin is normally recommended instead of this.

### PRI Liberty Meter

This plugin
will read the half-hour data from any PRI Liberty Meter.

The register address defined in the Meter Type corresponds to the decimal
value of the PACT Energy Type. Register totals aren't supported by these meters
but this plugin will read the instantaneous register value once per month and store
this as a Calibration Reading that can be used to estimate register totals.
Note that the only register for which this functionality is supported is the
Active energy (import + export) register (energy type 132).

These meters do not support passwords and any entered password will be ignored.
As a consequence, time adjustment functionality is not supported for these meters.

Serial numbers for PRI Liberty meters are 8 character alphanumeric
strings, e.g. DAL00067. When used in a multi-drop configuration the device ID must be
specified for the meter, which is the same as the meter's serial number.

### Relay PadPuls M1 M-Bus Pulse Counter

This plugin will read registers from the
Relay PadPuls M1 M-Bus Pulse Counter and store the data as half-hour data.

The pulse counter has a single register that corresponds to the 48 bit counter value.
The address used for the counter is not important
(although by convention can be set to 1).
When reading the pulse counter any units defined in the unit
will be ignored and the units specified in the Meter Type will be used.
Note that the pulse counter has an inbuilt scaling factor that is read by DCS and applied to the counter value.
If a scaling factor is specified in the Meter
Type this will also be applied.

Note that this meter is currently only supported in DCS via IDCs using M-bus over RS232.

Note also that this meter does not allow logged data to be collected via the IDC
and as such only the current register values are polled every half hour.
The polling is not performed at any precise time
(i.e. it is not performed precisely on the hour/half hour)
so the half-hour data is generated by DCS using linearly interpolation.

When defining a meter of this type the meter will be treated as a
serial meter connected to an IDC by RS232 and therefore the Connection Method will
be set to **IDC** and
the remote address will have the form 00:17:BF:XX:XX:XX-COMB.

To use M-Bus primary addressing the **DeviceId** must be set to the M-Bus address of the meter (if no DeviceId is set 0 will be used). This value is between 0 and 250.
To use M-Bus secondary addressing the DeviceId must be set to the 8 digit meter identity. This may or may not be the same as the meter serial number.

The Serial Communication Settings must also be set (**Initialisations string** should be empty and
**End of message (ms)** can also be left empty).

### Relay PadPuls M2 M-Bus Pulse Counter

This plugin is similar to the Relay PadPuls M1 M-Bus Pulse Counter plugin except that it supports 2 32 bit pulse
inputs which are treated as separate meters.

Each meter should use M-Bus secondary addressing and the DeviceId must be set to the 8 digit meter identity.
This will consist of XXXXXX01 for the first pulse input and XXXXXX02 for the seconds where XXXXXX is the meter's serial number.
In all other respects the meters should be defined according to the Relay PadPuls M1 M-Bus Pulse Counter plugin.

### Sensus HRI M-Bus Sensor

This plugin will read registers from the Sensus HRI M-Bus sensor and store the data as half-hour data.

The following register addresses are used:

| Register    | Address |
| ----------- | ------- |
| Volume (m3) | 1       |

Note that this meter is currently only supported in DCS via IDCs using M-bus over RS232.

Note also that this meter does not allow logged data to be collected via the IDC
and as such only the current register values are polled every half hour.
The polling is not performed at any precise time
(i.e. it is not performed precisely on the hour/half hour)
so the half-hour data is generated by DCS using linearly interpolation.

When defining a meter of this type the meter will be treated as a
serial meter connected to an IDC by RS232 and therefore the Connection Method will
be set to **IDC** and
the remote address will have the form 00:17:BF:XX:XX:XX-COMB.

To use M-Bus primary addressing the **DeviceId** must be set to the M-Bus address of the meter (if no DeviceId is set 0 will be used). This value is between 0 and 250.
To use M-Bus secondary addressing the DeviceId must be set to the 8 digit meter identity. This may or may not be the same as the meter serial number.

The Serial Communication Settings must also be set (**Initialisations string** should be empty and
**End of message (ms)** can also be left empty).

## Real Time Data Plugins

### Cyble Gas and Water Meter

This plugin can be used for Itron Cyble meters.
It will read the current values for volume and backflow volume.

Note that this meter is only supported via IDCs. See the metered data plugin for more details.

### Elster A1700

This plugin can be used for Elster A1700 meters. It will read the current values for voltage, current, power, power factor and phase angle for each phase.

### Itron CF Heat Meter

This plugin will read registers from the Itron CF family of heat meters. It will read the current values for energy, power, temperatures, volume and flow rate.

Note that this meter is only supported via IDCs. See the metered data plugin for more details.

### Kamstrup Multical

This plugin can be used for Kamstrup Multical 602/801 Heat meters. It will read the current values for temperature, flow rate and power.
Note that this meter is currently only supported in DCS via GPRS (i.e. TCP/IP). Ensure that the TCP port in the meter's remote address is set to 1025.

### Landis and Gyr T550 Heat Meter

This plugin can be used for Landis and Gyr T550 Heat meters. It will read the current values for energy, power, temperatures, volume and flow rate.

Note that this meter is only supported via IDCs. See the metered data plugin for more details.

### Relay PadPuls M1 M-Bus Pulse Counter

This plugin can be used for Relay PadPuls M1 M-Bus Pulse Counter meters.
It will read the current value for of the pulse input counter.

Note that this meter is only supported via IDCs. See the metered data plugin for more details.

### Relay PadPuls M2 M-Bus Pulse Counter

This plugin can be used for Relay PadPuls M2 M-Bus Pulse Counter meters.
It will read the current value for of the pulse input counter.

Note that this meter is only supported via IDCs. See the metered data plugin for more details.

### Sensus HRI M-Bus Sensor

This plugin can be used for meters using the Sensus HRI M-Bus sensor.
It will read the current value for volume.

Note that this meter is only supported via IDCs. See the metered data plugin for more details.

# Data Import/Export Plugins

## Data Export Plugins

### Default XML

The readings will be downloaded in the default Coherent XML format.

### CSV

The readings will be downloaded in the default Coherent CSV format.

### CSV (by half hour period)

The readings will be downloaded in a special CSV format where each day corresponds to a row and each column corresponds to a period.

## Data Import Plugins

### Metered Data Import

#### CSV Format

The data must be imported from a CSV file with one of the two following formats:

**Format 1**

This corresponds to the default CSV format used when downloading metered data from DCS and therefore has some redundant information which can be left blank.

| Column       | Description                                                                  |
| ------------ | ---------------------------------------------------------------------------- |
| Meter        | The ID of the meter or virtual meter.                                        |
| Name         | The name of the meter. This will be ignored during import.                   |
| SerialNumber | The serial number defined for the meter. This will be ignored during import. |
| Register     | The name of the register.                                                    |
| Date         | The date of the reading (format dd/MM/yyyy).                                 |
| StartTime    | The start time of the reading period (UTC) (format HH:mm:ss).                |
| Duration     | The duration (in minutes) of the reading period.                             |
| PeriodValue  | The reading value for the period.                                            |
| TotalValue   | The total value at the beginning of the period.                              |

**Format 2**

This is a simpler format when creating the data from scratch.

| Column      | Description                                                   |
| ----------- | ------------------------------------------------------------- |
| RegisterId  | The ID of the register.                                       |
| Date        | The date of the reading (format dd/MM/yyyy).                  |
| Time        | The start time of the reading period (UTC) (format HH:mm:ss). |
| Duration    | The duration (in minutes) of the reading period.              |
| PeriodValue | The reading value for the period.                             |
| TotalValue  | The total value at the beginning of the period.               |

### Manual Readings Data Import

#### CSV Format

The data must be imported from a CSV file. The format is as follows:

| Column   | Description                                                                                                                                                                                                                                                                |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Date     | The date of the reading (format dd/MM/yyyy).                                                                                                                                                                                                                               |
| Time     | The time of the manual reading period (UTC) (format HH:mm:ss). This column is optional and can be omitted entirely in which case the time is set to midnight. If the column is included but the value is blank for a particular row this will also be treated as midnight. |
| Register | The register id for which the manual reading applies.                                                                                                                                                                                                                      |
| Reading  | The manual meter reading.                                                                                                                                                                                                                                                  |

#### CSV Horizontal Format

The data must be imported from a CSV file. The format is as follows:

| Column   | Description                                                                                                                                                                                                                                                                                              |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Date     | The date of the reading (format dd/MM/yyyy).                                                                                                                                                                                                                                                             |
| Time     | The time of the manual reading period (UTC) (format HH:mm:ss). This column is optional and can be omitted entirely inwhich case the time is set to midnight. If the column is included but the value is blank for a particular row this will also be treated as midnight.                                |
| Rx or Mx | The register id (e.g. R10) or meter id (e.g. M200) for which the manual reading applies. If a meter id is provided then the reading will be assumed to belong to the 1st register in the meter. Any number of these columns may be included. The actual value of the column contains the meter readings. |

#### System Link CSV Format

The data must be imported from a CSV file. The format is as follows:

| Column     | Description                                  |
| ---------- | -------------------------------------------- |
| Code       | Meter Name.                                  |
| Date       | The date of the reading (format dd/MM/yyyy). |
| M1_Present | The manual meter reading.                    |

# Report Types

## Basic Billing Report

The Basic Billing Report will send an email with the costs associated with any number of
Registers and/or Virtual Meters for a day, week or month. The report will show the total cost as well as a break down by Register/Virtual Meter.

## CSV Readings

The CSV Readings Report will send an email with the readings for all included Groups, Registers and Virtual Meters.
If a Group is included all Registers and Virtual Meters in the Group will also be included.

Note that all Registers in Meters which have Data Collection Disabled set will NOT be included in the report.

The readings will be contained in an email attachment in a form that can be opened with a program such as Microsoft Excel
to process the data.

The CSV file contain one row for every reading and the following columns:

| Column       | Description                                                                                                       |
| ------------ | ----------------------------------------------------------------------------------------------------------------- |
| Meter        | The ID of the meter or virtual meter.                                                                             |
| Name         | The name of the meter or virtual meter.                                                                           |
| SerialNumber | The serial number defined for the meter (if available). If the reading is for a virtual meter this will be empty. |
| Register     | The name of the register. If the reading is for a virtual meter this will be empty.                               |
| Date         | The date of the reading.                                                                                          |
| StartTime    | The start time of the reading period (UTC).                                                                       |
| Duration     | The duration (in minutes) of the reading period.                                                                  |
| PeriodValue  | The reading value for the period.                                                                                 |
| TotalValue   | The total value at the beginning of the period. If the reading is for a virtual meter this will be 0.             |

## CSV Readings by Period

The CSV Readings By Period Report will send an email with the readings as for the CSV Readings report with the difference
that instead of one row per metering period there will be one
row per day and the readings for each metering period will be in individual columns.

## Data Collection

The Data Collection Report will send an email with statistics and about the metered data that has been collected by DCS over the preceding days
for all included Groups, Meters and Registers. The report can be used to ensure that data is being collected reliably. If a Group is included all Meters in the Group and all Registers in the each Meter will be included. If a Meter is included all Registers in the Meter will be included.

Note that the report frequency for this report can be Daily or Weekly. In the case of a Daily report the Data Collection for the previous 10 days is shown. In the case of a Weekly report the last 20 days are shown.

Note that Virtual Meters are ignored by this report.

Note that all Meters which have Data Collection Disabled set will NOT be included in the report.

## Data Collection Exception

The Data Collection Exception Report will send an email with a list of meters that have missing readings above a system configurable limit for the time span of the report
for all included Groups. The report can be used to ensure that data is being collected reliably. If a Group is included all Meters in the Group will be included. When subscribing to this report the user can choose to only receive the report when there are meters with missing readings by selecting "Only send on alarm condition".

Note that the report frequency for this report can be Daily, Weekly or Monthly.

Note that all Meters which have Data Collection Disabled set will NOT be included in the report.

## Daily Limits Report

The Daily Limit Report will calculate the daily totals for all specified registers and virtual meters and compare them against specified minimum and maximum limits.
The results will be sent by email.

The input for the Daily Limit Report must be specified in a Microsoft Office 2007 (or later) Excel file which can be uploaded when editing a subscription settings. The file must be of type "xlsx" and
have the following format:

| Column     | Position | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ---------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ID         | A1       | The ID of the register or virtual meter in the form Rx or VMx where x is an integer, e.g. to specify the register with ID 99 this column would contain R99, to specify the virtual meter with ID 27 this column would contain VM27.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| LowerLimit | B1       | A numerical value representing the lower limit that the daily total must be equal to or greater than for the specified register or virtual meter. This column may be left empty, in which case there is no lower limit.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| UpperLimit | C1       | A numerical value representing the upper limit that the daily total must be equal to or less than for the specified register or virtual meter. This column may be left empty, in which case there is no upper limit.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| DoW Filter | D1       | Optional column. If the Day of Week Filter exists the daily total is calculated only for days that correspond to the filter. The filter format consist of a 7 character string with each position representing a day of the week starting with Monday. To include a day of the week the corresponding position must contain the first character of the day, to exclude a day of the week the corresponding position must contain a character other than the first character of the day. E.g. 'xTWxxxx' would include Tuesday and Wednesday, '.TWTF..' would include Tuesday, Wednesday, Thursday and Friday, etc. Special "shorthand strings" Weekdays, Weekends exist. A non-existent or blank DoW Filter means all days of the week. Beware if using '.' as the "don't include character" as Excel may convert 3 dots into an ellipsis which will give a format error when processing the file. |
| ToD Filter | E1       | Optional column. If the Time of Day Filter exists only readings that correspond to the filter will be included in the daily total. The filter format consists of a comma separated list of included time ranges in the format HH:mm-HH:mm. The start of the day is 00:00 and end of the day is 00:00. E.g. '09:00-17:00' means 9am to 5pm, and '00:00-09:00, 17:00-00:00' means all times except 9am to 5pm. Note that resolution is 30 minutes so time ranges like 09:45-10:51 are not supported.                                                                                                                                                                                                                                                                                                                                                                                                |

> A CSV file (with type "csv") can also be used to specify the input for this report. If using the ToD Filter in a CSV file a semicolon separated list of included time ranges should be used instead of a comma separated list.

An simple example would be:

|     | A    | B          | C          |
| --- | ---- | ---------- | ---------- |
| 1   | ID   | LowerLimit | UpperLimit |
| 2   | R10  | 100        | 1000       |
| 3   | VM10 | 100        |            |
| 4   | VM12 |            | 999        |

An example with Day of Week and Time of Day filters would be:

|     | A    | B          | C          | D         | E                        |
| --- | ---- | ---------- | ---------- | --------- | ------------------------ |
| 1   | ID   | LowerLimit | UpperLimit | DoWFilter | ToDFilter                |
| 2   | R10  | 100        | 1000       | MTWTFxx   |                          |
| 3   | VM10 | 100        | 2000       | MTWTFxx   | 00:00-09:00, 17:00-00:00 |
| 4   | VM10 | 100        | 500        | MTWTFxx   | 09:00-17:00              |
| 5   | VM10 | 100        | 100        | xxxxxSS   | 00:00-09:00, 17:00-00:00 |
| 6   | VM10 | 100        | 500        | xxxxxSS   | 09:00-17:00              |
| 7   | VM12 | 10         | 999        | WEEKENDS  |
| 8   | VM12 | 10         | 999        | WEEKDAYS  | 00:00-09:00, 17:00-00:00 |

## Daily Usage Report

The Daily Usage Report will show the daily total usage (i.e. the sum of all period values) for any included registers.
The report can be run daily, weekly or monthly and it will included daily usage totals for each day in the reporting period.

Note that Meters, Virtual Meters and Groups are ignored by this report.

## End of Period Totals Report

The End of Period Totals Report produces a table of total register values at the end of the reporting period for each included register.
The supported reporting periods for this meter are Daily, Weekly and Monthly.

Note that Meters, Virtual Meters and Groups are ignored by this report.

## Excel Template Report

The Excel Template Report is a generic report that takes an Excel spreadsheet as input and fills designated cells metered data for a specified set of registers and/or virtual meters.

The input for the report must be specified in a Microsoft Office 2007 (or later) Excel file which can be uploaded when editing a report. The file must be of type "xlsx" and contain a sheet called "DCS" that defines the input for the report and where in the spreadsheet the output should be stored.

When the report is generated DCS will user the "DCS" configuration sheet to populate a copy of the spreadsheet with metered data. The resulting spreadsheet will then be included in the generated report.

Each row in the "DCS" sheet describes a source (a DCS register or virtual meter) that will be included in the report, a time range for readings and where in the spreadsheet the readings should be stored.

The format of the DCS sheet in the spreadsheet is as follows:

| Column            | Position | Description                                                                                                                                                                                                                                             |
| ----------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Source            | A1       | The ID of the register or virtual meter in the form Rx/VMx where x is an integer, e.g. to specify the register with ID 99 this column would contain R99. This field is mandatory.                                                                       |
| StartDate         | B1       | The start date for metered data relative to when the report is generated (see format below). Note that 2 and only 2 of the fields _StartDate_, _EndDate_ and _Span_ must be present.                                                                    |
| Span              | C1       | The date span from the start date metered data (see format below). Note that 2 and only 2 of the fields _StartDate_, _EndDate_ and _Span_ must be present.                                                                                              |
| EndDate           | D1       | The end date for metered data relative to when the report is generated (see format below). Note that 2 and only 2 of the fields _StartDate_, _EndDate_ and _Span_ must be present.                                                                      |
| IntegrationPeriod | E1       | The integration period for the metered data. Allowed values: HalfHour, Hour, Day, Week, Month. If omitted the value will be HalfHour.                                                                                                                   |
| UseLocal          | F1       | Specifies whether local time will be used for the metered data, if not GMT will be used. Allowed values: Yes, No. If omitted the value will be No.                                                                                                      |
| Calibrated        | G1       | Specifies whether the metered data will be calibrated. Allowed values: Yes, No. If omitted the value will be No.                                                                                                                                        |
| Interpolated      | H1       | Specifies whether the metered data will be interpolated. Allowed values: Yes, No. If omitted the value will be No.                                                                                                                                      |
| Format            | I1       | The format the metered data will be written in. Allowed values: List, Wide (see below for details). If omitted the value will be List. Note that Wide is only supported when the _IntergrationPeriod_ is HalfHour.                                      |
| Sheet             | J1       | The Excel sheet where the metered data will be written. If omitted a sheet with the same name as _Source_ will be created.                                                                                                                              |
| CellRef           | K1       | The cell where the metered data will be written. If omitted for the 1st source in a given sheet A1 will be assumed. If omitted in a subsequent source for a sheet the data will written in the row immediately below the previous data without headers. |

NB: The order of the columns is actually not significant as the column heading will be used to find the correct column.
If a column has an unrecognised heading then it will be ignored, therefore it possible to put descriptive columns such as "Comments" for users. Note also that columns names are NOT case sensitive. Columns for optional fields can be omitted completely and the default values will be used for all rows.

The syntax for _StartDate_ and _EndDate_ is as follows:

```text
{ THIS | LAST } { DAY | WEEK | MONTH | YEAR } [ { + | - } N { DAY[S] | WEEK[S} | MONTH[S] | YEAR[S] ]*

OR an absolute date may be specified.

* the [ { + | - } N { DAY[S] | WEEK[S} | MONTH[S] | YEAR[S] ] part may be repeated any number of times.

THIS: the beginning or end (inclusive) of the current Day, Week, Month or Year
LAST: the beginning or end (inclusive) of the previous Day, Week, Month or Year

Aliases can be used for some common constructs:

TODAY: equivalent to THIS DAY
YESTERDAY: equivalent to PREVIOUS DAY
```

The syntax for _Span_ is as follows:

```text
N { DAY[S] | WEEK[S] | MONTH[S} | YEAR[S] }
```

Examples of using StartDate, EndDate and Span:

|   | A | B | C | Meaning 
|---|---|---|---|---
| 1 | StartDate | Span | EndDate | Corresponding date range
| 2 | YESTERDAY | 1 DAY | | Yesterday
| 3 | LAST WEEK | 7 DAYS | | Previous week from Monday to Sunday inclusive
| 4 | LAST WEEK + 2 DAYS | 3 DAYS | | Previous week Wednesday, Thursday and Friday only
| 5 | THIS YEAR | | YESTERDAY | From the 1st of January until the end of yesterday
| 6 | LAST YEAR + 7 MONTHS + 10 DAYS | | YESTERDAY | From the 11th of August last year until the end of yesterday
| 7 |  | 21 DAYS | YESTERDAY | The last 21 days up to the end of yesterday
| 8 | 01/01/2001 | | THIS YEAR | From the 1st of January 2001 until the end of the current year

The *List* format of the metered data will be as follows:

Column | Description        
-------|------------
Source | The source from the configuration sheet.
StartTime | The start time of the reading period.
Duration | The duration (in minutes) of the reading period.
PeriodValue | The reading value for the period.
TotalValue | The total value at the beginning of the period. If the reading is for a virtual meter this will be 0.

The *Wide* format of the metered data will be as follows:

Column | Description        
-------|------------
Source | The source from the configuration sheet.
Date | The date for all readings in the row.
Total | The total value at the beginning of the day. If the reading is for a virtual meter this will be 0.
Time of day | A separate column for each period in the form HH:mm, i.e. columnds 00:00, 00:30, 01:00 ... 23:30        

## Generation Reading Charts Report

The Generation Reading Charts Report is designed to display reading charts for generator.
The report will display reading charts for each specified register and virtual meter
and display a line representing its rating.

The input for the Generation Reading Charts Report must be specified in a CSV file or a Microsoft Office 2007 (or later) Excel file which can be uploaded when editing a subscription settings.
The file must be of type "csv" or "xlsx" and
have the following format:

| Column    | Position | Description                                                                                                                                                                                                                         |
| --------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ID        | A1       | The ID of the register or virtual meter in the form Rx or VMx where x is an integer, e.g. to specify the register with ID 99 this column would contain R99, to specify the virtual meter with ID 27 this column would contain VM27. |
| MaxOutput | B1       | A numerical value representing the maximum output (or rating) in kW (assuming the unit is kWh)                                                                                                                                      |

An example would be:

|     | A    | B         |
| --- | ---- | --------- |
| 1   | ID   | MaxOutput |
| 2   | R10  | 100       |
| 3   | VM10 | 100       |
| 4   | VM12 | 250       |

## Historical Average Comparison Report

The Historical Average Report compares the average daily consumption for a current date window with that of an historical date windows and highlights if the difference is within specified limits.

The input for the report must be specified in a Microsoft Office 2007 (or later) Excel file which can be uploaded when editing a report. The file must be of type "xlsx" and have the following format:

| Column          | Position | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| --------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DcsId           | A1       | The ID of the register or virtual meter in the form Rx/VMx where x is an integer, e.g. to specify the register with ID 99 this column would contain R99.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Current Window  | B1       | Date window size in units of the report frequency (i.e. Days for daily report, Weeks for weekly report). Current refers to the most recent window, i.e. the subject of the report.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Previous Window | C1       | Date window size in units of the report frequency (i.e. Days for daily report, Weeks for weekly report). Previous refers to the window that the current window will be compared against.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| DoW Filter      | D1       | Optional column. The Day of Week Filter filters out days from each window that don't correspond. The filter format consist of a 7 character string with each position representing a day of the week starting with Monday. To include a day of the week the corresponding position must contain the first character of the day, to exclude a day of the week the corresponding position must contain a character other than the first character of the day. E.g. 'xTWxxxx' would be every Tuesday and Wednesday, '.TWTF..' would be every Tuesday, Wednesday, Thursday and Friday, etc. Special "shorthand strings" Weekdays, Weekends exist. A non-existent or blank DoW Filter means all days of the week. Beware if using '.' as the "don't include character" as Excel may convert 3 dots into an ellipsis which will give a format error when processing the file. |
| ToD Filter      | E1       | Optional column. The Time of Day Filter filters out hours in the day from each window that don't match. The filter format consists of a comma separated list of included time ranges in the format HH:mm-HH:mm. The start of the day is 00:00 and end of the day is 00:00. E.g. '09:00-17:00' means 9am to 5pm, and '00:00-09:00, 17:00-00:00' means all times except 9am to 5pm. Note that resolution is 30 minutes so time ranges like 09:45-10:51 are not supported.                                                                                                                                                                                                                                                                                                                                                                                                 |
| Offset          | F1       | The gap between the current and previous windows in units of the report frequency..                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Lower Threshold | G1       | The maximum amount that the current average is expected to be below the previous average. The threshold may be an absolution number or a percentage.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Upper Threshold | H1       | The maximum amount that the current average is expected to be above the previous average. The threshold may be an absolution number or a percentage.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

NB: The order of the columns is actually not significant as the column heading will be used to find the correct column.
If a column has an unrecognised heading then it will be ignored, therefore it possible to put descriptive columns such as "Comments" for users. Note also that columns names are NOT case sensitive.

An example to compare the daily consumption for a series of register with the daily average for the previous 10 days with various thresholds.

|     | A     | B              | C               | D      | E               | F               |
| --- | ----- | -------------- | --------------- | ------ | --------------- | --------------- |
| 1   | DcsId | Current Window | Previous Window | Offset | Lower Threshold | Upper Threshold |
| 2   | R99   | 1              | 10              | 0      |                 | 10%             |
| 3   | R201  | 1              | 10              | 0      | 5%              | 20%             |
| 4   | R2201 | 1              | 10              | 0      | 5%              | 20%             |

An example to compare the average daily consumption for the week for a series of register with the daily average for 4 weeks 24 weeks ago with various threholds. In these cases only certain days of the week and time ranges are of interest.

|     | A     | B              | C               | D      | E          | F           | G               | H               | I                                                        |
| --- | ----- | -------------- | --------------- | ------ | ---------- | ----------- | --------------- | --------------- | -------------------------------------------------------- |
| 1   | DcsId | Current Window | Previous Window | Offset | DoW Filter | ToD Filter  | Lower Threshold | Upper Threshold | Notes                                                    |
| 2   | R99   | 1              | 4               | 24     | Weekdays   | 09:00-17:00 |                 | 10%             | 9 to 5 weekdays only (don't care about lower threshold). |
| 3   | R201  | 1              | 4               | 24     | xTxTxxx    | 12:00-17:00 | 5%              | 20%             | Only interested in Tuesday and Thursday afternoons.      |
| 4   | R2201 | 1              | 4               | 24     |            |             | 5%              | 20%             | All day every day                                        |

## IDC Status Report

The IDC Status report will list all IDCs that have gone offline during the reporting period.
If the user subscribes to this report it will only be emailed if an
IDC goes offline during the reporting period.

## Readings Chart Report

The Readings Chart Report will send an email with the readings for all included Groups, Registers and Virtual Meters in chart form.
If a Group is included all Registers and Virtual Meters in the Group will also be included.

Note that all Registers in Meters which have Data Collection Disabled set will NOT be included in the report.

## Total Threshold Report

The Total Threshold Report compares the current total for a register with a specified maximum value.
If the value is above a specified alarm threshold this is highlighted in the report and the report is emailed to any subscribers.

The input for the report must be specified in a CSV file or Microsoft Office 2007 (or later) Excel file which can be uploaded when editing a subscription settings. The file must be of type "csv" or "xlsx" and
have the following format:

| Column         | Position | Description                                                                                                                         |
| -------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| ID             | A1       | The ID of the register in the form Rx where x is an integer, e.g. to specify the register with ID 99 this column would contain R99. |
| UpperLimit     | B1       | A numerical value representing the upper limit that the total must be equal to or less than.                                        |
| AlarmThreshold | C1       | A numerical value representing the threshold (as a percentage of the UpperLimit) at which the total will generate an alarm.         |

An example would be:

|     | A    | B          | C          |
| --- | ---- | ---------- | ---------- |
| 1   | ID   | LowerLimit | UpperLimit |
| 2   | R10  | 100000     | 80         |
| 3   | R110 | 1000       | 70         |

If the user subscribes to this report it will only be emailed if one or more totals have reached the alarm threshold.
