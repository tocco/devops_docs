Entity class generation
=======================

Introduction
------------
All entities in nice2 are currently defined by \*.xml files, which are then parsed
into an :java:ref:`EntityModel<ch.tocco.nice2.persist.model.EntityModel>` instance.

There are two different :java:extdoc:`EntityMode<org.hibernate.EntityMode>` in hibernate:

- POJO (a different class per entity)
- MAP (entities based on maps)

As the two modes cannot be mixed and we would like to be able to use typed entities (instead of just the
:java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>` interface) in the future we need to dynamically generate classes for the entity models.

The `javassist <http://jboss-javassist.github.io/javassist/>`_ library is used to generate the classes. The same
library is used by  Hibernate 5.2.x itself to generate proxy classes. `Bytebuddy <http://bytebuddy.net/>`_ would
be a more modern alternative, but the newest version is much `slower <https://stackoverflow.com/questions/45456076/bytebuddy-performance-in-hibernate>`_ than javassist.

Class generation
----------------
The classes are generated during startup by the :java:ref:`JavassistEntityPojoFactory<ch.tocco.nice2.persist.hibernate.pojo.ch.tocco.nice2.persist.hibernate.pojo.JavassistEntityPojoFactory>`.
At first an 'empty' class is generated for each entity model, so that they can already be referenced by other classes.

All generated entity classes inherit from the same base class (:java:ref:`AbstractPojoEntity<ch.tocco.nice2.persist.hibernate.pojo.AbstractPojoEntity>`),
which implements all of the logic required by the :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>`
interface.

The generated classes contain the following:

* a private field of the corresponding type for each field and relation
* a getter and setter for each field
* JPA annotations, which define the hibernate data model, on the getter methods

If the annotations are placed on the fields (instead of the getters), hibernate reads and writes from the fields
directly, without using the setters and getters.

This has the advantage that we can use the entity interceptors (e.g. security) in the getters and setters, which
is necessary if we want to be able to use the created classes directly (instead of using the ``Entity`` interface)
in the future. It is also required for the 'script listener' functionality, which also uses the getters and setters directly.

Transient entities
^^^^^^^^^^^^^^^^^^

There are so called 'session-only' entities in nice2 which are not mapped to the database (used only for data binding and the like).
A different base class (:java:ref:`AbstractSessionOnlyEntity<ch.tocco.nice2.persist.hibernate.pojo.AbstractSessionOnlyEntity>`)
is used for those entities and no JPA annotations are added.
They are basically normal java beans that implement the :java:ref:`Entity<ch.tocco.nice2.persist.entity.Entity>` interface.
If such an entity is used in an association with a normal entity, no JPA annotations may be used on both sides, only
a normal field (or collection) is created.

Entities
^^^^^^^^

Apart from the usual :java:extdoc:`Entity<javax.persistence.Entity>` annotation, the database table name is
explicitly defined with the :java:extdoc:`Table<javax.persistence.Table>` annotation (we need to use the same
naming strategy to be compatible with existing databases).
A custom :java:extdoc:`EntityPersister<org.hibernate.persister.entity.EntityPersister>` is defined as well (see
:ref:`persister` for details).

Fields
^^^^^^

All fields are annotated with the :java:extdoc:`Column<javax.persistence.Column>` annotation to define the
column name of this field (we need to use the same naming strategy to be compatible with existing databases).

**Primary Key**

The primary key must be annotated with :java:extdoc:`Id<javax.persistence.Id>`. If the key value is generated
by the database the annotation :java:extdoc:`GeneratedValue<javax.persistence.GeneratedValue>` is required as well.
For autoincrement columns, the correct strategy is ``IDENTITY``.

**Version**

Fields of type version are annotated with :java:extdoc:`Version<javax.persistence.Version>`, which enables optimistic
locking for this entity.

**Text fields**

The ``text`` datatype is a :java:extdoc:`String<java.lang.String>` that should be saved into a column with datatype
``text``. To achieve this we add the :java:extdoc:`Lob<javax.persistence.Lob>` annotation to the property.

.. note::
    In Hibernate 5.2.10 a String property annotated with :java:extdoc:`Lob<javax.persistence.Lob>` was automatically
    mapped to a ``text`` column in PostgreSQL.

    However the behaviour changed in version 5.2.11 (see the `migration guide <https://github.com/hibernate/hibernate-orm/wiki/Migration-Guide---5.2>`_).
    To be compatible with existing databases, we need the behaviour of 5.2.10. In order to accomplish this, a custom
    :java:extdoc:`ClobTypeDescriptor<org.hibernate.type.descriptor.sql.ClobTypeDescriptor>` is registered in the :java:ref:`ToccoPostgreSQLDialect<ch.tocco.nice2.persist.hibernate.dialect.ToccoPostgreSQLDialect>`
    which restores the behaviour of 5.2.10.

**Counter fields**

The ``counter`` datatype is a numeric type whose value is automatically generated. The value is incremented for every new entity instance.
The counter values are managed in the ``nice_counter`` table.

Counter fields are annotated with :java:ref:`Counter<ch.tocco.nice2.persist.hibernate.pojo.generator.Counter>`, which configures
the :java:ref:`CounterGeneration<ch.tocco.nice2.persist.hibernate.pojo.generator.CounterGeneration>` value generator. This
generator is only applied whenever a new entity is inserted (not when an entity is updated).

If the value of a counter field is manually set in the transaction it will not be overwritten.

At first, the counter entity (for the relevant entity type, field and business unit) is fetched from the database
using the ``PESSIMISTIC_WRITE`` lock mode.
The counter value is then updated using a `stateless session <http://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#_statelesssession>`_ to make sure that
database is updated immediately. This is necessary if the same counter is used multiple times in the same transaction.
It is important that the connection of the current session is also used in the stateless session to make sure that they use
the same database transaction.

.. note::
    It would probably make sense to use a database ``sequence`` for this purpose in the future.

**Custom user types**

Per default Hibernate can map all primitive types (and its wrapper classes) as well as references to other entities.
For all other classes that need to be mapped to the database an :java:extdoc:`UserType<org.hibernate.usertype.UserType>`
must be implemented (for immutable types the base class :java:ref:`ImmutableUserType<ch.tocco.nice2.persist.hibernate.usertype.ImmutableUserType>`
can be used). The user type contains the logic how a specific object should be read from the :java:extdoc:`ResultSet<java.sql.ResultSet>`
and written to the :java:extdoc:`PreparedStatement<java.sql.PreparedStatement>`.

For example the :java:ref:`Login<ch.tocco.nice2.types.Login>` class is mapped with a custom user type (:java:ref:`LoginUserType<ch.tocco.nice2.persist.hibernate.usertype.LoginUserType>`).
A new user type can be registered with a bootstrap contribution (see :ref:`bootstrap`):

.. code-block:: java

    classLoaderService.addContribution(TypeContributor.class, ((typeContributions, serviceRegistry) -> {
        typeContributions.contributeType(new BinaryUserType(binaryAccessProvider, binaryHashingService), Binary.class.getName());
        typeContributions.contributeType(new LoginUserType(), Login.class.getName());
        typeContributions.contributeType(new UuidToStringUserType(), UUID.class.getName());
    }));

**Other fields**

The ``nullable``, ``unique`` and if applicable ``precision`` and ``scale`` properties are set on the :java:extdoc:`Column<javax.persistence.Column>` annotation.
These properties are only used for schema generation in test cases (databases are setup by liquibase), not for
validation!
The type ``decimal`` (without precision and scale) is handled specially, because Hibernate would use a default
precision and scale, but in this case we want to use the column type ``decimal`` without any precision or scale.

.. _generated-fields-annotations:

Generated fields
^^^^^^^^^^^^^^^^

It is possible to define custom data types whose values are automatically set when an entity is saved or updated.
These fields are annotated either with the :java:ref:`AlwaysGeneratedValue<ch.tocco.nice2.persist.hibernate.pojo.generator.AlwaysGeneratedValue>`
for fields which should be updated on create and update or the :java:ref:`InsertGeneratedValue<ch.tocco.nice2.persist.hibernate.pojo.generator.InsertGeneratedValue>`
for fields which should only be updated when the entity is created.

See :ref:`generated-values`.

Associations
^^^^^^^^^^^^

Associations (relations) are annotated with one of the following JPA annotations (depending on the type):

- :java:extdoc:`OneToMany<javax.persistence.OneToMany>`
- :java:extdoc:`ManyToOne<javax.persistence.ManyToOne>`
- :java:extdoc:`ManyToMany<javax.persistence.ManyToMany>`

So far all associations are bi-directional (even if this does not always make sense).
In a ManyToOne/OneToMany association, the ManyToOne side is always the owning side. In a ManyToMany association,
the owning side needs to be explicitly specified (with the :java:extdoc:`JoinTable<javax.persistence.JoinTable>`
annotation).
The owning side is responsible for persisting the relationship - if a change is only done on the inverse side of
an association, it will not be persisted! For example in a ManyToMany association, entities must always be added
and removed from the owning side, otherwise the mapping table won't be updated.

For collections a :java:extdoc:`LinkedHashSet<java.util.LinkedHashSet>` is used, because we want :java:extdoc:`LinkedHashSet<java.util.Set>` semantics
(no duplicates), but need to iterate over the elements in the same order as they were inserted (to support sorting by the database).

All associations (including ManyToOne) are configured to be loaded lazily by specifying the :java:extdoc:`FetchType<javax.persistence.FetchType>`
on the annotation. Per default only to many associations are loaded lazily, that's why we need to explicitly configure
it for to one associations.

When a collection has been initialized it cannot be reloaded from the database (unless the entire object is reloaded).
However when a  :java:ref:`Relation<ch.tocco.nice2.persist.entity.Relation>` is resolved, the data should always be
loaded from the database (because this was the behaviour of the old persistence implementation).
To support this behaviour we use a custom collection type (:java:extdoc:`CollectionType<org.hibernate.annotations.CollectionType>`).

See :ref:`collections` chapter for more details.

A custom :java:extdoc:`CollectionPersister<org.hibernate.persister.collection.CollectionPersister>` is also configured (see
:ref:`persister` for details).