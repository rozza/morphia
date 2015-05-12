+++
date = "2015-03-17T15:36:56Z"
title = "Annotations"
[menu.main]
  parent = "Reference Guides"
  weight = 85
  pre = "<i class='fa fa-file-text-o'></i>"
+++

# Annotations

Below is a list of all the annotations and a brief descriptions of how to use them.

## Entity
Marks entities to be stored directly in a collection. This annotations is optional in most cases (though this is likely to change in 
future versions). There is no harm in including it to be more verbose, and make clear the intention for the class.  The definition of 
this annotation looks like this:

```java
public @interface Entity {
  String value() default Mapper.IGNORED_FIELDNAME;
  CappedAt cap() default @CappedAt(0);
  boolean noClassnameStored() default false;
  boolean queryNonPrimary() default false;
  String concern() default "";
}
```
| Parameter           | Usage           |
| ------------------- | --------------  |
| value()             | Defines the collection to use.  Defaults to using the classname |
| cap()               | Marks this collection as capped and sets the size to use.  See the [`@Capped`]({{< ref "#capped" >}}) below|
| noClassnameStored() | Tells morphia to not store the classname in the document.  The default is to store the classname. |
| queryNonPrimary()   | Indicates that queries against this collection can use secondaries.  The default is primary only reads. |
| concern()           | The WriteConcern to use when writing to this collection. |


## Indexes

In addition to being able to declare an index on a single field you can also declare the indexes at the class level. This allows you to 
 create more than just a single field index; it allows you to create compound indexes with multiple fields.

```java
public @interface Indexes {
    Index[] value();
}
```

### Index
 
```java
public @interface Index {
    String value() default "";
    String name() default "";
    boolean unique() default false;
    boolean dropDups() default false;
    boolean background() default false;
    boolean sparse() default false;
    boolean disableValidation() default false;
    int expireAfterSeconds() default -1;

    Field[] fields() default {};
    IndexOptions options() default @IndexOptions();
}
```

There are two pieces to this annotation that are mutually exclusive.  The first group of parameters are considered legacy.  They are safe
 to use since but are unlikely to survive past the 1.x series.  These options and more have been conglomerated in the `@Field` annotation.
 
| Parameter           | Usage           |
| ------------------- | --------------  |
| value() | List of fields (prepended with "-" for desc; defaults to asc). |  
| name() | The name of the index |
| unique() | Requires values in the index to be unique |
| dropDups() | Drop any duplicate values during the creation of a unique index. |
| background()| Create this index in the background.  There are some [considerations](http://docs.mongodb.org/manual/tutorial/build-indexes-in-the-background/#considerations) to keep in mind.
| sparse() | Create a [sparse](http://docs.mongodb.org/manual/core/index-sparse/) index |
| disableValidation() | By default, morphia will validate field names being index.  This disables those checks. |
| expireAfterSeconds() | Creates a [TTL Index](http://docs.mongodb.org/manual/core/index-ttl/) on a date field. |
| fields() | This is the new way to define which fields to index. [Details]({{< ref "#Field" >}}) can be found below. |
| options() | This is the new way to define index options. [Details]({{< ref "#IndexOptions" >}}) can be found below.  |

#### Field
```java
 public @interface Field {
     String value();
     IndexType type() default IndexType.ASC;
     int weight() default -1;
 }
```

| Parameter           | Usage           |
| ------------------- | --------------  |
| value() | The field to include in the index |
| type() | The type of index to create.  This includes the following values:   `ASC`, `DESC`, `GEO2D`, `GEO2DSPHERE`, `TEXT` |
| weight() | When defining a text index, this is the weight to apply |

#### IndexOptions
```java
public @interface IndexOptions {
    String name() default "";
    boolean unique() default false;
    boolean dropDups() default false;
    boolean background() default false;
    boolean sparse() default false;
    boolean disableValidation() default false;
    int expireAfterSeconds() default -1;
    String language() default "";
    String languageOverride() default "";
}
```
| Parameter           | Usage           |
| ------------------- | --------------  |
| name() | The name of the index |
| unique() | Requires values in the index to be unique |
| dropDups() | Drop any duplicate values during the creation of a unique index. |
| background() | Create this index in the background.  There are some [considerations](http://docs.mongodb.org/manual/tutorial/build-indexes-in-the-background/#considerations) to keep in mind.
| sparse() | Create a [sparse](http://docs.mongodb.org/manual/core/index-sparse/) index |
| disableValidation() | By default, morphia will validate field names being index.  This disables those checks. |
| expireAfterSeconds() | Creates a [TTL Index](http://docs.mongodb.org/manual/core/index-ttl/) on a date field. |
| language() | Default language for the index. |
| languageOverride() | The field in the document to use to override the default language. |

## Id
Marks a field in an `@Entity` to be the "_id" field in mongodb.

## Property
An optional annotation instructing morphia to persist field in to the document given to mongodb.  By default, the field name is used as 
the property name.  This can be overridden by passing a String with the new name to the annotation.

## Transient
Instructs morphia to ignore this field when converting an entity to a document.  The java keyword `transient` can also be used instead.

## Serialized
Instructs morphia to serialize this field.

```java
public @interface Serialized {
  String value() default Mapper.IGNORED_FIELDNAME;
  boolean disableCompression() default false;
}
```
| Parameter           | Usage           |
| ------------------- | --------------  |
| value() | the field name to use in the document |
| disableCompression() | By default, morphia compresses the `byte[]` after serialization.  Setting this to true disables the compression. |


## NotSaved
Instructs morphia to ignore this field when saving but will still be loaded.
 
_Good for data migration._

## AlsoLoad
When a field gets remapped to a new name, you can either update the database and migrate all the fields at once or use this annotation 
to tell morphia what older names to try if the current one fails.  It is an error to have values under both the old and new key names 
when loading a document.

_Good for data migration._

```java
public @interface AlsoLoad {
  String[] value();
}
```

| Parameter           | Usage           |
| ------------------- | --------------  |
| value() | The array of names to try when loading the field |

## Indexed
The field will be indexed. See [the datastore docs](Datastore).

## Version
Marks a field in an `@Entity` to control optimistic locking for that entity. If the versions change while modifying an entity (including 
deletes) a `ConcurrentModificationException` will be thrown. This field will be automatically managed for you -- there is no need to set
 a value and you should not do so anyway.  If the another name beside the Java field name is desired, a name can be passed to this 
 annotation to change the document's field name.

```java
@Entity
class MyClass {
   ...
   @Version Long v;
}
```

## Reference
Marks fields as stored in another collection and which are linked (by a `DBRef` field). When the Entity is loaded, the referenced Entity
 can also be loaded.  Any document referenced via an `@Reference` field must have already been saved in mongodb or have the Java object's
  `@Id` already assigned.  Otherwise, no key can be copied in to the `Key` for storage in the database.  If you're always saving the 
  referenced entity in the mapped collection (`Datastore` can be told to save in to a collection other than the mapped collection) a lot 
  of space can be saved by using the `idOnly()` parameter to just save the key value.

```java
public @interface Reference {
  String value() default Mapper.IGNORED_FIELDNAME;
  Class<?> concreteClass() default Object.class;
  boolean ignoreMissing() default false;
  boolean lazy() default false;
  boolean idOnly() default false;
}
```
| Parameter           | Usage           |
| ------------------- | --------------  |
| value() | The field name to use in the document.  Defaults to the Java field name. |
| ignoreMissing() | Ignore any missing documents |
| lazy() | Instructs morphia to defer loading of the referenced document. |
| idOnly() | Instructs morphia to only store the key of the referenced document rather than a full `DBRef` |


## Embedded
In contrast to `@Reference` where a nested Java reference ends up as a separate document in a collection, `@Embedded` tells morphia 
to embed the document created from the Java object in the document of the parent object.  This annotation can be applied to the class of 
the embedded type or on the field holding the embedded instance.

```java
public @interface Embedded {
  String value() default Mapper.IGNORED_FIELDNAME;
  Class<?> concreteClass() default Object.class;
}
```
| Parameter           | Usage           |
| ------------------- | --------------  |
| value() | The field name to use in the document.  Defaults to the Java field name. |
| concreteClass() | The concrete class to use when instantiating the embedded entity |

## Lifecycle Annotations

There are various annotations which can be used to register callbacks on certain lifecycle events. These include Pre/Post-Persist (Save), and Pre/Post-Load.

- `@PreLoad` - Called before mapping the datastore object to the entity (POJO); the DBObject is passed as an argument (you can add/remove/change values)
- `@PostLoad` - Called after mapping to the entity
- `@PrePersist` - Called before save, it can return a DBObject in place of an empty one.
- `@PostPersist` - Called after the save call to the datastore
- `@PostSave` - Called before the save call to the datastore

### Examples
[Here](https://github.com/mongodb/morphia/blob/master/morphia/src/test/java/org/mongodb/morphia/TestQuery.java#L63) is a one of the test classes.

All parameters and return values are options in your implemented methods.

#### `@PrePersist`
Here is a simple example of an entity that always saves the Date it was last updated at.
```java
class BankAccount {
  @Id String id;
  Date lastUpdated = new Date();

  @PrePersist void prePersist() {lastUpdated = new Date();}
}
```

#### `@EntityListeners`
In addition, you can separate the lifecycle event implementation in an external class, or many.
```java
@EntityListeners(BackAccountWatcher.class)
public class BankAccount {
  @Id String id;
  Date lastUpdated = new Date();
}

class BankAccountWatcher{

  @PrePersist void prePersist(BankAccount act) {act.lastUpdated = new Date();}

}
```

### No Delete Support
Because deletes are usually done with queries there is no way to support a Delete lifecycle event. If, or when, server-side triggers are
  enabled there may be some support for this, but even then it will be hard to imagine how this would logically fit.