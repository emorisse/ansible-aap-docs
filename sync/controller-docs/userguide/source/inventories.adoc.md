== Inventories

An `Inventory` is a collection of hosts against which jobs may be
launched, the same as an Ansible inventory file. Inventories are divided
into groups and these groups contain the actual hosts. Groups may be
sourced manually, by entering host names into the automation controller,
or from one of its supported cloud providers.

Note

If you have a custom dynamic inventory script, or a cloud provider that
is not yet supported natively in the controller, you can also import
that into the controller. Refer to {ag_inv_import} in the Automation
Controller Administration Guide.

The Inventories window displays a list of the inventories that are
currently available. The inventory list may be sorted by name and
searched type, organization, description, owners and modifiers of the
inventory, or additional criteria as needed.

image:inventories-home-with-examples.png[Inventories
- home with examples]

The list of Inventory details includes:

* *Name*: The inventory name. Clicking the Inventory name navigates to
the properties screen for the selected inventory, which shows the
inventory's groups and hosts. (This view is also accessible from the
image:edit-button.png[edit button] icon.)
* *Status*

The statuses are:

* *Success*: when the inventory source sync completed successfully
* *Disabled*: no inventory source added to the inventory
* *Error*: when the inventory source sync completed with error

An example of inventories of various states, including one with detail
for a disabled state:

image:inventories-home-with-status.png[image]

* *Type*: Identifies whether it is a standard inventory or a Smart
Inventory.
* *Organization*: The organization to which the inventory belongs.
* *Actions*: The following actions are available for the selected
inventory:
+
_________________________________________________________________________________________________________________________
** *Edit* (image:edit-button.png[edit
button]): Edit the properties for the selected inventory
** *Copy* (): Makes a copy of an existing inventory as a template for
creating a new one
_________________________________________________________________________________________________________________________

include::work_items_deletion_warning.adoc[]

[[ug_inventories_smart]]
=== Smart Inventories

A Smart Inventory is a collection of hosts defined by a stored search
that can be viewed like a standard inventory and made to be easily used
with job runs. Organization administrators have admin permission to
inventories in their organization and can create a Smart Inventories. A
Smart Inventory is identified by `KIND=smart`. You can define a Smart
Inventory using the same method being used with Search.
`InventorySource` is directly associated with an Inventory.

The `Inventory` model has the following new fields that are blank by
default but are set accordingly for Smart Inventories:

* `kind` is set to `smart` for Smart Inventories
* `host_filter` is set AND `kind` is set to `smart` for Smart
Inventories.

The `host` model has a related endpoint, `smart_inventories` that
identifies a set of all the Smart Inventory a host is associated with.
The membership table is updated every time a job runs against a smart
inventory.

Note

To update the memberships more frequently, you can change the file-based
setting `AWX_REBUILD_SMART_MEMBERSHIP` to *True* (default is False).
This will update memberships in the following events:

* a new host is added
* an existing host is modified (updated or deleted)
* a new Smart Inventory is added
* an existing Smart Inventory is modified (updated or deleted)

You can view actual inventories without being editable:

* Names of Host and Group created as a result of an inventory source
sync
* Group records cannot be edited or moved

You cannot create hosts from a Smart Inventory host endpoint
(`/inventories/N/hosts/`) as with a normal inventory. The administrator
of a Smart Inventory has permission to edit fields such as the name,
description, variables, and the ability to delete, but does not have the
permission to modify the `host_filter`, because that will affect which
hosts (that have a primary membership inside another inventory) are
included in the smart inventory. Note, `host_filter` only apply to hosts
inside of inventories inside of the Smart Inventory's organization.

In order to modify the `host_filter`, you need to be the organization
administrator of the inventory's organization. Organization admins
already have implicit "admin" access to all inventories inside the
organization, therefore, this does not convey any permissions they did
not already possess.

Administrators of the Smart Inventory can grant other users (who are not
also admins of your organization) permissions like "use" "adhoc" to the
smart inventory, and these will allow the actions indicate by the role,
just like other standard inventories. However, this will not give them
any special permissions to hosts (which live in a different inventory).
It will not allow them direct read permission to hosts, or permit them
to see additional hosts under `/#/hosts/`, although they can still view
the hosts under the smart inventory host list.

In some situations, you can modify the following:

* A new Host manually created on Inventory w/ inventory sources
* In Groups that were created as a result of inventory source syncs
* Variables on Host and Group are changeable

Hosts associated with the Smart Inventory are manifested at view time.
If the results of a Smart Inventory contains more than one host with
identical hostnames, only one of the matching hosts will be included as
part of the Smart Inventory, ordered by Host ID.

[[ug_host_filters]]
==== Smart Host Filter

You can use a search filter to populate hosts for an inventory. This
feature utilized the capability of the fact searching feature.

Facts generated by an Ansible playbook during a Job Template run are
stored by the automation controller into the database whenever
`use_fact_cache=True` is set per-Job Template. New facts are merged with
existing facts and are per-host. These stored facts can be used to
filter hosts via the `/api/v2/hosts` endpoint, using the `GET` query
parameter `host_filter` For example:
`/api/v2/hosts?host_filter=ansible_facts__ansible_processor_vcpus=8`

The `host_filter` parameter allows for:

* grouping via ()
* use of the boolean and operator:
** `__` to reference related fields in relational fields
** `__` is used on ansible_facts to separate keys in a JSON key path
** `[] is used to denote a json array in the path specification
** `""` can be used in the value when spaces are wanted in the value
* "classic" Django queries may be embedded in the `host_filter`

Examples:

....
/api/v2/hosts/?host_filter=name=localhost
/api/v2/hosts/?host_filter=ansible_facts__ansible_date_time__weekday_number="3"
/api/v2/hosts/?host_filter=ansible_facts__ansible_processor[]="GenuineIntel"
/api/v2/hosts/?host_filter=ansible_facts__ansible_lo__ipv6[]__scope="host"
/api/v2/hosts/?host_filter=ansible_facts__ansible_processor_vcpus=8
/api/v2/hosts/?host_filter=ansible_facts__ansible_env__PYTHONUNBUFFERED="true"
/api/v2/hosts/?host_filter=(name=localhost or name=database) and (groups__name=east or groups__name="west coast") and ansible_facts__an
....

You can search `host_filter` by host name, group name, and Ansible
facts.

The format for a group search is:

....
groups.name:groupA
....

The format for a fact search is:

....
ansible_facts.ansible_fips:false
....

You can also perform Smart Search searches, which consist a host name
and host description.

....
host_filter=name=my_host
....

If a search term in `host_filter` is of string type, to make the value a
number (e.g. `2.66`), or a JSON keyword (e.g. `null`, `true` or `false`)
valid, add double quotations around the value to prevent the controller
from mistakenly parsing it as a non-string:

....
host_filter=ansible_facts__packages__dnsmasq[]__version="2.66"
....

[[ug_host_filter_facts]]
==== Define host filter with ansible_facts

To use `ansible_facts` to define the host filter when creating Smart
Inventories, perform the following steps:

[arabic]
. In the _Create new smart inventory screen_, click the
image:search-button.png[search] button next
to the *Smart host filter* field to open a pop-up window to filter hosts
for this inventory.

image:inventories-smart-create-filter-highlighted.png[image]

[arabic, start=2]
. In the search pop-up window, change the search criteria from *Name* to
*Advanced* and select *ansible_facts* from the *Key* field.

image:inventories-smart-define-host-filter.png[image]

If you wanted to add an ansible fact of

....
/api/v2/hosts/?host_filter=ansible_facts__ansible_processor[]="GenuineIntel"
....

In the search field, enter `ansible_processor[]="GenuineIntel"` (no
extra spaces or `__` before the value) and press *[Enter]*.

image:inventories-smart-define-host-filter-facts.png[image]

The resulting search criteria for the specified ansible fact populates
in the lower part of the window.

image:inventories-smart-define-host-filter-facts2.png[image]

[arabic, start=3]
. Click *Select* to add it to the *Smart host filter* field.

image:inventories-smart-create-filter-added.png[image]

[arabic, start=4]
. Click *Save* to save the new Smart Inventory.

The Details tab of the new Smart Inventory opens and displays the
specified ansible facts in the *Smart host filter* field.

image:inventories-smart-create-details.png[image]

[arabic, start=5]
. From the Details view, you can edit the *Smart host filter* field by
clicking *Edit* and delete existing filter(s), clear all existing
filters, or add new ones.

image:inventories-smart-define-host-filter-facts-group.png[image]

[[ug_inventories_plugins]]
=== Inventory Plugins

Inventory updates use dynamically-generated YAML files which are parsed
by their respective inventory plugin. In Automation Controller Version
4.3.0, users can provide the new style inventory plugin config directly
to the controller via the inventory source `source_vars` for all the
following inventory sources:

* xref:ug_source_ec2[]
* xref:ug_source_gce[]
* xref:ug_source_azure[]
* xref:ug_source_vmvcenter[]
* xref:ug_source_satellite[]
* xref:ug_source_insights[]
* xref:ug_source_openstack[]
* xref:ug_source_rhv[]
* xref:ug_source_tower[]

Newly created configurations for inventory sources will contain the
default plugin configuration values. If you want your newly created
inventory sources in 3.8 to match the output of a 3.7 source, you must
apply a specific set of configuration values for that source. To ensure
backward compatibility, the controller uses "templates" for each of
these sources to force the output of inventory plugins into the legacy
format. Refer to `ir_inv_plugin_templates_reference` section of this
guide for each source and their respective templates to help you migrate
to the new style inventory plugin output.

`source_vars` that contain `plugin: foo.bar.baz` as a top-level key will
be replaced with the appropriate fully-qualified inventory plugin name
at runtime based on the `InventorySource` source. For example, if ec2 is
selected for the `InventorySource` then, at run-time, plugin will be set
to `amazon.aws.aws_ec2`.

[[ug_inventories_add]]
=== Add a new inventory

Adding a new inventory involves several components:

* xref:ug_inventories_add_permissions[]
* xref:ug_inventories_add_groups[]
* xref:ug_inventories_add_host[]
* xref:ug_inventories_add_source[]
* xref:ug_inventories_view_completed_jobs[]

To create a new inventory or Smart Inventory:

[arabic]
. Click the *Add* button, and select the type of inventory to create.

The type of inventory is identified at the top of the create form.

image:inventories-create-new-inventory.png[Inventories_create_new
- create new inventory]

[arabic, start=2]
. Enter the appropriate details into the following fields:

* *Name*: Enter a name appropriate for this inventory.
* *Description*: Enter an arbitrary description as appropriate
(optional).
* *Organization*: Required. Choose among the available organizations.
* *Smart Host Filter*: (Only applicable to Smart Inventories) Click the
image:search-button.png[search] button to
open a separate window to filter hosts for this inventory. These options
are based on the organization you chose.
+
Filters are similar to tags in that tags are used to filter certain
hosts that contain those names. Therefore, to populate the *Smart Host
Filter* field, you are specifying a tag that contains the hosts you
want, not actually selecting the hosts themselves. Enter the tag in the
*Search* field and press [Enter]. Filters are case-sensitive. Refer to
the xref:ug_host_filters[] section for more information.
* *Instance Groups*: Click the
image:search-button.png[search] button to
open a separate window. Choose the instance groups for this inventory to
run on. If the list is extensive, use the search to narrow the options.
* *Variables*: Variable definitions and values to be applied to all
hosts in this inventory. Enter variables using either JSON or YAML
syntax. Use the radio button to toggle between the two.

image:inventories-create-new-saved-inventory.png[Inventories_create_new_saved
- create new inventory]

[arabic, start=3]
. Click *Save* when done.

After saving the new inventory, you can proceed with configuring
permissions, groups, hosts, sources, and view completed jobs, if
applicable to the type of inventory. For more instructions, refer to the
subsequent sections.

[[ug_inventories_add_permissions]]
==== Add permissions

include::permissions.adoc[]

[[ug_inventories_add_groups]]
==== Add groups

Inventories are divided into groups, which may contain hosts and other
groups, and hosts. Groups are only applicable to standard inventories
and is not a configurable directly through a Smart Inventory. You can
associate an existing group through host(s) that are used with standard
inventories. There are several actions available for standard
inventories:

* Create a new Group
* Create a new Host
* Run a command on the selected Inventory
* Edit Inventory properties
* View activity streams for Groups and Hosts
* Obtain help building your Inventory

Note

Inventory sources are not associated with groups. Spawned groups are
top-level and may still have child groups, and all of these spawned
groups may have hosts.

To create a new group for an inventory:

[arabic]
. Click the *Add* button to open the *Create Group* window.

image:inventories-add-group-new.png[Inventories_manage_group_add]

[arabic, start=2]
. Enter the appropriate details into the required and optional fields:

* *Name*: Required
* *Description*: Enter an arbitrary description as appropriate
(optional)
* *Variables*: Enter definitions and values to be applied to all hosts
in this group. Enter variables using either JSON or YAML syntax. Use the
radio button to toggle between the two.

[arabic, start=3]
. When done, click *Save*.

===== Add groups within groups

To add groups within groups:

[arabic]
. Click the *Related Groups* tab.
. Click the *Add* button, and select whether to add a group that already
exists in your configuration or create a new group.
. If creating a new group, enter the appropriate details into the
required and optional fields:

* *Name*: Required
* *Description*: Enter an arbitrary description as appropriate
(optional)
* *Variables*: Enter definitions and values to be applied to all hosts
in this group. Enter variables using either JSON or YAML syntax. Use the
radio button to toggle between the two.

[arabic, start=4]
. When done, click *Save*.

The *Create Group* window closes and the newly created group displays as
an entry in the list of groups associated with the group that it was
created for.

image:inventories-add-group-subgroup-added.png[Inventories
add group subgroup]

If you chose to add an existing group, available groups will appear in a
separate selection window.

image:inventories-add-group-existing-subgroup.png[Inventories
add group existing subgroup]

Once a group is selected, it displays as an entry in the list of groups
associated with the group.

{empty}5. To configure additional groups and hosts under the subgroup,
click on the name of the subgroup from the list of groups and repeat the
same steps described in this section.

===== View or edit inventory groups

The list view displays all your inventory groups at once, or you can
filter it to only display the root group(s). An inventory group is
considered a root group if it is not a subset of another group.

You may be able to delete a subgroup without concern for dependencies,
since the controller will look for dependencies such as any child groups
or hosts. If any exists, a confirmation dialog displays for you to
choose whether to delete the root group and all of its subgroups and
hosts; or promote the subgroup(s) so they become the top-level inventory
group(s), along with their host(s).

image:inventories-groups-delete-root-with-children.png[image]

[[ug_inventories_add_host]]
==== Add hosts

You can configure hosts for the inventory as well as for groups and
groups within groups. To configure hosts:

[arabic]
. Click the *Hosts* tab.
. Click the *Add* button, and select whether to add a host that already
exists in your configuration or create a new host.
. If creating a new host, select the
image:on-off-toggle-button.png[toggle button]
button to specify whether or not to include this host while running
jobs.
. Enter the appropriate details into the required and optional fields:

* *Host Name*: Required
* *Description*: Enter an arbitrary description as appropriate
(optional)
* *Variables*: Enter definitions and values to be applied to all hosts
in this group. Enter variables using either JSON or YAML syntax. Use the
radio button to toggle between the two.

[arabic, start=5]
. When done, click *Save*.

The *Create Host* window closes and the newly created host displays as
an entry in the list of hosts associated with the group that it was
created for.

image:inventories-add-group-host-added.png[Inventories
add group host]

If you chose to add an existing host, available hosts will appear in a
separate selection window.

image:inventories-add-existing-host.png[Inventories
add existing host]

Once a host is selected, it displays as an entry in the list of hosts
associated with the group. You can disassociate a host from this screen
by selecting the host and click the *Disassociate* button.

Note

You may also run ad hoc commands from this screen. Refer to
xref:ug_inventories_run_ad_hoc[] for more detail.

{empty}6. To configure additional groups for the host, click on the name
of the host from the list of hosts.

image:inventories-add-group-host-added-emphasized.png[Inventories
add group host emphasized]

This opens the Details tab of the selected host.

image:inventories-add-group-host-details.png[Inventories
add group host details]

[arabic, start=7]
. Click the *Groups* tab to configure groups for the host.

___________________________________________________________________________________________________
--
[loweralpha]
. Click the *Add* button to associate the host with an existing group.

__________________________________________________________________________________
Available groups appear in a separate selection window.

image:inventories-add-group-hosts-add-groups.png[image]
__________________________________________________________________________________

[loweralpha, start=2]
. Click to select the group(s) to associate with the host and click
*Save*.

Once a group is associated, it displays as an entry in the list of
groups associated with the host.

--
___________________________________________________________________________________________________

[arabic, start=8]
. If a host was used to run a job, you can view details about those jobs
in the *Completed Jobs* tab of the host and click *Expanded* to view
details about each job.

image:inventories-add-host-view-completed-jobs.png[image]

[[ug_inventories_add_source]]
==== Add source

Inventory sources are not associated with groups. Spawned groups are
top-level and may still have child groups, and all of these spawned
groups may have hosts. Adding a source to an inventory only applies to
standard inventories. Smart inventories inherit their source from the
standard inventories they are associated with. To configure the source
for the inventory:

[arabic]
. In the inventory you want to add a source, click the *Sources* tab.
. Click the *Add* button.

This opens the Create Source window.

image:inventories-create-source.png[Inventories
create source]

[arabic, start=3]
. Enter the appropriate details into the required and optional fields:

______________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
* *Name*: Required
* *Description*: Enter an arbitrary description as appropriate
(optional)
* *Execution Environment*: Optionally search
(image:search-button.png[search]) or enter
the name of the execution environment with which you want to run your
inventory imports. Refer to the {ug_execution_environments} section for
details on building an execution environment.
* *Source*: Choose a source for your inventory. Refer to the
xref:ug_inventory_sources[] section for more information about each source
and details for entering the appropriate information.
______________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

[[ug_add_inv_common_fields]]
[arabic, start=4]
. After completing the required information for your chosen
`inventory source <ug_inventory_sources>`, you can continue to
optionally specify other common parameters, such as verbosity, host
filters, and variables.

_____________________________________________________________________________________________________________
Note

The *Regions*, *Instance Filters*, and *Only Group By* fields have been
removed in automation controller 3.8.
_____________________________________________________________________________________________________________

[arabic, start=5]
. Select the appropriate level of output on any inventory source's
update jobs from the *Verbosity* drop-down menu.
. Use the *Host Filter* field to specify only matching host names to be
imported into the controller.
. In the *Enabled Variable*, specify the controller to retrieve the
enabled state from the given dictionary of host variables. The enabled
variable may be specified using dot notation as 'foo.bar', in which case
the lookup will traverse into nested dicts, equivalent to:
`from_dict.get('foo', {}).get('bar', default)`.
. If you specified a dictionary of host variables in the *Enabled
Variable* field, you can provide a value to enable on import. For
example, if `enabled_var='status.power_state'` and
`enabled_value='powered_on'` with the following host variables, the host
would be marked enabled:

___________________________________________________________________________________________________________________________________________________________________________________
....
{
"status": {
"power_state": "powered_on",
"created": "2020-08-04T18:13:04+00:00",
"healthy": true
},
"name": "foobar",
"ip_address": "192.168.2.1"
}
....

If `power_state` were any value other than `powered_on`, then the host
would be disabled when imported into the controller. If the key is not
found, then the host will be enabled.
___________________________________________________________________________________________________________________________________________________________________________________

[arabic, start=9]
. All cloud inventory sources have the following update options:

_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
--
* *Overwrite*: If checked, any hosts and groups that were previously
present on the external source but are now removed, will be removed from
the controller inventory. Hosts and groups that were not managed by the
inventory source will be promoted to the next manually created group, or
if there is no manually created group to promote them into, they will be
left in the "all" default group for the inventory.

______________________________________________________________________________________________________________________________________
When not checked, local child hosts and groups not found on the external
source will remain untouched by the inventory update process.
______________________________________________________________________________________________________________________________________

* *Overwrite Variables*: If checked, all variables for child groups and
hosts will be removed and replaced by those found on the external
source. When not checked, a merge will be performed, combining local
variables with those found on the external source.
* *Update on Launch*: Each time a job runs using this inventory, refresh
the inventory from the selected source before executing job tasks. To
avoid job overflows if jobs are spawned faster than the inventory can
sync, selecting this allows you to configure a *Cache Timeout* to cache
prior inventory syncs for a certain number of seconds.

_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
The "Update on Launch" setting refers to a dependency system for
projects and inventory, and it will not specifically exclude two jobs
from running at the same time. If a cache timeout is specified, then the
dependencies for the second job is created and it uses the project and
inventory update that the f.adoc[] job spawned. Both jobs then wait for
that project and/or inventory update to finish before proceeding. If
they are different job templates, they can then both start and run at
the same time, if the system has the capacity to do so. If you intend to
use the controller's provisioning callback feature with a dynamic
inventory source, *Update on Launch* should be set for the inventory
group.

If you sync an inventory source that uses a project that has *Update On
Launch* set, then the project may automatically update (according to
cache timeout rules) before the inventory update starts.

You can create a job template that uses an inventory that sources from
the same project that the template uses. In this case, the project will
update and then the inventory will update (if updates are not already
in-progress, or if the cache timeout has not already expired).
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

--
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

[arabic, start=10]
. Review your entries and selections and click *Save* when done. This
allows you to configure additional details, such as schedules and
notifications.
. To configure schedules associated with this inventory source, click
the *Schedules* tab.

_____________________________________________________________________________________________
[loweralpha]
. If schedules are already set up; review, edit, or enable/disable your
schedule preferences.
. if schedules have not been set up, refer to {ug_scheduling} for more
information.
_____________________________________________________________________________________________

Note

The *Notifications* tab is only present after you save the newly-created
source.

image:inventories-create-source-with-notifications-tab.png[image]

[arabic, start=12]
. To configure notifications for the source, click the *Notifications*
tab.

_________________________________________________________________________________________________________________________________________________________________________________
[loweralpha]
. If notifications are already set up, use the toggles to enable or
disable the notifications to use with your particular source. For more
detail, see {ug_notifications_on_off}.
. if notifications have not been set up, refer to {ug_notifications} for
more information.
_________________________________________________________________________________________________________________________________________________________________________________

[arabic, start=13]
. Review your entries and selections and click *Save* when done.

Once a source is defined, it displays as an entry in the list of sources
associated with the inventory. From the *Sources* tab you can perform a
sync on a single source, or sync all of them at once. You can also
perform additional actions such as scheduling a sync process, and edit
or delete the source.

image:inventories-view-sources.png[Inventories
view sources]

[[ug_inventory_sources]]
===== Inventory Sources

Choose a source which matches the inventory type against which a host
can be entered:

====== Sourced from a Project

An inventory that is sourced from a project means that is uses the SCM
type from the project it is tied to. For example, if the project's
source is from GitHub, then the inventory will use the same source.

[arabic]
. To configure a project-sourced inventory, select *Sourced from a
Project* from the Source field.
. The Create Source window expands with additional fields. Enter the
following details:

___________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
* *Credential*: Optionally specify the credential to use for this
source.
* *Project*: Required. Specify the project this inventory is using as
its source. Click the
image:search-button.png[search] button to
choose from a list of projects. If the list is extensive, use the search
to narrow the options.
* *Inventory File*: Required. Select an inventory file associated with
the sourced project. If not already populated, you can type it into the
text field within the drop down menu to filter the extraneous file
types. In addition to a flat file inventory, you can point to a
directory or an inventory script.

image:inventories-create-source-sourced-from-project-filter.png[image]
___________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

[arabic, start=3]
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.
. In order to pass to the custom inventory script, you can optionally
set environment variables in the *Environment Variables* field. You may
also place inventory scripts in source control and then run it from a
project. See {ag_inv_import} in the Automation Controller Administration
Guide for detail.

image:inventories-create-source-sourced-from-project-example.png[Inventories
- create source - sourced from project example]

Note

If you are executing a custom inventory script from SCM, please make
sure you set the execution bit (i.e. `chmod +x`) on the script in your
upstream source control. If you do not, the controller will throw a
`[Errno 13] Permission denied` error upon execution.

[[ug_source_ec2]]
====== Amazon Web Services EC2

[arabic]
. To configure an AWS EC2-sourced inventory, select *Amazon EC2* from
the Source field.
. The Create Source window expands with additional fields. Enter the
following details:

____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
* *Credential*: Optionally choose from an existing AWS credential (for
more information, refer to xref:ug_credentials[]).
+
If the controller is running on an EC2 instance with an assigned IAM
Role, the credential may be omitted, and the security credentials from
the instance metadata will be used instead. For more information on
using IAM Roles, refer to the
http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-%20roles-for-amazon-ec2.html[IAM_Roles_for_Amazon_EC2_documentation_at_Amazon].
____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

[arabic, start=3]
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.
. Use the *Source Variables* field to override variables used by the
`aws_ec2` inventory plugin. Enter variables using either JSON or YAML
syntax. Use the radio button to toggle between the two. For a detailed
description of these variables, view the
https://cloud.redhat.com/ansible/automation-hub/repo/published/amazon/aws/content/inventory/aws_ec2[aws_ec2
inventory plugin documenation].

image:inventories-create-source-AWS-example.png[Inventories
- create source - AWS EC2 example]

Note

If you only use `include_filters`, the AWS plugin always returns all the
hosts. To use this properly, the f.adoc[] condition on the `or` must be on
`filters` and then build the rest of the `OR` conditions on a list of
`include_filters`.

[[ug_source_gce]]
====== Google Compute Engine

[arabic]
. To configure a Google-sourced inventory, select *Google Compute
Engine* from the Source field.
. The Create Source window expands with the required *Credential* field.
Choose from an existing GCE Credential. For more information, refer to
xref:ug_credentials[].

image:inventories-create-source-GCE-example.png[Inventories
- create source - GCE example]

[arabic, start=3]
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.
. Use the *Source Variables* field to override variables used by the
`gcp_compute` inventory plugin. Enter variables using either JSON or
YAML syntax. Use the radio button to toggle between the two. For a
detailed description of these variables, view the
https://cloud.redhat.com/ansible/automation-hub/repo/published/google/cloud/content/inventory/gcp_compute[gcp_compute
inventory plugin documenation].

[[ug_source_azure]]
====== Microsoft Azure Resource Manager

[arabic]
. To configure a Azure Resource Manager-sourced inventory, select
*Microsoft Azure Resource Manager* from the Source field.
. The Create Source window expands with the required *Credential* field.
Choose from an existing Azure Credential. For more information, refer to
xref:ug_credentials[].
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.
. Use the *Source Variables* field to override variables used by the
`azure_rm` inventory plugin. Enter variables using either JSON or YAML
syntax. Use the radio button to toggle between the two. For a detailed
description of these variables, view the
https://cloud.redhat.com/ansible/automation-hub/repo/published/azure/azcollection/content/inventory/azure_rm[azure_rm
inventory plugin documentation].

image:inventories-create-source-azurerm-example.png[Inventories
- create source - Azure RM example]

[[ug_source_vmvcenter]]
====== VMware vCenter

[arabic]
. To configure a VMWare-sourced inventory, select *VMware vCenter* from
the Source field.
. The Create Source window expands with the required *Credential* field.
Choose from an existing VMware Credential. For more information, refer
to xref:ug_credentials[].
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.
. Use the *Source Variables* field to override variables used by the
`vmware_inventory` inventory plugin. Enter variables using either JSON
or YAML syntax. Use the radio button to toggle between the two. For a
detailed description of these variables, view the
https://github.com/ansible-collections/community.vmware/blob/main/plugins/inventory/vmware_vm_inventory.py[vmware_inventory
inventory plugin].

_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
Starting with Ansible 2.9, VMWare properties have changed from lower
case to camelCase. The controller provides aliases for the top-level
keys, but lower case keys in nested properties have been discontinued.
For a list of valid and supported properties starting with Ansible 2.9,
refer to the primary
https://github.com/ansible/ansible/blob/devel/docs/docsite.adoc[]/scenario_guides/vmware_scenarios/vmware_inventory_vm_attributes.adoc[][documentation
for hostvars from VMWare inventory imports] in GitHub.
_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

image:inventories-create-source-vmware-example.png[Inventories
- create source - VMWare example]

[[ug_source_satellite]]
====== Red Hat Satellite 6

[arabic]
. To configure a Red Hat Satellite-sourced inventory, select *Red Hat
Satellite* from the Source field.
. The Create Source window expands with the required *Credential* field.
Choose from an existing Satellite Credential. For more information,
refer to xref:ug_credentials[].
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.
. Use the *Source Variables* field to specify parameters used by the
foreman inventory source. Enter variables using either JSON or YAML
syntax. Use the radio button to toggle between the two. For a detailed
description of these variables, refer to the
https://docs.ansible.com/ansible/latest/collections/theforeman/foreman/foreman_inventory.html[theforeman.foreman.foreman
– Foreman inventory source] in the Ansible documentation.

image:inventories-create-source-rhsat6-example.png[Inventories
- create source - RH Satellite example]

If you encounter an issue with the controller inventory not having the
"related groups" from Satellite, you might need to define these
variables in the inventory source. See the inventory plugins template
example for `ir_plugin_satellite` in the Ansible Automation Platform
Installation and Reference Guide for detail.

If you see the message,
`"no foreman.id" variable(s) when syncing the inventory`, refer to the
solution on the Red Hat Customer Portal at:
https://access.redhat.com/solutions/5826451. Be sure to login with your
customer credentials to access the full article.

[[ug_source_insights]]
====== Red Hat Insights

[arabic]
. To configure a Red Hat Insights-sourced inventory, select *Red Hat
Insights* from the Source field.
. The Create Source window expands with the required *Credential* field.
Choose from an existing Insights Credential. For more information, refer
to xref:ug_credentials[].
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.
. Use the *Source Variables* field to override variables used by the
`insights` inventory plugin. Enter variables using either JSON or YAML
syntax. Use the radio button to toggle between the two. For a detailed
description of these variables, view the
https://cloud.redhat.com/ansible/automation-hub/repo/published/redhat/insights/content/inventory/insights[insights
inventory plugin].

image:inventories-create-source-insights-example.png[Inventories
- create source - RH Insights example]

[[ug_source_openstack]]
====== OpenStack

[arabic]
. To configure an OpenStack-sourced inventory, select *OpenStack* from
the Source field.
. The Create Source window expands with the required *Credential* field.
Choose from an existing OpenStack Credential. For more information,
refer to xref:ug_credentials[].
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.
. Use the *Source Variables* field to override variables used by the
`openstack` inventory plugin. Enter variables using either JSON or YAML
syntax. Use the radio button to toggle between the two. For a detailed
description of these variables, view the
https://docs.ansible.com/ansible/latest/collections/openstack/cloud/openstack_inventory.html[openstack
inventory plugin] in the Ansible collections documentation.

image:inventories-create-source-openstack-example.png[Inventories
- create source - OpenStack example]

[[ug_source_rhv]]
====== Red Hat Virtualization

[arabic]
. To configure a Red Hat Virtualization-sourced inventory, select *Red
Hat Virtualization* from the Source field.
. The Create Source window expands with the required *Credential* field.
Choose from an existing Red Hat Virtualization Credential. For more
information, refer to xref:ug_credentials[].
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.
. Use the *Source Variables* field to override variables used by the
`ovirt` inventory plugin. Enter variables using either JSON or YAML
syntax. Use the radio button to toggle between the two. For a detailed
description of these variables, view the
https://cloud.redhat.com/ansible/automation-hub/repo/published/redhat/rhv/content/inventory/ovirt[ovirt
inventory plugin].

image:inventories-create-source-rhv-example.png[Inventories
- create source - RHV example]

Note

Red Hat Virtualization (ovirt) inventory source requests are secure by
default. To change this default setting, set the key `ovirt_insecure` to
*true* in `source_variables`, which is only available from the API
details of the inventory source at the `/api/v2/inventory_sources/N/`
endpoint.

[[ug_source_tower]]
====== Red Hat Ansible Automation Platform

[arabic]
. To configure a automation controller-sourced inventory, select *Red
Hat Ansible Automation Platform* from the Source field.
. The Create Source window expands with the required *Credential* field.
Choose from an existing Ansible Automation Platform Credential. For more
information, refer to xref:ug_credentials[].
. You can optionally specify the verbosity, host filter, enabled
variable/value, and update options as described in the main procedure
for `adding a source <ug_add_inv_common_fields>`.

___________________________________________________________________________________
image:inventories-create-source-rhaap-example.png[image]
___________________________________________________________________________________

[arabic, start=4]
. Use the *Source Variables* field to override variables used by the
`controller` inventory plugin. Enter variables using either JSON or YAML
syntax. Use the radio button to toggle between the two. For a detailed
description of these variables, view the
https://cloud.redhat.com/ansible/automation-hub/repo/published/ansible/controller/content/inventory/controller[controller
inventory plugin] (requires your Red Hat Customer login).

[[ug_customscripts]]
===== Export old inventory scripts

Despite the removal of the custom inventory scripts API, the scripts are
still saved in the database. The commands described in this section
allows you to recover the scripts in a format that is suitable for you
to subsequently check into source control. Usage looks like this:

....
$ awx-manage export_custom_scripts --filename=my_scripts.tar
Dump of old custom inventory scripts at my_scripts.tar
....

Making use of the output:

....
$ mkdir my_scripts
$ tar -xf my_scripts.tar -C my_scripts
....

The naming of the scripts is `_<pk>__<name>`. This is the naming scheme
used for project folders.

:

....
$ ls my_scripts
_10__inventory_script_rawhook             _19__                                       _30__inventory_script_listenhospital
_11__inventory_script_upperorder          _1__inventory_script_commercialinternet45   _4__inventory_script_whitestring
_12__inventory_script_eastplant           _22__inventory_script_pinexchange           _5__inventory_script_literaturepossession
_13__inventory_script_governmentculture   _23__inventory_script_brainluck             _6__inventory_script_opportunitytelephone
_14__inventory_script_bottomguess         _25__inventory_script_buyerleague           _7__inventory_script_letjury
_15__inventory_script_wallisland          _26__inventory_script_lifesport             _8__random_inventory_script_
_16__inventory_script_wallisland          _27__inventory_script_exchangesomewhere     _9__random_inventory_script_
_17__inventory_script_bidstory            _28__inventory_script_boxchild
_18__p                                    _29__inventory_script_we.adoc[]ress
....

Each file contains a script. Scripts can be `bash/python/ruby/more`, so
the extension is not included. They are all directly executable
(assuming the scripts worked). If you execute the script, it dumps the
inventory data.

....
$ ./my_scripts/_11__inventory_script_upperorder 
{"group_\ud801\udcb0\uc20e\u7b0e\ud81c\udfeb\ub12b\ub4d0\u9ac6\ud81e\udf07\u6ff9\uc17b": {"hosts": 
["host_\ud821\udcad\u68b6\u7a51\u93b4\u69cf\uc3c2\ud81f\uddbe\ud820\udc92\u3143\u62c7", 
"host_\u6057\u3985\u1f60\ufefb\u1b22\ubd2d\ua90c\ud81a\udc69\u1344\u9d15", 
"host_\u78a0\ud820\udef3\u925e\u69da\ua549\ud80c\ude7e\ud81e\udc91\ud808\uddd1\u57d6\ud801\ude57", 
"host_\ud83a\udc2d\ud7f7\ua18a\u779a\ud800\udf8b\u7903\ud820\udead\u4154\ud808\ude15\u9711", 
"host_\u18a1\u9d6f\u08ac\u74c2\u54e2\u740e\u5f02\ud81d\uddee\ufbd6\u4506"], "vars": {"ansible_host": "127.0.0.1", "ansible_connection": 
"local"}}}
....

You can verify functionality with `ansible-inventory`. This should give
the same data, but reformatted.

....
$ ansible-inventory -i ./my_scripts/_11__inventory_script_upperorder --list --export
....

In the above example, you could `cd` into `my_scripts` and then issue a
`git init` command, add the scripts you want, push it to source control,
and then create an SCM inventory source in the automation controller
user interface.

For more information on syncing or using custom inventory scripts, refer
to {ag_inv_import} in the[] in the Automation Controller Administration
Guide.

[[ug_inventories_view_completed_jobs]]
==== View completed jobs

If an inventory was used to run a job, you can view details about those
jobs in the *Completed Jobs* tab of the inventory and click *Expanded*
to view details about each job.

image:inventories-view-completed-jobs.png[Inventories
view completed jobs]

[[ug_inventories_run_ad_hoc]]
=== Running Ad Hoc Commands

To run an ad hoc command:

[arabic]
. Select an inventory source from the list of hosts or groups. The
inventory source can be a single group or host, a selection of multiple
hosts, or a selection of multiple groups.

image:inventories-add-group-host-added.png[ad
hoc-commands-inventory-home]

[arabic, start=2]
. Click the *Run Command* button.

The Run command window opens.

image:ad-hoc-run-execute-command.png[image]

[arabic, start=3]
. Enter the details for the following fields:

* *Module*: Select one of the modules that the automation controller
supports running commands against.
+
+---------+----------------+----------+-------------+ | command |
apt_repository | mount | win_service |
+---------+----------------+----------+-------------+ | shell | apt_rpm
| ping | win_updates |
+---------+----------------+----------+-------------+ | yum | service |
selinux | win_group |
+---------+----------------+----------+-------------+ | apt | group |
setup | win_user | +---------+----------------+----------+ + | apt_key |
user | win_ping | |
+---------+----------------+----------+-------------+
* *Arguments*: Provide arguments to be used with the module you
selected.
* *Limit*: Enter the limit used to target hosts in the inventory. To
target all hosts in the inventory enter `all` or `*`, or leave the field
blank. This is automatically populated with whatever was selected in the
previous view prior to clicking the launch button.
* *Machine Credential*: Select the credential to use when accessing the
remote hosts to run the command. Choose the credential containing the
username and SSH key or password that Ansbile needs to log into the
remote hosts.
* *Verbosity*: Select a verbosity level for the standard output.
* *Forks*: If needed, select the number of parallel or simultaneous
processes to use while executing the command.
* *Show Changes*: Select to enable the display of Ansible changes in the
standard output. The default is OFF.
* *Enable Privilege Escalation*: If enabled, the playbook is run with
administrator privileges. This is the equivalent of passing the
`--become` option to the `ansible` command.
* *Extra Variables*: Provide extra command line variables to be applied
when running this inventory. Enter variables using either JSON or YAML
syntax. Use the radio button to toggle between the two.

image:ad-hoc-commands-inventory-run-command.png[ad
hoc-commands-inventory-run-command]

[arabic, start=4]
. Click *Next* to choose the execution environment you want the ad-hoc
command to be run against.

image:ad-hoc-commands-inventory-run-command-ee.png[image]

[arabic, start=5]
. Click *Next* to choose the credential you want to use and click the
*Launch* button.

The results display in the *Output* tab of the module's job window.

image:ad-hoc-commands-inventory-results-example.png[ad
hoc-commands-inventory-results-example]
