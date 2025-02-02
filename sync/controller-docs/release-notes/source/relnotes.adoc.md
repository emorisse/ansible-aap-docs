[[release_notes]]
== Release Notes

Refer to the latest
https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/[Product
Documentation for Red Hat Ansible Automation Platform] for the complete
Automation Platform documentation.

=== Automation Controller Version 4.3

include::relnotes_current.adoc[]

=== Automation Controller Version 4.2.1

=== Automation Controller Version 4.2

*Introduced*

* Graphical visualization of the automation topology to show the types
of nodes, the links between them and their statuses

*Added*

* For VM-based installs, the controller will now automatically mount the
system trust store in execution environments when jobs run
* *Log Format For API 4XX Errors* field to the Logging settings form to
allow customization of 4xx error messages that are produced when the API
encounters an issue with a request
* Ability to use labels with inventory
* Ability to flag users as superusers and auditors in SAML integration
* Support for expanding and collapsing plays and tasks in the job output
UI
* Filtering job output UI by multiple event types
* Various default search filters to a number of list views
* Top-level list of instances to now be visible in the UI
* A pop-up message when a user copies a resource
* Job Templates tab to Credentials and Inventories to view all the
templates that use that particular credential or inventory

*Updated*

* Controller to use Python 3.9
* Django's `SESSION_COOKIE_NAME` setting to a non-default value. *Note*,
any external clients that previously used the `sessionid` cookie will
need to change. Refer to `api_session_auth` for more detail.
* Controller to support podman-style volume mount syntax in the *Paths
to expose to isolated jobs* field of the Jobs Settings of the UI
* Isolated path to be exposed in OCP/K8s as HostPath
* Upgraded Django from version 2.2 to 3.2
* Modified usage of `ansible_facts` on Advanced Search to add more
flexibility to the usage of `ansible_facts` when creating a smart
inventory
* The controller node for a job running on an execution node now incurs
a penalty of 1 unit of capacity to account for the system load that
controlling a job incurs. This can be adjusted with the file-based
setting `AWX_CONTROL_NODE_TASK_IMPACT`.
* Project updates to always run in the `controlplane` instance group
* Slack notifications to allow replying to a thread instead of just
channels
* UI performance to improve job output
* Job status icons to be more accessible
* Display of only usable inventories when launching a job
* Browser tab to show more information about which page the user is
currently viewing
* Controller to now load variables _after_ job template extra variables
to prevent overriding the meta variables injected into each job run

*Deprecated*

* The concept of "committed capacity" from Instance Groups due to the
removal of RabbitMQ
* Inventory source option to *Update on project update* - this field
updates the inventory source if its project pulled a new revision. In
the future, when updating an inventory source, the controller shall
automatically run project updates if the project itself is set to
*Update on launch*.

*Removed*

* Case sensitivity around hostcount

=== Automation Controller Version 4.1.3

*Automation Controller fixes:*

* Receptor no longer fails in FIPS mode
* Added the ability to exit gracefully and recover quickly when a
service in the control plane crashes
* The `create_partition` method will skip creating a table if it already
exists
* Having logging enabled no longer breaks migrations if the migration
sends logs to an external aggregator
* Fixed the metrics endpoint (`/api/v2/metrics`) to no longer produce
erroneous `500` errors

*Execution Environment fixes:*

* Enhanced the execution environment copy process to reduce required
space in the `/tmp` directory
* Allowed execution environment images to be pulled from automation
controller only
* Added the *ansible-builder-rhel8* image to the setup bundle
* Modified base execution environment images so that controller backups
can run in the container

*Automation Controller UI fixes:*

* Upon saving a schedule, the date chooser no longer changes to the day
before the selected date
* Fixed the ability to create manual projects in Japanese and other
supported non-English languages
* Forks information no longer missing in running job details
* Project selected for deletion is now removed as expected when running
a project sync
* The *Admin* option in the Team Permissions is now disabled so that a
user cannot select it when it is not applicable to the available
organization(s)
* Large workflow templates no longer cause browsers to crash when
linking nodes near the end of the template
* References to Ansible Tower are replaced with Automation Controller
throughout the UI, including tooltips where documentation is referenced

*Installation fixes specific to Automation Controller:*

* Updated the Receptor to 1.2.3 everywhere as needed

=== Automation Controller Version 4.1.2

*Automation Controller fixes*:

* Upgraded Django version to 3.2 LTS
* System (management) jobs are now able to be canceled
* Rsyslog no longer needs manual intervention to send out logs after
hitting a 40x error
* Credential lookup plugins now respect the `AWX_TASK_ENV` setting
* Fixed the controller to list valid subscriptions from Satellite when
having multiple quantities from the same SKU
* Updated Receptor version to 1.2.1, which includes several fixes

*Execution Environment fixes*:

* The host trusted cert store is now exposed to execution environments
by default. See xref:ag_isolation_variables[] for detail.
* Mounting the `/etc/ssh` or `/etc/` to isolated jobs now works in
podman
* User customization of execution environment mount options and mount
paths are now supported
* Fixed SELinux context on `/var/lib/awx/.local/share/containers` and
ensure awx as podman storage
* Fixed failures to no longer occur when the semanage `fcontext` has
been already set for the expected directory

*Automation Controller UI fixes*:

* Fixed the ability to create manual projects in Japanese and other
suppported non-English languages
* Fixed the controller UI to list the roles for organizations when using
non-English web browsers
* Fixed the job output to display all job type events, including source
control update events over websockets
* Fixed the `TypeError` when running a command on a host in a smart
inventory
* Fixed the encrypted password in surveys to no longer show up as
plaintext in the Edit Order page

*Installation fixes specific to Automation Controller*:

* Fixed duplicate Galaxy credentials with no default organization
* Running the `./setup.sh -b` out of the installer directory no longer
fails to load group vars
* The installer no longer fails when IPV6 is disabled
* Fixed unnecessary `become_user:root` entries in the installation
* Modified database backup and restore logic to compress dump data
* Creating default execution environments no longer fails when password
has special characters
* Fixed installations of execution environments when installing without
internet access
* Upgrading to AAP 2.1 no longer breaks when the Django superuser is
missing
* Rekey now allowed with existing key

=== Automation Controller Version 4.1.1

* Added the ability to specify additional nginx headers
* Fixed analytics gathering to collect all the data the controller
needed to collect
* Fixed the controller to no longer break subsequent installer runs when
deleting the demo organization

=== Automation Controller Version 4.1

*Introduced*

* Connected Receptor nodes to form a control plane and execution `mesh`
configurations
* The special `controlplane` instance group to allow for the task
manager code to target an OpenShift Controller node to run the project
update
* The ability to render a configured mesh topology in a graph in the
installer
* Controller 4.1 execution nodes can be remote
* Node types for Controller 4.1 (`control, hybrid, execution, hop`,
`control`, `hybrid`, `execution`, `hop`) installed for different sets of
services and provide different capabilities, allowing for scaling nodes
that provide the desired capability such as job execution or serving of
web requests to the API/UI.

*Added*

* The ability for the platform installer to allow users to install
execution nodes and express receptor mesh topology in the inventory
file. The platform installer will also be responsible for deprovisioning
nodes.
* Work signing to the receptor mesh so that control plane nodes have the
exclusive authority to submit receptor work to execution nodes over the
mesh
* Support for pre-population of execution environment name, description,
and image from query parameters when adding a new execution environment
in the Controller User Interface
* Ability to trigger a reload of the topology configuration in Receptor
without interrupting work execution
* Using Public Key Infrastructure (PKI) for securing the Receptor mesh
* Added importing execution environments from Automation Hub into the
controller to improve the platform experience

*Updated*

* The controller to support new controller control plane and execution
mesh
* Task manager will only run project updates and system jobs on nodes
with `node_type` of "control" or "hybrid"
* Task manager will only run jobs, inventory updates, and ad hoc
commands on nodes with `node_type` of "hybrid" or "execution"
* Heartbeat and capacity check to work with Receptor execution nodes
* Reaper to work with the addition of execution nodes
* Controller User Interface to not show control instances as an option
to associate with instance groups
* The Associate pop-up screen to display host names when adding an
existing host to a group
* Validators for editing miscellaneous authentication parameters
* Advanced search key options to be grouped
* SAML variables default values
* Survey validation on Prompt on Launch
* Login redirect

*Deprecated*

* None

*Removed*

* The ability to delete the default instance group through the User
Interface

=== Automation Controller Version 4.0.1

* Upgraded Django version to 3.2 LTS
* Updated receptor to version 1.2.1

=== Automation Controller Version 4.0

*Introduced*

* Support for automation execution environments. All automation now runs
in execution environments via containers, either directly via OpenShift,
or locally via podman
* New PatternFly 4 based user-interface for increased performance,
security, and consistency with other Ansible Automation Platform
components

*Added*

* Added identity provider support for GitHub Enterprise
* Support for RHEL system crypto profiles to nginx configuration
* The ability to disable local system users and only pull users from
configured identity providers
* Additional Prometheus metrics for tracking job event processing
performance
* New `awx-manage` command for dumping host automation information
* Red Hat Insights as an inventory source
* Ability to set server-side password policies using Django's
`AUTH_PASSWORD_VALIDATORS` setting
* Support for Centrify Vault as a credential lookup plugin
* Support for namespaces in Hashicorp Vault credential plugin

*Updated*

* OpenShift deployment to be done via an Operator instead of a playbook
* Python used by application to Python 3.8
* Nginx used to version 1.18
* PostgreSQL used to PostgreSQL 12, and moved to partitioned databases
for performance
* The “container groups” feature to general availability from Tech
Preview; now fully utilizes execution environments
* Insights remediation to use new Red Hat Insights inventory source
rather than utilizing scan playbooks with arbitrary inventory
* Subscriptions display to count hosts automated on instead of hosts
imported
* Inventory source, credential, and Ansible content collection to
reference [.title-ref]#controller# instead of [.title-ref]#tower#

*Deprecated*

* None

*Removed*

* Support for deploying on CentOS (any version) and RHEL 7
* Support for Mercurial projects
* Support for custom inventory scripts stored in controller (use
`awx-manage export_custom_scripts` to export them)
* Resource profiling code (`AWX_RESOURCE_PROFILING_*`)
* Support for custom Python virtual environments for execution. Use new
`awx-manage` tools for assisting in migration
* Top-level /api/v2/job_events/ API endpoint
* The ability to disable job isolation
