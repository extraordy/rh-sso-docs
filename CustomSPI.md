# Implementing Custom SPI

Red Hat Single Sign-On can be extended implementing a custom Service Provider
Interface (SPI).
This task requires a basic knowledge of Java and JEE specifications.

The Keycloak **Javadoc** is available at the following [link](https://www.keycloak.org/docs-api/5.0/javadocs/index.html).

Basically, to define a custom SPI must implement the `Provider` and a `ProviderFactory` interfaces, defined in the `org.keycloak.provider` package.  
The `Provider` interface only defines a `close()` method and is extended by
a series of subinterfaces.

### Hello World SPI
The following example will illustrate the details of a simple REST based SPI
that implements the `RealmResourceProvider` interface, which extends `Provider`.
This example is taken from the **Keycloak** repository in the **examples**
subfolder:
https://github.com/keycloak/keycloak/tree/master/examples/providers/rest.


#### Defining the ProviderFactory
First, we start from the `HelloResourceProviderFactory` class, which implements
the `RealmResourceProviderFactory` interface. This interface inherits from the
`ProviderFactory<T extends Provider>` interface the following methods used to
create the custom provider:
- `close()`
- `create()`
- `getId()`
- `init()`
- `order()`
- `postInit()`

The `HelloResourceProviderFactory` class is defined in the code snippet below.

```
package org.keycloak.examples.rest;

import org.keycloak.Config.Scope;
import org.keycloak.models.KeycloakSession;
import org.keycloak.models.KeycloakSessionFactory;
import org.keycloak.services.resource.RealmResourceProvider;
import org.keycloak.services.resource.RealmResourceProviderFactory;

public class HelloResourceProviderFactory implements RealmResourceProviderFactory {

    public static final String ID = "hello";

    @Override
    public String getId() {
        return ID;
    }

    @Override
    public RealmResourceProvider create(KeycloakSession session) {
        return new HelloResourceProvider(session);
    }

    @Override
    public void init(Scope config) {
    }

    @Override
    public void postInit(KeycloakSessionFactory factory) {
    }

    @Override
    public void close() {
    }

}
```

Notice the overrides of the above mentioned methods from the `ProviderFactory`
interface.
The `create()` method takes as an argument the session info (of type `KeycloakSession` and returns an `HelloResourceProvider` instance
using its constructor.
The `postInit()` method takes as an argument a factory of type
`KeycloakSessionFactory` but returns nothing in this example.

#### Defining the Provider
Once the provider factory is defined, we move to the provider implementation.
The class `HelloResourceProvider` extends the `RealmResourceProvider` interface:
this interface has the purpose of creating **JAX-RS** subresources instances for
paths relative to the realms's REST API. This basically means that we can use
this interface to extend the APIs by adding more restful class methods annotated with GET/POST/PUT/DELETE HTTP methods.   


```
package org.keycloak.examples.rest;

import org.keycloak.models.KeycloakSession;
import javax.ws.rs.Produces;

public class HelloResourceProvider implements RealmResourceProvider {

    private KeycloakSession session;

    public HelloResourceProvider(KeycloakSession session) {
        this.session = session;
    }

    @Override
    public Object getResource() {
        return this;
    }

    @GET
    @Produces("text/plain; charset=utf-8")
    public String get() {
        String name = session.getContext().getRealm().getDisplayName();
        if (name == null) {
            name = session.getContext().getRealm().getName();
        }
        return "Hello " + name;
    }

    @Override
    public void close() {
    }

}
```

Notice the constructor which takes the session as the only argument, needed to
maintain the session context.

The most important method here is `get()`, which returns a `String` value
containint the concatenation `"Hello" + name` where the name is extracted from
the session using the the `getContext()`, `getRealm()` and `getDisplayName()`
(or `getName()`) methods.  

The string output is rendered as **text/plain** in **utf-8** charset, as
stated by the `@Produces` annotation, which specify the type of output rendered
by this REST service.   

The `@GET` annotations specify which HTTP method is expected
to invoke this funcion.

#### Defining the META-INF service
The implemented ProviderFactory must be defined as a service in the META-INF
folder to have the application server point correctly to the resource used to
produce the provider.
In the project folder the new file will be created in the subdirectory
`src/main/resources/META-INF/services` and named
`org.keycloak.services.resource.RealmResourceProviderFactory`.

In this small example the file will contain only the factory name:
```
#
# Copyright 2016 Red Hat, Inc. and/or its affiliates
# and other contributors as indicated by the @author tags.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
org.keycloak.examples.rest.HelloResourceProviderFactory
```

### Building
To build the example provider checkout to a specific branch of the repository
and build using the **maven** tool.

```
$git clone https://github.com/keycloak/keycloak
$cd keycloak/examples/providers/rest
$ git checkout 5.0.0
$ mvn clean package
```

The build will produce in the `target` folder the `hello-rest-example.jar` file.

### Deploying the provider
This file can be deployed in two way in Red Hat Single Sign-on:
- Using the builtin JBoss EAP deployment scanner: the artifact should be copied
  in the `.../standalone/deployments` directory. The provider jar will work as
  any other deployed component in JBoss EAP. This method can be applied to
  standalone instances and not to domain installations.

- Installing the artifact as a module. This method can be applied to both
  standalone and domain installations and is more elegant. The native JBoss EAP
  CLI can be used to accomplish this task:

  ```
  $KEYCLOAK_HOME/bin/jboss-cli.sh \
  --command="module add --name=org.keycloak.examples.hello-rest-example
  --resources=target/hello-rest-example.jar
  --dependencies=org.keycloak.keycloak-core,org.keycloak.keycloak-server-spi,
    org.keycloak.keycloak-server-spi-private,javax.ws.rs.api"
  ```

  **IMPORTANT**: The above mentioned command must be written as a single line with
  no breaks. The command must be executed in every standalone node and every host
  when in domain mode.

  This command defines a new module called `org.keycloak.examples.hell-rest-example`,
  the artifact jar, which was built in the step before, and all its dependencies.
  The module will be installed in the
  `modules/org/keycloak/examples/hello-rest-example/main` directory in the RH-SSO
  installation.
  The command copies the `hello-rest-example.jar` file and creates the `module.xml`
  file, whose content reflects the parameters passed before:

  ```
  <?xml version='1.0' encoding='UTF-8'?>

  <module xmlns="urn:jboss:module:1.1" name="org.keycloak.examples.hello-rest-example">

    <resources>
        <resource-root path="hello-rest-example.jar"/>
    </resources>

    <dependencies>
        <module name="org.keycloak.keycloak-core"/>
        <module name="org.keycloak.keycloak-server-spi"/>:
        <module name="org.keycloak.keycloak-server-spi-private"/>
        <module name="javax.ws.rs.api"/>
    </dependencies>
  ```

### Provider Registration
The last step, after the artifact deployment, is the registration of the newly
created provider.
The **keycloak-server** subsystem in the JBoss EAP config files (both
*standalone.xml* and *domain.xml*) contains a **providers** section where
new providers can be defined and/or enabled.
If the provider has been deployed as a module, it must be registered with the
following syntax:

```
<providers>
   ...
   <provider>module:org.keycloak.examples.hello-rest-example</provider>
</providers>
```

**IMPORTANT**: After registration the RH-SSO instance(s) must be restarted.

### Testing the API
After restarting the RH-SSO the new implemented REST API can be tested using
a browser or the curl command:

```
$ curl -X GET http://localhost:8080/auth/realms/master/hello
Hello rh-sso
```

This demonstrated that the API has been registered correctly.

### Conclusions
The example described in this guide is very minimal but is also a good foundation 
to extend further considerations. More details on the development of 
custom providers in Red Hat Single Sign-On in the [SPI Section](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html/server_developer_guide/providers) in the [Server Developer Guide](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html/server_developer_guide/index).
