# Implementing Custom SPI

Red Hat Single Sign-On can be extended implementing a custom Service Provider
Interface (SPI).
This task requires a basic knowledge of Java and JEE specifications.

The Keycloak **Javadoc** is available at the following [link](https://www.keycloak.org/docs-api/5.0/javadocs/index.html).

Basically, to define a custom SPI must implement the `Provider` and a `ProviderFactory` interfaces, defined in the `org.keycloak.provider` package.  
The `Provider` interface only defines a `close()` method and is extended by
a series of subinterfaces.

### SPI Hello World
The following example will illustrate the details of a simple REST based SPI
that implements the `RealmResourceProvider` interface, which extends `Provider`.
This example is took from the **Keycloak** repository in the **examples**
subfolder:
https://github.com/keycloak/keycloak/tree/master/examples/providers/rest.

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
interface. The `create()` method returns an `HelloResourceProvider` instance
using its constructor, which takes the keycloak **session** as the only argument.


Once the provider factory is defined, we move to the provider implementation.

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
