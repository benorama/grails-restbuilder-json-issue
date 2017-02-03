# Grails RestBuilder/JSON issue

## Config

Grails **3.2.5**

## Description

This app uses Spring resources to create an instance of `RestBuilder`.

It looks like the default `RestBuilder` constructor creates some issue with JSON converters and marshalling.
Once `RestBuilder` default constructor is called, JSON marshalling is broken.

Here is a related issue #825 solved since Grails grails-datastore-rest-client **6.0.5+**:
https://github.com/grails/grails-data-mapping/issues/825

Source of the issue is the following code:

```groovy
def currentConfiguration = ConvertersConfigurationHolder.getConverterConfiguration(JSON)
if(currentConfiguration instanceof DefaultConverterConfiguration) {
    // init manually
    DefaultConverterConfiguration defaultConfig = ((DefaultConverterConfiguration)currentConfiguration)
    defaultConfig.registerObjectMarshaller(new MapMarshaller())
    defaultConfig.registerObjectMarshaller(new ArrayMarshaller());
    defaultConfig.registerObjectMarshaller(new ByteArrayMarshaller());
    defaultConfig.registerObjectMarshaller(new CollectionMarshaller());
    defaultConfig.registerObjectMarshaller(new GroovyBeanMarshaller());
}
```

To reproduce the issue:

* `grails run-app`
* go to `/console`

It will generate a `StringIndexOutOfBoundsException` exception when trying to execute `App.start(<%= json as JSON %>);` in console index.gsp.

## Workaround

* Create a new `RestBuilder` class in your own package and remove the code above from the constructor.
* Use this new class in our Spring resources instead. 

Why is this code required? `JSON` already handles default class marshalling, isn't it?