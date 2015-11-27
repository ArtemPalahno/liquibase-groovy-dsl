# Groovy Liquibase
A pluggable parser for [Liquibase](http://liquibase.org) that allows the
creation of changelogs in a Groovy DSL, rather than hurtful XML. If this DSL
isn't reason enough to adopt Liquibase, then there is no hope for you.  This
project was started by Tim Berglund, and is currently maintained by Steve
Saliman.

## News
###November 30, 2015
The Liquibase Groovy DSL now supports Liquibase 3.4.2.

###August 3, 2015
The Liquibase Groovy DSL now supports Liquibase 3.3.5.

###May 16, 2015
We are proud to announce that the Liquibase Groovy DSL is now a part of the 
Liquibase organization.  I will continue maintain the code, but bringing this 
project into the Liquibase organization will help keep all things Liquibase 
together in one place.  This will help promote Liquibase adoption by making it
easier for more people to use, and it will help people stay up to date with the
latest releases.  As part of that move, the artifact name has changed from 
```net.saliman:groovy-liquibase-dsl``` to
```org.liquibase:liquibase-groovy-dsl``` to be consistent with the rest of the
Liquibase artifacts.  A special thank you to Nathan Voxland for his help and 
support in bringing the Liquibase project and the Groovy DSL together into one 
home.
 
###March 9, 2015
The Liquibase Groovy DSL now supports Liquibase 3.3.2, and is built with Groovy
2.4.1.  Version 1.0.2 fixes a bug with version 1.0.1 that prevented it from
working with Java 7 or earler.

**IMPORTANT NOTE FOR USERS UPGRADING FROM A PRE 1.0.0 RELEASE OF THE GROOVY DSL:**

Version 1.0.0 of the Groovy Liquibase DSL uses Liquibase 3, instead of Liquibase
2, and several things have been deprecated from the Groovy DSL to maintain
compatibility with Liquibase XML. A list of deprecated items can be found in the
*Usage* section.  To upgrade to version 1.0.0, we strongly recommend the
following procedure:

1. Make sure all of your Liquibase managed databases are up to date by running
   an update on them *before upgrading the Groovy DSL*.

2. Create a new, throw away database to test your Liquibase change sets.  Run
   an update on this new database with the latest version of the Groovy DSL.
   This is important because of the deprecated items in the Groovy DSL, and
   because there are some subtle differences in the ways the different Liquibase
   versions generate SQL.  For example, adding a default value to a boolean
   column in MySql using ```defaultValue: "0"``` worked fine in Liquibase 2, but
   in Liquibase 3, it generates SQL that doesn't work for MySql;
   ```defaultValueNumeric: 0``` needs to be used instead.

3. Once you are sure all of your change sets work with the latest Groovy DSL and
   Liquibase 3, clear all checksums that were calculated by Liquibase 2 by using
   the ```clearChecksums``` command in all databases.

4. Finally, run a ```changeLogSync``` on all databases to calculate new
    checksums.

## Usage
Simply include this project's jar file in your class path, and Liquibase can
parse elegant Groovy changelogs instead of ugly XML ones. The DSL syntax is 
intended to mirror the [Liquibase XML]
(http://www.liquibase.org/documentation/databasechangelog.html) syntax directly,
such that mapping elements and attributes from the Liquibase documentation to
Groovy builder syntax will result in a valid changelog. Hence this DSL is not
documented separately from the Liquibase XML format.  We will, however let you
know about the minor differences or enhancements to the XML format, and help out
with a couple of the gaping holes in Liquibase's documentation of the XML.

##### Deprecated and Unsupported Items
* Liquibase has a ```whereParam``` element for changes like the ```update``` 
  change.  It isn't documented in the Liquibase documentation, and I don't see 
  any benefits of using it over the simpler ```where``` element, so it has been
  left out of the Groovy DSL.
* In the Liquibase XML, you can set a ```sql``` attribute in a ```sqlFile```
  change, but that doesn't make a lot sense, so this has been disabled in the
  Groovy DSL.
* The documentation mentions a ```referencesUniqueColumn``` attribute of the
  ```addForeignKeyConstraint``` change, but what it doesn't tell you is that it
  is ignored.  In the code, Liquibase has marked this item as being deprecated,
  so we've deprecated it as well, and we let you know about it.
* If you were using the DSL prior to version 1.0.0, a changeSet could have an
  ```alwaysRun``` property.  This is inconsistent with Liquibase and has been
  replaced in 1.0.0 with ```runAlways```
* Prior to 1.0.0, the DSL allowed a ```path``` attribute in an ```include```.
  This is no longer allowed.  ```includeAll``` should be used instead.
* Prior to 1.0.0, the DSL allowed ```createStoredProcedure``` changes.  This has
  been replaced with ```createProcedure```.
* Prior to 1.0.0, the DSL allowed a File object to be passed as an attribute to
  ```loadData``` and ```loadUpdateData``` changes.  This is no longer supported,
  the path to the file should be used instead.
* Prior to 1.0.0, the DSL allowed constraint attributes to be set as methods
  in a constraint closure.  This is inconsistent with the rest of the DSL and
  has been removed.

##### Additions to the XML format:
* In general, boolean attributes can be specified as either strings or booleans.
  For example, ```changeSet(runAlways: 'true')``` can also be written as
  ```changeSet(runAlways: true)```.
* The Groovy DSL supports a simplified means of passing arguments to the
  ```executeCommand change```.  Instead of:

  ```groovy
execute {
  arg(value: 'somevalue')
}
```
  You can use this the simpler form:
```groovy
execute {
  arg 'somevalue'
}
```
* The ```sql``` change does not require a closure for the actual SQL.  You can
  just pass the string like this: ```sql 'select some_stuff from some_table'```
  If you want to use the ```comments``` element of a ```sql``` change, you need
  to use the closure form, and the comment must be in the closure BEFORE the
  SQL, like this:

  ```groovy
sql {
  comment('we should not have added this...')
  'delete from my_table'
}
```
* The  ```stop``` change can take a message as an argument as well as an
  attribute.  In other words, ```stop 'message'``` works as well as the more
  XMLish ```stop(message: 'message')```
* A ```customPrecondition```  can take parameters.  the XMLish way to pass them
  is with ```param(name: 'myParam', value: 'myValue')``` statements in the
  customPrecondition's closure.  In the Groovy DSL, you can also have
   ```myParam('myValue')```
* The ```validChecksum``` element of a change set is not well documented.
  Basically you can use this when changeSet's current checksum will not match
  what is stored in the database. This might happen if you, for example want to
  reformat a changeSet to add white space.  This doesn't change the
  functionality of the changeset, but it will cause Liquibase to generate new
  checksums for it.  The ```validateChecksum``` element tells Liquibase to
  consider the checksums in the ```validChecksum``` element to be valid, even
  if it doesn't match what is in the database.
* The Liquibase documentation tells you how to set a property for a
  databaseChangeLog by using the ```property``` element.  What it doesn't tell
  you is that you can also set properties by loading a property file.  To do
  this, you can have ```property(file: 'my_file.properties')``` in the closure
  for the databaseChangeLog.
* Liquibase has an ```includeAll``` element in the databaseChangeLog that
  includes all the files in the given directory.  The Groovy DSL implementation
  only includes groovy files, and it makes sure they are included in
  alphabetical order.  This is really handy for keeping changes in a different
  file for each release.  As long as the file names are named with the release
  numbers in mind, Liquibase will apply changes in the correct order.
* Remember, the Groovy DSL is basically just Groovy closures, so you can use
  groovy code to do things you could never do in XML, such as this:

```groovy
sql { """
  insert into some_table(data_column, date_inserted)
  values('some_data', '${new Date().toString()}')
"""
}
```

##### Items that were left out of the XML documentation
* The ```createIndex``` and ```dropIndex``` changes have an undocumented
  ```associatedWith``` attribute.  From an old Liquibase forum, it appears to be
   an attempt to solve the problem that occurs because some databases
   automatically create indexes on primary keys and foreign keys, and others
   don't.  The idea is that you would have a change to create the primary key or
   foreign key, and another to create the index for it.  The index change would
   use the ```associatedWith``` attribute to let Liquibase know that this index
   will already exist for some databases so that Liquibase can skip the change
   if we are in one of those databases.  The Liquibase authors do say it is
   experimental, so use at your own risk...
* The ```executeCommand``` change has an undocumented ```os``` attribute.  The
  ```os``` attribute is a string with  a list of operating systems under which
  the command should execute.  If present, the ```os.name``` system property
  will be checked against this list, and the command will only run if the
  operating system is in the list.
* The ```column``` element has some undocumented attributes that are pretty
  significant.  They include:
    - ```valueSequenceNext```, ```valueSequenceCurrent```, and
      ```defaultValueSequenceNext```, which appear to link values for a column
      to database sequences.

    - A column can be set auto-number if it the ```autoIncrement``` attribute is
      set to true, but did you know that you can also control the starting
      number and the increment interval with the ```startWith``` and
      ```incrementBy``` attributes?
* The ```constraints``` element also has some hidden gems:
    - Some databases automatically create indexes for primary keys. The
      ```primaryKeyTablespace``` can be used to control the tablespace.
    - A foreign key can be made by using the ```references``` attribute like
      this: ```references: 'monkey(id)'```, It can also be done like this:
      ```referencedTableName: 'monkey', referencedColumnNames: 'id'``` for those
    who prefer to separate out the table from the column.
    - There is also a ```checkConstraint``` attribute, that appears to be
      useful for defining a check constraint, but I could not determine the
      proper syntax for it yet.  For now, it may be best to stick to custom
      ```sql``` changes to define check constraints.
* The ```createSequence``` change has an ```cacheSize``` attribute that sets
  how many numbers of the sequence will be fetched into memory for each query
  that accesses the sequence.
* The documentation for version 3.1.1 of Liquibase mentions the new
  ```beforeColumn```, ```afterColumn```, and ```position``` attributes that you
  can put on a ```column``` statement to control where a new column is placed in
  an existing table.  What the documentation leaves out is that these attributes
  don't work :-)
* Version 3.4.0 of Liquibase introduced two new attributes to the 
  ```includeAll``` element of a databaseChangeLog, both of which are
  undocumented.  The first one is the ```errorIfMissingOrEmpty``` attribute.
  It defaults to ```true```, but if it is set to ```false```, Liquibase will
  ignore errors caused by invalid or empty directories and move on.  The second
  one is the ```resourceFilter``` attribute.  A resourceFilter is the name of a
  class that implements ```liquibase.changelog.IncludeAllFilter``` interface, 
  which allows developers to implement sophisticated logic to decide what files
  from a directory should be included (in addition to the *.groovy filter that
  the Groovy DSL imposes). 
* Liquibase 3.4.0 added the undocumented ```forIndexCatalogName```,
  ```forIndexSchemaName```, and ```forIndexName``` attributes to the 
  ```addPrimaryKey``` and ```addUniqueConstraint``` changes.  These attributes
  allow you to specify the index that will be used to implement the primary key
   and unique constraint, respectively.
* Liquibase 3.4.0 added the undocumented ```cacheSize``` and ```willCycle``` 
  attributes to the ```alterSequence```  change. ```cacheSize``` sets how many 
  numbers of the sequence will be fetched into memory for each query that 
  accesses the sequence.  ```willCycle``` determines if the sequence should 
  start over when it reaches its maximum value.

## License
This code is released under the Apache Public License 2.0, just like Liquibase 2.0.

## TODOs

 * Support for the customChange. Using groovy code, liquibase changes and database SQL in a changeSet.
 * Support for extensions. modifyColumn is probably a good place to start.
