[[ug_execution_environments]]
Execution Environments ===========

include::execution_environs.adoc[]

[[ug_build_ees]]
Building an Execution Environment -----------------

Using Ansible content that depends on non-default dependencies (custom
virtual environments) can be tricky. Packages must be installed on each
node, play nicely with other software installed on the host system, and
be kept in sync. Previously, jobs ran inside of a virtual environment at
`/var/lib/awx/venv/ansible` by default, which was pre-loaded with
dependencies for ansible-runner and certain types of Ansible content
used by the Ansible control machine.

To help simplify this process, container images can be built that serve
as Ansible
https://docs.ansible.com/ansible/latest/network/getting_started/basic_concepts.html#control-node[control
nodes]. These container images are referred to as automation execution
environments, which you can create with ansible-builder and then
ansible-runner can make use of those images.

== Install ansible-builder

In order to build images, either installations of podman or docker is
required along with the ansible-builder Python package. The
`--container-runtime` option needs to correspond to the Podman/Docker
executable you intend to use.

To install from PyPi:

....
$ pip install ansible-builder
....

To install from the mainline development branch:

....
$ pip install https://github.com/ansible/ansible-builder/archive/devel.zip
....

To install from a specific tag or branch, replace `<ref>` in the
following example:

....
$ pip install https://github.com/ansible/ansible-builder/archive/<ref>.zip
....

[[build_ee]]
Build an execution environment ~~~~~~~~~~~~~~~

Ansible-builder is used to create an execution environment.

An execution environment is expected to contain:

* Ansible
* Ansible Runner
* Ansible Collections
* Python and/or system dependencies of:
** modules/plugins in collections
** content in ansible-base
** custom user needs

Building a new execution environment involves a definition (a `.yml`
file) that specifies which content you would like to include in your
execution environment, such as collections, Python requirements, and
system-level packages. The content from the output generated from
migrating to execution environments has some of the required data that
can be piped to a file or pasted into this definition file. See
`migrate_new_venv` for more detail. If you did not migrate from a
virtual environment, you can create a definition file with the required
data outlined in `ref_ee_definition`.

Collection developers can declare requirements for their content by
providing the appropriate metadata. For more information, refer to
`ref_collections_metadata`.

== Run the builder

Once you created a definition, use this procedure to build your
execution environment.

The `ansible-builder build` command takes an execution environment
definition as an input. It outputs the build context necessary for
building an execution environment image, and proceeds with building that
image. The image can be re-built with the build context elsewhere, and
produces the same result. By default, it looks for a file named
`execution-environment.yml` in the current directory.

For the illustration purposes, the following example
`execution-environment.yml` file is used as a starting point:

....
---
version: 1
dependencies:
  galaxy: requirements.yml
....

The content of `requirements.yml`:

....
---
collections:
  - name: awx.awx
....

To build an execution environment using the files above, run:

....
$ ansible-builder build
...
STEP 7: COMMIT my-awx-ee
--> 09c930f5f6a
09c930f5f6ac329b7ddb321b144a029dbbfcc83bdfc77103968b7f6cdfc7bea2
Complete! The build context can be found at: context
....

In addition to producing a ready-to-use container image, the build
context is preserved, which can be rebuilt at a different time and/or
location with the tooling of your choice, such as `docker build` or
`podman build`.

Use an execution environment in jobs --------------------

Depending on whether an execution environment is made available for
global use or tied to an organization, you must have the appropriate
level of administrator privileges in order to use an execution
environment in a job. Execution environments tied to an organization
require Organization administrators to be able to run jobs with those
execution environments.

In order to use an execution environment in a job, you must have created
one using ansible-builder. See `build_ee` for detail. Once an execution
environment is created, you can use it to run jobs. Use the automation
controller user interface to specify the execution environment to use in
your job templates.

Note

Before running a job or job template that uses an execution environment
that has a credential assigned to it, be sure that the credential
contains a username, host, and password.

[arabic]
. Click *Execution Environments* from the left navigation bar.
. Add an execution environment by selecting the *Add* button.
. Enter the appropriate details into the following fields:

* *Name*: Enter a name for the execution environment (required).
* *Image*: Enter the image name (required). The image name requires its
full location (repo), the registry, image name, and version tag in the
example format of
`quay.io/ansible/awx-ee:latestrepo/project/image-name:tag`.
* *Job Type*: optionally choose the type of pull when running jobs:

______________________________________________________________________________________________
* *Always pull container before running*: Pulls the latest image file
for the container.
* *No pull option has been selected*: No pulls specified.
* *Never pull container before running*: Never pull the latest version
of the container image.
______________________________________________________________________________________________

* *Description*: optional.
* *Organization*: optionally assign the organization to specifically use
this execution environment. To make the execution environment available
for use across multiple organizations, leave this field blank.
* *Registry credential*: If the image has a protected container
registry, provide the credential to access it.

image:ee-new-ee-form-filled.png[image]

[arabic, start=4]
. Click *Save*.

Now your newly added execution environment is ready to be used in a job
template. To add an execution environment to a job template, specify it
in the *Execution Environment* field of the job template, as shown in
the example below. For more information on setting up a job template,
see xref:ug_JobTemplates[] in the Automation Controller User Guide.

image:job-template-with-example-ee-selected.png[image]

Once you added an execution environment to a job template, you can see
those templates listed in the *Templates* tab of the execution
environment:

image:ee-details-templates-list.png[image]

Execution Environment mount options ----------------------------

Rebuilding an execution environment is one way to add certs, but
inheriting certs from the host provides a more convenient solution. For
VM-based installs, the controller automatically mounts the system trust
store in the execution environment when jobs run.

Additionally, you may customize execution environment mount options and
mount paths in the *Paths to expose to isolated jobs* field of the Job
Settings page, where it supports podman-style volume mount syntax. Refer
to the
https://docs.podman.io/en/latest/markdown/podman-run.1.html#volume-v-source-volume-host-dir-container-dir-options[Podman
documentation] for detail.

In some cases where the `/etc/ssh/*` files were added to the execution
environment image due to customization of an execution environment, an
SSH error may occur. For example, exposing the
`/etc/ssh/ssh_config.d:/etc/ssh/ssh_config.d:O` path allows the
container to be mounted, but the ownership permissions are not mapped
correctly.

If you encounter this error, or have upgraded from an older version of
the controller (e.g. 3.8.x), perform the following steps:

[arabic]
. Change the container ownership on the mounted volume to `root`.
. In the *Paths to expose to isolated jobs* field of the Job Settings
page, using the current example, expose the path as such:

image:settings-paths2expose-iso-jobs.png[image]

Note

The `:O` option is only supported for directories. It is highly
recommended that you be as specific as possible, especially when
specifying system paths. Mounting `/etc` or `/usr` directly have impact
that make it difficult to troubleshoot.

This informs podman to run a command similar to the example below, where
the configuration is mounted and the `ssh` command works as expected.

....
podman run -v /ssh_config:/etc/ssh/ssh_config.d/:O ...
....

To expose isolated paths in OpenShift or Kubernetes containers as
HostPath, assume the following configuration:

image:settings-paths2expose-iso-jobs-mount-containers.png[image]

Use the *Expose host paths for Container Groups* toggle to enable it.

Once the playbook runs, the resulting Pod spec will display similar to
the example below. Note the details of the `volumeMounts` and `volumes`
sections.

image:mount-containers-playbook-run-podspec.png[image]
