---
layout: default
title: CAS - SAML2 Dynamic Metadata Management
---

# SAML2 Dynamic Metadata Management

Management of service provider metadata in a dynamic on-the-fly fashion may be accomplished via strategies outlined here.

## Metadata Query Protocol

CAS also supports the [Dynamic Metadata Query Protocol](https://spaces.internet2.edu/display/InCFederation/Metadata+Query+Protocol)
which is a REST-like API for requesting and receiving arbitrary metadata. In order to configure a CAS SAML service to retrieve its metadata
from a Metadata query server, the metadata location must be configured to point to the query server instance. Here is an example:

```json
{
  "@class" : "org.apereo.cas.support.saml.services.SamlRegisteredService",
  "serviceId" : "the-entity-id-of-the-sp",
  "name" : "SAMLService",
  "id" : 10000003,
  "evaluationOrder" : 10,
  "metadataLocation" : "http://mdq.server.org/entities/{0}"
}
```

...where `{0}` serves as an entityID placeholder for which metadata is to be queried.

## MongoDb

Metadata documents may also be stored in and fetched from a MongoDb instance.  This may specially be used to avoid copying metadata files across CAS nodes in a cluster, particularly where one needs to deal with more than a few bilateral SAML integrations. Metadata documents are stored in and fetched from a single pre-defined collection that is taught to CAS via settings.  The outline of the document is as follows:

| Field                     | Description
|--------------|---------------------------------------------------
| `id`                          | The identifier of the record.
| `name`             | Indexed field which describes and names the metadata briefly.
| `value`              | The XML document representing the metadata for the service provider.
| `signature`              | The contents of the signing key to validate metadata, if any.

Support is enabled by including the following module in the overlay:

```xml
<dependency>
  <groupId>org.apereo.cas</groupId>
  <artifactId>cas-server-support-saml-idp-metadata-mongo</artifactId>
  <version>${cas.version}</version>
</dependency>
```

SAML service definitions must then be designed as follows to allow CAS to fetch metadata documents from MongoDb instances:

```json
{
  "@class" : "org.apereo.cas.support.saml.services.SamlRegisteredService",
  "serviceId" : "the-entity-id-of-the-sp",
  "name" : "SAMLService",
  "id" : 10000003,
  "description" : "A MongoDb-based metadata resolver",
  "metadataLocation" : "mongodb://"
}
```

<div class="alert alert-info"><strong>Metadata Location</strong><p>
The metadata location in the registration record above simply needs to be specified as <code>mongodb://</code> to signal to CAS that SAML metadata for registered service provider must be fetched from MongoDb data sources defined in CAS configuration. 
</p></div>

## JPA

Metadata documents may also be stored in and fetched from a relational database instance.  This may specially be used to avoid copying metadata files across CAS nodes in a cluster, particularly where one needs to deal with more than a few bilateral SAML integrations. Metadata documents are stored in and fetched from a single pre-defined table  (i.e. `SamlMetadataDocument`) whose connection information is taught to CAS via settings and is automatically generated.  The outline of the table is as follows:

| Field                     | Description
|--------------|---------------------------------------------------
| `id`                          | The identifier of the record.
| `name`             | Indexed field which describes and names the metadata briefly.
| `value`              | The XML document representing the metadata for the service provider.
| `signature`              | The contents of the signing key to validate metadata, if any.

Support is enabled by including the following module in the overlay:

```xml
<dependency>
  <groupId>org.apereo.cas</groupId>
  <artifactId>cas-server-support-saml-idp-metadata-jpa</artifactId>
  <version>${cas.version}</version>
</dependency>
```

SAML service definitions must then be designed as follows to allow CAS to fetch metadata documents from database  instances:

```json
{
  "@class" : "org.apereo.cas.support.saml.services.SamlRegisteredService",
  "serviceId" : "the-entity-id-of-the-sp",
  "name" : "SAMLService",
  "id" : 10000003,
  "description" : "A relational-db-based metadata resolver",
  "metadataLocation" : "jdbc://"
}
```

<div class="alert alert-info"><strong>Metadata Location</strong><p>
The metadata location in the registration record above simply needs to be specified as <code>jdbc://</code> to signal to CAS that SAML metadata for registered service provider must be fetched from JDBC data sources defined in CAS configuration. 
</p></div>

## Groovy

A metadata location for a SAML service definition may  point to an external Groovy script, allowing the script to programmatically determine and build the metadata resolution machinery to be added to the collection of the existing resolvers. 

```json
{
  "@class" : "org.apereo.cas.support.saml.services.SamlRegisteredService",
  "serviceId" : "the-entity-id-of-the-sp",
  "name" : "SAMLService",
  "id" : 10000003,
  "description" : "A Groovy-based metadata resolver",
  "metadataLocation" : "file:/etc/cas/config/groovy-metadata.groovy"
}
```

The outline of the script may be as follows:

```groovy
import java.util.*
import org.apereo.cas.support.saml.*;
import org.apereo.cas.support.saml.services.*;
import org.opensaml.saml.metadata.resolver.*;

def Collection<MetadataResolver> run(final Object... args) {
    def registeredService = args[0]
    def samlConfigBean = args[1]
    def samlProperties = args[2]
    def logger = args[3]

    /*
     Stuff happens where you build the relevant metadata resolver instance(s).
     When done, simply wrap the results into a collection and return.
     A null or empty collection will be ignored by CAS.
  */
  def metadataResolver = ...
   return CollectionUtils.wrap(metadataResolver);
}
```

The parameters passed are as follows:

| Parameter             | Description
|-----------------------|-----------------------------------------------------------------------
| `registeredService`             | The object representing the corresponding service definition in the registry.
| `samlConfigBean`            | The object representing the OpenSAML configuration class holding various builder and marshaller factory instances.
| `samlProperties`         | The object responsible for capturing the CAS SAML IdP properties defined in the configuration.
| `logger`              | The object responsible for issuing log messages such as `logger.info(...)`.
