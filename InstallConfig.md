# Installation and Configuration Guide

## Prerequisites
The miminum requirements for RH SSO 7.3 are:

- Java 8 JDK
- zip or gzip or tar binaries
- at least 512 MiB of RAM
- at least 1GiB of disk space
- a shared external DB. This is required to run RH-SSO in cluster mode. In
  single instance mode RH-SSO runs using an H2 database (not suitable for
  production).
- network multicast support, required for cluster mode. This is used by the
  jgroups subsystem to announce the nodes in the cluster.
- on Linux, use `/dev/urandom` for RNG. To set this the java system property
  `java.security.egd` must be set on startup to the value `file:/dev/urandom`.

## Install modes
There are 2 main installation modes: from zip file or from rpm. The container
native approach will be described later.

### Installation from Zip file
The zip file named **rh-sso-7.3.0.GA.zip** must be extracted to the desired
path using the **unzip** or **gunzip** utility.

```
# unzip rh-sso-7.3.0.GA.zip -d /opt
```

This will create the `/opt/rh-sso-7.3` folder.

### Installation from RPM
The RPM method is preferred in many cases. The **RH-SSO 7.3** and the **JBoss
EAP 7.2** repositories must be enabled before installing.

```
# subscription-manager repos --enable=jb-eap-7.2-for-rhel-<RHEL_VERSION>-server-rpms --enable=rhel-<RHEL_VERSION>-server-rpms
# subscription-manager repos --enable=rh-sso-7.3-for-rhel-<RHEL-VERSION>-server-rpms
```

RH-SSO is installed as a package group:

```
# yum groupinstall rh-sso7
```

The default installation path is `/opt/rh/rh-sso7/root/usr/share/keycloak`.

### Installed directory tree
The installed directory tree is similar to a JBoss EAP installation, with the
**bin/** directory containing the startup scripts and utilities, the
**standalone/** directory containing the standalone mode config, data and log
files, the **domain/** directory containing the domain mode files and the
**modules/** directory containing the installed modules:

```
# ls -al /opt/rh-sso-7.3
drwxr-x--x. 4 keycloak keycloak   4096 Jan  9 23:11 bin
drwxr-xr-x. 4 keycloak keycloak     45 Jan  9 23:11 docs
drwxr-x--x. 5 keycloak keycloak     50 Jan  9 23:11 domain
-rw-r--r--. 1 keycloak keycloak    419 Jan  9 23:11 JBossEULA.txt
-rw-r--r--. 1 keycloak keycloak 480958 Jan  9 23:11 jboss-modules.jar
-rw-r--r--. 1 keycloak keycloak  10637 Jan  9 23:11 License.html
-rw-r--r--. 1 keycloak keycloak  26530 Jan  9 23:11 LICENSE.txt
drwxr-xr-x. 3 keycloak keycloak     39 Jan  9 23:11 modules
drwxr-x--x. 8 keycloak keycloak     91 Mar 28 22:33 standalone
drwxr-xr-x. 5 keycloak keycloak     66 Jan  9 23:11 themes
-rw-r--r--. 1 keycloak keycloak     42 Jan  9 22:59 version.txt
drwxr-xr-x. 2 keycloak keycloak     42 Jan  9 23:11 welcome-content
```

## Startup modes
There are three possible startup modes for RH-SSO:
- **Standalone** mode, for single PoC installations and scenarios where no
  clustering is needed. Not recommended for production scenarios.
- **Standalone clustered** mode. In this scenario different standalone
  instances are clustered together. A share database is mandatory.
- **Domain clustered** mode. This approach simplifies the management of many
  instances using the JBoss EAP managed domain approach.

### Standalone mode
To start a single standalone instance of RH-SSO run the `standalone.sh` script
in the `bin/` folder:

```
# /opt/rh-sso-7.3/bin/standalone.sh
```

By default RH-SSO starts binding to the loopback interface 127.0.0.1. To bind
to a different address use the `-b` and - optionally - `-bmanagement` options:

```
# /opt/rh-sso-7.3/bin/standalone.sh -b 0.0.0.0
```

Sometimes it is useful to run RH-SSH with a custom port offset. To achieve this
goal use the `-Djboss.socket.binding.port-offset` option:

```
# /opt/rh-sso-7.3/bin/standalone.sh -b 0.0.0.0 -Djboss.socket.binding.port-offset=100
```

The standalone mode uses the `.../standalone/configuration/standalone.xml`
config file.

### Standalone clustered mode
In clustered mode the RH-SSO instances are clustered together using **Jgroups**.
A dedicated cache container is also enabled in the **Infinispan** subsystem.
This mode uses the `…​/standalone/configuration/standalone-ha.xml` config file
in the clustered instances.
Clustered RH-SSH instances usually reside in different hosts but can be executed
in the same host using the above mentioned port-offset approach.

To start an instance in clustered mode:
```
# /opt/rh-sso-7.3/bin/standalone.sh --server-config=standalone-ha.xml
```

Once again, this command starts the standalone instance binding it to the
loopback interfaces. To bind it on a different address use the `-b` and
`-bmanagement` options.

```
# /opt/rh-sso-7.3/bin/standalone.sh --server-config=standalone-ha.xml -b 0.0.0.0
```

### Domain clustered mode
In domain clustered mode RH-SSO instances can be centrally managed. This is
a key improvements versus the standalone clustered mode where the instances must
be managed separately.  
In domain mode a central domain controller an separated host controllers manage
and run the server instances, organized in **server groups**.

#### Profiles
Domain mode offers thee different profiles, available in the configuration file
`.../domain/configuration/domain.xml`. The profiles are named respectively
**auth-server-standalone**, **auth-server-clustered** and **load-balancer**.  
- the **auth-server-standalone** can be used to implement the
domain mode without clustering.  
- **auth-server-clustered** profile is used to run the instances in clustered
mode.  
- the **load-balancer** profile is used to manage the load balancer instances,
frontend of the Keycloak instances. Load balancers are not mandatory and can be
safely disabled and replaced with external hardware or software solutions.

#### Server groups
Two server groups are defined in the `domain.xml`:
- **load-balancer-group**, bound to the **load-balancer** profile.
- **auth-server-group** bound to the **auth-server-clustered** profile.

### Host config files
Hosts in domain mode have the purpose of running the server instances. By
default the RH-SSO installation provides two host files already suitable for
a PoC domain installation:
- `.../domain/configuration/host-master.xml`. The host-master is executed in the
  domain controller and runs two server instances by default: **load-balancer**
  and **server-one**. Users are free to remove the **load-balancer** for better
  production environments and replace **server-one**.
- `.../domain/configuration/host-slave.xml`. This file is executed by a slave
  host and runs a **server-two** instance.

Both **server-one** and **server-two** are executed in the **auth-server-clutered**
server group.  


> **_NOTE:_** The purpose of this configurations is to provide a simple out-of-the-box scenario. For production environments an architectural planning
> of the needed instances and their distribution is mandatory.

#### Domain Startup
In domain mode the domain controller must be started first. To start the domain controller:

```
# .../bin/domain.sh --host-config=host-master.xml
```

By default the host controller binds to 127.0.0.1. To bind its management
interface to a different address:

```
# .../bin/domain.sh --host-config=host-master.xml Djboss.bind.address.management=192.168.10.20
```

The domain controller will consume the `host-master.xml` file.

After the domain controller slave host can be started. If the hosts reside on
separate machines (most common scenario) an AS user belonging to the
**ManagementRealm** security realm must be created and its credentials added in
the `host-slave.xml` config file. This is **NOT** a RH-SSO user but a JBoss EAP
user whose only purpose is to authenticate the slave hosts to the controller.

To create the AS user run the command `.../bin/add-user.sh`. The following
example adds an **admin** user to the **ManagementRealm**:

```
$ add-user.sh
 What type of user do you wish to add?
  a) Management User (mgmt-users.properties)
  b) Application User (application-users.properties)
 (a): a
 Enter the details of the new user to add.
 Using realm 'ManagementRealm' as discovered from the existing property files.
 Username : admin
 Password recommendations are listed below. To modify these restrictions edit the add-user.properties configuration file.
  - The password should not be one of the following restricted values {root, admin, administrator}
  - The password should contain at least 8 characters, 1 alphabetic character(s), 1 digit(s), 1 non-alphanumeric symbol(s)
  - The password should be different from the username
 Password :
 Re-enter Password :
 What groups do you want this user to belong to? (Please enter a comma separated list, or leave blank for none)[ ]:
 About to add user 'admin' for realm 'ManagementRealm'
 Is this correct yes/no? yes
 Added user 'admin' to file '/.../standalone/configuration/mgmt-users.properties'
 Added user 'admin' to file '/.../domain/configuration/mgmt-users.properties'
 Added user 'admin' with groups to file '/.../standalone/configuration/mgmt-groups.properties'
 Added user 'admin' with groups to file '/.../domain/configuration/mgmt-groups.properties'
 Is this new user going to be used for one AS process to connect to another AS process?
 e.g. for a slave host controller connecting to the master or for a Remoting connection for server to server EJB calls.
 yes/no? yes
 To represent the user add the following to the server-identities definition <secret value="bWdtdDEyMyE=" />
```

The last line is the most important:

```
 To represent the user add the following to the server-identities definition <secret value="bWdtdDEyMyE=" />
```

The xml-encoded secret must be copied inside the `host-slave.xml` file in the
**ManagementRealm** configuration:

```
     <management>
         <security-realms>
             <security-realm name="ManagementRealm">
                 <server-identities>
                     <secret value="bWdtdDEyMyE="/>
                 </server-identities>
```

In the same `host-slave.xml` file the `<domain-controller>` section must be
updated with the user name:


```
    <domain-controller>
        <remote username="admin" security-realm="ManagementRealm">
            <discovery-options>
                <static-discovery name="primary" protocol="${jboss.domain.master.protocol:remote}" host="${jboss.domain.master.address:192.168.10.20}" port="${jboss.domain.master.port:9999}"/>
            </discovery-options>
        </remote>
    </domain-controller>
```

The host slaves can now be started:

```
# ../bin/domain.sh --host-config=host-slave.xml
```

On boot the host slaves will be bound to the controller (master) and will
receive informations about the subsystems and server groups configurations,
propagating them to the server instances.

### Data persistence and caching in clustered mode
When operating in clustered mode, both standalone and domain, some special
considerations must be applied.  

#### Caching
If the instances are spawned across multiple datacenters with multiple clusters
and active/active mode is desired a distributed cache like **JBoss Data Grid**
must be used in order to guarantee the availability of cached data across
multiple datacenters. Stretched clusters across multiple datacenters are not
an option in this scenario.
RH-SSO uses a dedicated Infinispan cache-container named **keycloak** to cache persistent data to optimize performances. The cache-container holds the
following caches, which need to be replicated across the datacenters:

- Realms
- Users
- Sessions
- Authentication Sessions
- Offline Sessions
- Client Sessions
- Offline Client Sessions
- Login Failures
- Work
- Authorizations
- Keys
- Action Tokens

#### Databases
In clustered mode a external database is always mandatory. When operating across
multiple datacenters the database must be synchronously replicated in
active/active mode across regions.

## Configuration

### SPI Providers
RH-SSO is a modular system that allows administrators to implement new features
using **Service Provider Interfaces (SPI)**.
SPI are configured in the **keycloak-server** subsystem and provide the means to
configure the behavior of RH-SSO.

An interesting example is the **connectionJPA** SPI.
It configures the connections to the persistence using **JPA** (Java
Persistence API).  

A fresh installation should look like this:

```
<spi name="connectionsJpa">
    <provider name="default" enabled="true">
        <properties>
            <property name="dataSource" value="java:jboss/datasources/KeycloakDS"/>
            <property name="initializeEmpty" value="true"/>
            <property name="migrationStrategy" value="update"/>
            <property name="migrationExport" value="${jboss.home.dir}/keycloak-database-update.sql"/>
        </properties>
    </provider>
</spi>
```

By default RH-SSO uses an H2 database whose JNDI name is the value of the
**dataSource** property above: `java:jboss/datasources/KeycloakDS`. This
dataSource JNDI name is the one expected by the JPA **entityManager**. In order to update the database connection the EAP `datasource` subsystem must be modified.  

The **initializeEmpty** property tells RH-SSO to initialize the database if
empty.

The **migrationStrategy** property defines the migration strategy for the
database: it can be `update`, `manual` or `validate`. The **migrationExport**
is the path where the SQL migration file should be written.

Among the disabled properties there is one useful for debugging but very verbose
and discouraged in production: **showSql**, to enable the logging of all SQL
commands in console.

Providers can be easily enabled and disabled by changing the **enabled** key.

### Cache configuration
RH-SSO relies on the **Infinispan** subsystem to manage caches, which are of two
kinds:
- **local** caches, whose only purpose is to sit in front of database transactions
  and increase the performances for data reads/writes. `realms`, `users`,
  `authorization`, `keys` caches belong to this kind.
- **distributed** caches, whose data is distributed across a certain number
  of nodes in the cluster. The number of nodes owning the distributed data can
  be set in the the `owners` property and, by default is **1**, which means that
  only a single node will hold the data. If the node is lost the cached data is
  lost too and users are forced to reautheticate. Caches like `sessions`,
  `authenticatedSessions`, `offlineSessions`, `clientSessions`,
  `offlineClientSessions`, `loginFailures`, `actionTokens` belong to this kind.

Along with local and distributed caches a **replicated** cache called `work` is
configured to invalidate local caches data across all the cluster nodes.

By disabling the SPI `realmCache` and/or `userCache` the related caches will be
deactivated.

### References
https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html-single/server_installation_and_configuration_guide/#clustered-domain-example

https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.2/html-single/configuration_guide/#set_up_domain_two_machines
