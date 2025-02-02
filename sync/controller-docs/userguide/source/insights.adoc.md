[[insights]]
== Setting up Insights Remediations

Automation controller supports integration with Red Hat Insights. Once a
host is registered with Insights, it will be continually scanned for
vulnerabilities and known configuration conflicts. Each of the found
problems may have an associated fix in the form of an Ansible playbook.
Insights users create a maintenance plan to group the fixes and,
ultimately, create a playbook to mitigate the problems. Automation
controller tracks the maintenance plan playbooks via an Insights
project. Authentication to Insights via Basic Auth is backed by a
special Insights Credential, which must first be established in
automation controller. To ultimately run an Insights Maintenance Plan,
you need an Insights project, and an Insights inventory.

=== Create Insights Credential

To create a new credential for use with Insights:

[arabic]
. Click *Credentials* from the left navigation bar to access the
Credentials page.
. Click the *Add* button located in the upper right corner of the
Credentials screen.
. Enter the name of the credential to be used in the *Name* field.
. Optionally enter a description for this credential in the
*Description* field.
. In the *Organization* field, optionally enter the name of the
organization with which the credential is associated, or click the
image:search-button.png[search] button and
select it from the pop-up window.
. In the *Credential Type* field, enter *Insights* or select it from the
drop-down list.

image:credential-types-popup-window-insights.png[image]

[arabic, start=7]
. Enter a valid Insights credential in the *Username* and *Password*
fields. The Insights credential is the user's link:[Red Hat Customer
Portal] account username and password.
+

image:insights-create-with-demo-credentials.png[Credentials
- create with demo insights credentials]

[arabic, start=8]
. Click *Save* when done.

=== Create an Insights Project

To create a new Insights project:

[arabic]
. Click *Projects* from the left navigation bar to access the Projects
page.
. Click the *Add* button located in the upper right corner of the
Projects screen.
. Enter the appropriate details into the required fields, at minimum.
Note the following fields requiring specific Insights-related entries:

* *Name*: Enter the name for your Insights project.
* *Organization*: Enter the name of the organization associated with
this project, or click the
image:search-button.png[search] button and
select it from the pop-up window.
* *SCM Type*: Select *Red Hat Insights*.
* Upon selecting the SCM type, the *Source Details* field expands.

[arabic, start=4]
. The *Credential* field is pre-populated with the Insights credential
you previously created. If not, enter the credential, or click the
image:search-button.png[search] button and
select it from the pop-up window.
. Click to select the update option(s) for this project from the
*Options* field, and provide any additional values, if applicable. For
information about each option, click the tooltip
image:tooltips-icon.png[tooltip] next to the
options.

image:insights-create-project-insights-form.png[Insights
- create demo insights project form]

[arabic, start=6]
. Click *Save* when done.

All SCM/Project syncs occur automatically the first time you save a new
project. However, if you want them to be updated to what is current in
Insights, manually update the SCM-based project by clicking the
image:update-button.png[update] button under
the project's available Actions.

This process syncs your Insights project with your Insights account
solution. Notice that the status dot beside the name of the project
updates once the sync has run.

image:insights-create-project-insights-succeed.png[Insights
- demo insights project success]

=== Create Insights Inventory

The Insights playbook contains a [.title-ref]#hosts:# line where the
value is the hostname that Insights itself knows about, which may be
different than the hostname that Tower knows about. To use an Insights
playbook, you will need an Insights inventory.

To create a new inventory for use with Insights, see
xref:ug_source_insights[].

=== Remediate Insights Inventory

Remediation of an Insights inventory allows Tower to run Insights
playbooks with a single click. This is done by creating a Job Template
to run the Insights remediation.

[arabic]
. Click *Job Templates* from the left navigation bar to access the Job
Templates page.
. Create a new Job Template, with the appropriate details into the
required fields, at minimum. Note the following fields requiring
specific Insights-related entries:

* *Name*: Enter the name of your Maintenance Plan.
* *Job Type*: If not already populated, select *Run* from the drop-down
menu list.
* *Inventory*: Select the Insights Inventory you previously created.
* *Project*: Select the Insights project you previously created.
* *Playbook*: Select a playbook associated with the Maintenance Plan you
want to run from the drop-down menu list.
* *Credential*: Enter the credential to use for this project or click
the image:search-button.png[search] button
and select it from the pop-up window. The credential does not have to be
an Insights credential.
* *Verbosity*: Keep the default setting, or select the desired verbosity
from the drop-down menu list.

image:insights-create-new-job-template-maintenance-plan-filled.png[Insights
- maintenance plan template filled]

[arabic, start=3]
. Click *Save* when done.
. Click the image:launch-button.png[launch]
icon to launch the job template.

Once complete, the job results display in the Job Details page.
