[[ag_ent_auth]]
== Setting up Enterprise Authentication

This section describes setting up authentication for the following
enterprise systems:

Note

For LDAP authentication, see xref:ag_auth_ldap[].

SAML, RADIUS, and TACACS+ users are categorized as 'Enterprise' users.
The following rules apply to Enterprise users:

* Enterprise users can only be created via the first successful login
attempt from remote authentication backend.
* Enterprise users cannot be created/authenticated if non-enterprise
users with the same name has already been created in the controller.
* The controller passwords of enterprise users should always be empty
and cannot be set by any user if there are enterprise backend-enabled.
* If enterprise backends are disabled, an enterprise user can be
converted to a normal controller user by setting the password field.
However, this operation is irreversible, as the converted controller
user can no longer be treated as enterprise user.

[[ag_auth_azure]]
=== Azure AD settings

To set up enterprise authentication for Microsoft Azure Active Directory
(AD), you will need to obtain an OAuth2 key and secret by registering
your organization-owned application from Azure at
https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app.
Each key and secret must belong to a unique application and cannot be
shared or reused between different authentication backends. In order to
register the application, you must supply it with your webpage URL,
which is the Callback URL shown in the Settings Authentication screen.

[arabic]
. Click *Settings* from the left navigation bar.
. On the left side of the Settings window, click *Azure AD settings*
from the list of Authentication options.
. The *Azure AD OAuth2 Callback URL* field is already pre-populated and
non-editable. Once the application is registered, Azure displays the
Application ID and Object ID.
. Click *Edit* and copy and paste Azure's Application ID to the *Azure
AD OAuth2 Key* field.
+
Following Azure AD's documentation for connecting your app to Microsoft
Azure Active Directory, supply the key (shown at one time only) to the
client for authentication.
. Copy and paste the actual secret key created for your Azure AD
application to the *Azure AD OAuth2 Secret* field of the Settings -
Authentication screen.
. For details on completing the mapping fields, see xref:ag_org_team_maps[].
. Click *Save* when done.
. To verify that the authentication was configured correctly, logout of
automation controller and the login screen will now display the
Microsoft Azure logo to allow logging in with those credentials.

image:configure-tower-auth-azure-logo.png[image]

For application registering basics in Azure AD, refer to the
https://docs.microsoft.com/en-us/azure/active-directory/develop/v2-overview[Azure
AD Identity Platform (v2)] overview.

=== LDAP Authentication

Refer to the xref:ag_auth_ldap[] section.

[[ag_auth_radius]]
=== RADIUS settings

automation controller can be configured to centrally use RADIUS as a
source for authentication information.

[arabic]
. Click *Settings* from the left navigation bar.
. On the left side of the Settings window, click *RADIUS settings* from
the list of Authentication options.
. Click *Edit* and enter the Host or IP of the Radius server in the
*Radius Server* field. If this field is left blank, Radius
authentication is disabled.
. Enter the port and secret information in the next two fields.
. Click *Save* when done.

[[ag_auth_saml]]
=== SAML settings

SAML allows the exchange of authentication and authorization data
between an Identity Provider (IdP - a system of servers that provide the
Single Sign On service) and a Service Provider (in this case, automation
controller). automation controller can be configured to talk with SAML
in order to authenticate (create/login/logout) controller users. User
Team and Organization membership can be embedded in the SAML response to
the controller.

image:configure-tower-auth-saml-topology.png[image]

The following instructions describe automation controller as the service
provider. To authenticate users through RHSSO (keycloak), refer to the
https://www.ansible.com/blog/red-hat-single-sign-on-integration-with-ansible-tower[Red
Hat Single Sign On Integration with the Automation Controller] blog.

To setup SAML authentication:

[arabic]
. Click *Settings* from the left navigation bar.
. On the left side of the Settings window, click *SAML settings* from
the list of Authentication options.
. The *SAML Assertion Consume Service (ACS) URL* and *SAML Service
Provider Metadata URL* fields are pre-populated and are non-editable.
Contact the Identity Provider administrator and provide the information
contained in these fields.
. Click *Edit* and set the *SAML Service Provider Entity ID* to be the
same as the *Base URL of the controller host* field that can be found in
the Miscellaneous System settings screen by clicking *Settings* from the
left navigation bar. Through the API, it can be viewed in the
`/api/v2/settings/system`, under the `CONTROLLER_BASE_URL` variable. The
Entity ID can be set to any one of the individual controller cluster
nodes, but it is good practice to set it to the URL of the Service
Provider. Ensure that the Base URL matches the FQDN of the load balancer
(if used).

Note

The Base URL is different for each node in a cluster. Commonly, a load
balancer will sit in front of many controller cluster nodes to provide a
single entry point, the controller Cluster FQDN. The SAML Service
Provider must be able establish an outbound connection and route to the
controller Cluster Node or the controller Cluster FQDN set in the SAML
Service Provider Entity ID.

In this example, the Service Provider is the controller cluster, and
therefore, the ID is set to the controller Cluster FQDN.

image:configure-tower-auth-saml-spentityid.png[image]

[arabic, start=5]
. Create a server certificate for the Ansible cluster. Typically when an
Ansible cluster is configured, the controller nodes will be configured
to handle HTTP traffic only and the load balancer will be an SSL
Termination Point. In this case, an SSL certificate is required for the
load balancer, and not for the individual controller Cluster Nodes. SSL
can either be enabled or disabled per individual controller node, but
should be disabled when using an SSL terminated load balancer. It is
recommended to use a non-expiring self signed certificate to avoid
periodically updating certificates. This way, authentication will not
fail in case someone forgets to update the certificate.

Note

The *SAML Service Provider Public Certificate* field should contain the
entire certificate, including the "-----BEGIN CERTIFICATE-----" and
"-----END CERTIFICATE-----".

If you are using a CA bundle with your certificate, include the entire
bundle in this field.

image:configure-tower-auth-saml-cert.png[image]

As an example for public certs:

....
-----BEGIN CERTIFICATE——
... cert text ...
-----END CERTIFICATE——
....

[arabic, start=6]
. Create an optional private key for the controller to use as a service
provider (SP) and enter it in the *SAML Service Provider Private Key*
field.

As an example for private keys:

....
-----BEGIN PRIVATE KEY--
... key text ...
-----END PRIVATE KEY——
....

[arabic, start=7]
. Provide the IdP with some details about the controller cluster during
the SSO process in the *SAML Service Provider Organization Info* field.

....
{
  "en-US": {
    "url": "http://www.example.com",
    "displayname": "Example",
    "name": "example"
  }
}
....

For example:

image:configure-tower-auth-saml-org-info.png[image]

Note

These fields are required in order to properly configure SAML within the
controller.

[arabic, start=8]
. Provide the IdP with the technical contact information in the *SAML
Service Provider Technical Contact* field. Do not remove the contents of
this field.

....
{
"givenName": "Some User",
"emailAddress": "suser@example.com"
}
....

For example:

image:configure-tower-auth-saml-techcontact-info.png[image]

[arabic, start=9]
. Provide the IdP with the support contact information in the *SAML
Service Provider Support Contact* field. Do not remove the contents of
this field.

....
{
"givenName": "Some User",
"emailAddress": "suser@example.com"
}
....

For example:

image:configure-tower-auth-saml-suppcontact-info.png[image]

[arabic, start=10]
. In the *SAML Enabled Identity Providers* field, provide information on
how to connect to each Identity Provider listed. The controller expects
the following SAML attributes in the example below:

....
Username(urn:oid:0.9.2342.19200300.100.1.1)
Email(urn:oid:0.9.2342.19200300.100.1.3)
FirstName(urn:oid:2.5.4.42)
LastName(urn:oid:2.5.4.4)
....

If these attributes are not known, map existing SAML attributes to
lastname, firstname, email and username.

Configure the required keys for each IDp:

____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
* `attr_user_permanent_id` - the unique identifier for the user. It can
be configured to match any of the attribute sent from the IdP. Usually,
it is set to `name_id` if `SAML:nameid` attribute is sent to the
controller node or it can be the username attribute, or a custom unique
identifier.
* `entity_id` - the Entity ID provided by the Identity Provider
administrator. The admin creates a SAML profile for the controller and
it generates a unique URL.
* `url` - the Single Sign On (SSO) URL the controller redirects the user
to, when SSO is activated.
* `x509_cert` - the certificate provided by the IdP admin generated from
the SAML profile created on the Identity Provider. Remove the
`--BEGIN CERTIFICATE--` and `--END CERTIFICATE--` headers, then enter
the cert as one non-breaking string.
____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

_________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
Multiple SAML IdPs are supported. Some IdPs may provide user data using
attribute names that differ from the default OIDs
(https://github.com/omab/python-social-auth/blob/master/social/backends/saml.py).
The SAML `NameID` is a special attribute used by some Identity Providers
to tell the Service Provider (the controller cluster) what the unique
user identifier is. If it is used, set the `attr_user_permanent_id` to
`name_id` as shown in the example. Other attribute names may be
overridden for each IdP as shown below.
_________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________

....
{
"myidp": {
  "entity_id": "https://idp.example.com",
  "url": "https://myidp.example.com/sso",
  "x509cert": ""
},
"onelogin": {
  "entity_id": "https://app.onelogin.com/saml/metadata/123456",
  "url": "https://example.onelogin.com/trust/saml2/http-post/sso/123456",
  "x509cert": "",
  "attr_user_permanent_id": "name_id",
  "attr_first_name": "User.FirstName",
  "attr_last_name": "User.LastName",
  "attr_username": "User.email",
  "attr_email": "User.email"
  }
}
....

image:configure-tower-auth-saml-idps.png[image]

Warning

Do not create a SAML user that shares the same email with another user
(including a non-SAML user). Doing so will result in the accounts being
merged. Be aware that this same behavior exists for System Admin users,
thus a SAML login with the same email address as the System Admin user
will login with System Admin privileges. For future reference, you can
remove (or add) Admin Privileges based on SAML mappings, as described in
subsequent steps.

Note

The IdP provides the email, last name and firstname using the well known
SAML urn. The IdP uses a custom SAML attribute to identify a user, which
is an attribute that the controller is unable to read. Instead, the
controller can understand the unique identifier name, which is the URN.
Use the URN listed in the SAML “Name” attribute for the user attributes
as shown in the example below.

image:configure-tower-auth-saml-idps-urn.png[image]

[arabic, start=11]
. Optionally provide the *SAML Organization Map*. For further detail,
see xref:ag_org_team_maps[].
. The controller can be configured to look for particular attributes
that contain Team and Organization membership to associate with users
when they log into the controller. The attribute names are defined in
the *SAML Organization Attribute Mapping* and the *SAML Team Attribute
Mapping* fields.

*Example SAML Organization Attribute Mapping*

Below is an example SAML attribute that embeds user organization
membership in the attribute _member-of_.

....
<saml2:AttributeStatement>
    <saml2:Attribute FriendlyName="member-of" Name="member-of"
NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified">
        <saml2:AttributeValue>Engineering</saml2:AttributeValue>
        <saml2:AttributeValue>IT</saml2:AttributeValue>
        <saml2:AttributeValue>HR</saml2:AttributeValue>
        <saml2:AttributeValue>Sales</saml2:AttributeValue>
    </saml2:Attribute>
    <saml2:Attribute FriendlyName="admin-of" Name="admin-of" 
NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified">
        <saml2:AttributeValue>Engineering</saml2:AttributeValue>
    </saml2:Attribute>
</saml2:AttributeStatement> 
....

Below is the corresponding controller configuration.

....
{
  "saml_attr": "member-of",
  "saml_admin_attr": "admin-of",
  "remove": true,
  "remove_admins": false
}
....

`saml_attr`: is the SAML attribute name where the organization array can
be found and `remove` is set to *True* to remove a user from all
organizations before adding the user to the list of Organizations. To
keep the user in whatever Organization(s) they are in while adding the
user to the Organization(s) in the SAML attribute, set `remove` to
*False*.

`saml_admin_attr`: Similar to the `saml_attr` attribute, but instead of
conveying organization membership, this attribute conveys admin
organization permissions.

*Example SAML Team Attribute Mapping*

Below is another example of a SAML attribute that contains a Team
membership in a list.

....
<saml:AttributeStatement>
     <saml:Attribute
        xmlns:x500="urn:oasis:names:tc:SAML:2.0:profiles:attribute:X500"
        x500:Encoding="LDAP"
        NameFormat="urn:oasis:names:tc:SAML:2.0:attrname-format:uri"
        Name="urn:oid:1.3.6.1.4.1.5923.1.1.1.1"
        FriendlyName="eduPersonAffiliation">
        <saml:AttributeValue
            xsi:type="xs:string">member</saml:AttributeValue>
        <saml:AttributeValue
            xsi:type="xs:string">staff</saml:AttributeValue>
        </saml:Attribute>
</saml:AttributeStatement>
....

....
{
    "saml_attr": "eduPersonAffiliation",
    "remove": true,
    "team_org_map": [
    {
        "team": "member",
        "organization": "Default1"
    },
    {
        "team": "staff",
        "organization": "Default2"
    }
  ]
}
....

* `saml_attr`: The SAML attribute name where the team array can be
found.
* `remove`: Set `remove` to *True* to remove user from all Teams before
adding the user to the list of Teams. To keep the user in whatever
Team(s) they are in while adding the user to the Team(s) in the SAML
attribute, set `remove` to *False*.
* `team_org_map`: An array of dictionaries of the form
`{ "team": "<AWX Team Name>", "organization": "<AWX Org Name>" }` that
defines mapping from controller Team -> controller Organization. This is
needed because the same named Team can exist in multiple Organizations
in the controller. The organization to which a team listed in a SAML
attribute belongs to, would be ambiguous without this mapping.

You could create an alias to override both Teams and Orgs in the *SAML
Team Attribute Mapping*. This option becomes very handy in cases when
the SAML backend sends out complex group names, like in the example
below:

....
{
 "remove": false,
 "team_org_map": [
  {
   "team": "internal:unix:domain:admins",
   "organization": "Default",
   "team_alias": "Administrators"
  },
  {
   "team": "Domain Users",
   "organization_alias": "OrgAlias",
   "organization": "Default"
  }
 ],
 "saml_attr": "member-of"
}
....

Once the user authenticates, the controller creates organization and
team aliases, as expected.

[arabic, start=13]
. Optionally provide team membership mapping in the *SAML Team Map*
field. For further detail, see xref:ag_org_team_maps[].
. Optionally provide security settings in the *SAML Security Config*
field. This field is the equivalent to the
`SOCIAL_AUTH_SAML_SECURITY_CONFIG` field in the API. Refer to the
https://github.com/onelogin/python-saml#settings[OneLogin's SAML Python
Toolkit] for further detail.

The controller uses the `python-social-auth` library when users log in
through SAML. This library relies on the `python-saml` library to make
available the settings for the next two optional fields, *SAML Service
Provider Extra Configuration Data* and *SAML IDP to EXTRA_DATA Attribute
Mapping*.

[arabic, start=15]
. The *SAML Service Provider Extra Configuration Data* field is
equivalent to the `SOCIAL_AUTH_SAML_SP_EXTRA` in the API. Refer to the
https://github.com/onelogin/python-saml#settings[python-saml library
documentation] to learn about the valid service provider extra
(`SP_EXTRA`) parameters.
. The *SAML IDP to EXTRA_DATA Attribute Mapping* field is equivalent to
the `SOCIAL_AUTH_SAML_EXTRA_DATA` in the API. See Python's
http://python-social-auth-docs.readthedocs.io/en/latest/backends/saml.html#advanced-settings[SAML
Advanced Settings] documentation for more information.

[[ag_auth_saml_user_flags_attr_map]]
[arabic, start=17]
. The *SAML User Flags Attribute Mapping* field allows you to map SAML
roles and attributes to special user flags. The following attributes are
valid in this field:

* `is_superuser_role`: Specifies one or more SAML roles which will grant
a user the superuser flag
* `is_superuser_attr`: Specifies a SAML attribute which will grant a
user the superuser flag
* `is_superuser_value`: Specifies one or more values required for
`is_superuser_attr` that is required for the user to be a superuser
* `remove_superusers`: Boolean indicating if the superuser flag should
be removed for users or not. Defaults to `true`. (See below for more
details)
* `is_system_auditor_role`: Specifies one or more SAML roles which will
grant a user the system auditor flag
* `is_system_auditor_attr`: Specifies a SAML attribute which will grant
a user the system auditor flag
* `is_system_auditor_value`: Specifies one or more values required for
`is_system_auditor_attr` that is required for the user to be a system
auditor
* `remove_system_auditors`: Boolean indicating if the system_auditor
flag should be removed for users or not. Defaults to `true`. (See below
for more details)

The `role` and `value` fields are lists and are [.title-ref]#or# logic.
So if you specify two roles: [.title-ref]#[ "Role 1", "Role 2" ]# and
the SAML user has either role the logic will consider them to have the
required role for the flag. This is the same with the `value` field, if
you specify: [.title-ref]#[ "Value 1", "Value 2"]# and the SAML user has
either value for their attribute the logic will consider their attribute
value to have matched.

If `role` and `attr` are both specified for either `superuser` or
`system_auditor`, the settings for `attr` will take precedence over a
`role`. System Admin and System Auditor roles are evaluated at login for
a SAML user. If you grant a SAML user one of these roles through the UI
and not through the SAML settings, the roles will be removed on the
user's next login unless the `remove` flag is set to false. The remove
flag, if `false`, will never allow the SAML adapter to remove the
corresponding flag from a user. The following table describes how the
logic works.

[width="100%",cols="25%,11%,27%,12%,14%,11%",options="header",]
|=======================================================================
|Has one or more roles |Has Attr |Has one or more Attr Values |Remove
Flag |Previous Flag |Is Flagged
|No |No |N/A |True |False |No

|No |No |N/A |False |False |No

|No |No |N/A |True |True |No

|No |No |N/A |False |True |Yes

|Yes |No |N/A |True |False |Yes

|Yes |No |N/A |False |False |Yes

|Yes |No |N/A |True |True |Yes

|Yes |No |N/A |False |True |Yes

|No |Yes |Yes |True |False |Yes

|No |Yes |Yes |False |False |Yes

|No |Yes |Yes |True |True |Yes

|No |Yes |Yes |False |True |Yes

|No |Yes |No |True |False |No

|No |Yes |No |False |False |No

|No |Yes |No |True |True |No

|No |Yes |No |False |True |Yes

|No |Yes |Unset |True |False |Yes

|No |Yes |Unset |False |False |Yes

|No |Yes |Unset |True |True |Yes

|No |Yes |Unset |False |True |Yes

|Yes |Yes |Yes |True |False |Yes

|Yes |Yes |Yes |False |False |Yes

|Yes |Yes |Yes |True |True |Yes

|Yes |Yes |Yes |False |True |Yes

|Yes |Yes |No |True |False |No

|Yes |Yes |No |False |False |No

|Yes |Yes |No |True |True |No

|Yes |Yes |No |False |True |Yes

|Yes |Yes |Unset |True |False |Yes

|Yes |Yes |Unset |False |False |Yes

|Yes |Yes |Unset |True |True |Yes

|Yes |Yes |Unset |False |True |Yes
|=======================================================================

Each time a SAML user authenticates to automation controller, these
checks will be performed and the user flags will be altered as needed.
If `System Administrator` or `System Auditor` is set for a SAML user
within the UI, the SAML adapter will override the UI setting based on
the rules above. If you would prefer that the user flags for SAML users
do not get removed when a SAML user logs in, you can set the `remove_`
flag to `false`. With the remove flag set to `false`, a user flag set to
`true` through either the UI, API or SAML adapter will not be removed.
However, if a user does not have the flag, and the above rules determine
the flag should be added, it will be added, even if the flag is `false`.

Example:

....
{
    "is_superuser_attr": "blueGroups",
    "is_superuser_role": "is_superuser",
    "is_superuser_value": "cn=My-Sys-Admins,ou=memberlist,ou=mygroups,o=myco.com",
    "is_system_auditor_attr": "blueGroups",
    "is_system_auditor_role": "is_system_auditor",
    "is_system_auditor_value": "cn=My-Auditors,ou=memberlist,ou=mygroups,o=myco.com"
}
....

[arabic, start=18]
. Click *Save* when done.
. To verify that the authentication was configured correctly, load the
auto-generated URL found in the *SAML Service Provider Metadata URL*
into a browser. It should output XML output, otherwise, it is not
configured correctly.
+
Alternatively, logout of automation controller and the login screen will
now display the SAML logo to indicate it as a alternate method of
logging into automation controller.
+
image:configure-tower-auth-saml-logo.png[image]

==== Transparent SAML Logins

For transparent logins to work, you must first get IdP-initiated logins
to work. To achieve this:

[arabic]
. Set the `RelayState` on the IdP to the key of the IdP definition in
the `SAML Enabled Identity Providers` field as previously described. In
the example given above, `RelayState` would need to be either `myidp` or
`onelogin`.
. Once this is working, specify the redirect URL for non-logged-in users
to somewhere other than the default controller login page by using the
*Login redirect override URL* field in the Miscellaneous Authentication
settings window of the *Settings* menu, accessible from the left
navigation bar. This should be set to
`/sso/login/saml/?idp=<name-of-your-idp>` for transparent SAML login, as
shown in the example.

image:configure-tower-system-login-redirect-url.png[image]

Note

The above is a sample of a typical IdP format, but may not be the
correct format for your particular case. You may need to reach out to
your IdP for the correct transparent redirect URL as that URL is not the
same for all IdPs.

[arabic, start=3]
. After transparent SAML login is configured, to log in using local
credentials or a different SSO, go directly to
`https://<your-tower-server>/login`. This provides the standard
controller login page, including SSO authentication buttons, and allows
you to log in with any configured method.

==== Enabling Logging for SAML

You can enable logging messages for the SAML adapter the same way you
can enable logging for LDAP. Refer to the `ldap_logging` section.

[[ag_auth_tacacs]]
=== TACACS+ settings

Terminal Access Controller Access-Control System Plus (TACACS+) is a
protocol that handles remote authentication and related services for
networked access control through a centralized server. In particular,
TACACS+ provides authentication, authorization and accounting (AAA)
services, in which you can configure automation controller to use as a
source for authentication.

Note

This feature is deprecated and will be removed in a future release.

[arabic]
. Click *Settings* from the left navigation bar.
. On the left side of the Settings window, click *TACACs+ settings* from
the list of Authentication options.
. Click *Edit* and enter information in the following fields:

* *TACACS+ Server*: Provide the hostname or IP address of the TACACS+
server with which to authenticate. If this field is left blank, TACACS+
authentication is disabled.
* *TACACS+ Port*: TACACS+ uses port 49 by default, which is already
pre-populated.
* *TACACS+ Secret*: Secret key for TACACS+ authentication server.
* *TACACS+ Auth Session Timeout*: Session timeout value in seconds. The
default is 5 seconds.
* *TACACS+ Authentication Protocol*: The protocol used by TACACS+
client. Options are *ascii* or *pap*.

image:configure-tower-auth-tacacs.png[image]

[arabic, start=4]
. Click *Save* when done.
