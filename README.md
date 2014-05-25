GraphAware Neo4j TimeTree
=========================

[![Build Status](https://travis-ci.org/graphaware/neo4j-timetree.png)](https://travis-ci.org/graphaware/neo4j-timetree) | <a href="http://graphaware.com/downloads/" target="_blank">Downloads</a> | <a href="http://graphaware.com/site/timetree/latest/apidocs/" target="_blank">Javadoc</a> | Latest Release: 2.0.3.4.3

GraphAware TimeTree is a simple library representing time in Neo4j as a tree of time instants. The tree is built on-demand,
supports resolutions of one year down to one millisecond and has time zone support.

Getting the Software
--------------------

### Server Mode

When using Neo4j in the <a href="http://docs.neo4j.org/chunked/stable/server-installation.html" target="_blank">standalone server</a> mode,
you will need the <a href="https://github.com/graphaware/neo4j-framework" target="_blank">GraphAware Neo4j Framework</a> and GraphAware Neo4j TimeTree .jar files (both of which you can <a href="http://graphaware.com/downloads/" target="_blank">download here</a>) dropped
into the `plugins` directory of your Neo4j installation. After Neo4j restart, you will be able to use the REST APIs of the TimeTree.

### Embedded Mode / Java Development

Java developers that use Neo4j in <a href="http://docs.neo4j.org/chunked/stable/tutorials-java-embedded.html" target="_blank">embedded mode</a>
and those developing Neo4j <a href="http://docs.neo4j.org/chunked/stable/server-plugins.html" target="_blank">server plugins</a>,
<a href="http://docs.neo4j.org/chunked/stable/server-unmanaged-extensions.html" target="_blank">unmanaged extensions</a>,
GraphAware Runtime Modules, or Spring MVC Controllers can include use the TimeTree as a dependency for their Java project.

#### Releases

Releases are synced to <a href="http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22timetree%22" target="_blank">Maven Central repository</a>. When using Maven for dependency management, include the following dependency in your pom.xml.

    <dependencies>
        ...
        <dependency>
            <groupId>com.graphaware.neo4j</groupId>
            <artifactId>timetree</artifactId>
            <version>2.0.3.4.3</version>
        </dependency>
        ...
    </dependencies>

#### Snapshots

To use the latest development version, just clone this repository, run `mvn clean install` and change the version in the
dependency above to 2.0.3.4.4-SNAPSHOT.

#### Note on Versioning Scheme

The version number has two parts. The first four numbers indicate compatibility with Neo4j GraphAware Framework.
 The last number is the version of the TimeTree library. For example, version 2.0.3.4.3 is version 3 of the TimeTree
 compatible with GraphAware Neo4j Framework 2.0.3.4.

Using GraphAware TimeTree
-------------------------

TimeTree allows you to represent events as nodes and link them to nodes representing instants of time in order to capture
the time of the event's occurrence. For instance, if you wanted to express the fact that an email was sent on a specific
day, you would create a node labelled `Email` and link it to a node labelled `Day` using a `SENT_ON` relationship.

![email linked to day](https://github.com/graphaware/neo4j-timetree/raw/master/docs/image1.pdf)

In order to be able to ask interesting queries, such as "show me all emails sent in a specific month", people <a href="http://neo4j.com/blog/modeling-a-multilevel-index-in-neoj4/" target="_blank">often built</a>
a time-tree in Neo4j with a root, years on the first level, months on the second level, etc. Something like this:

![time tree](https://github.com/graphaware/neo4j-timetree/raw/master/docs/image2.pdf)

One way of building such tree is of course pre-generating it, for instance using a Cypher query. Michael Huger has written
about that here.

The approach taken by GraphAware TimeTree is to build the tree on-demand, as nodes representing time instants are requested.
For example, you can ask the library "give me a node representing 24th May 2014". You'll get the node (Java) or its ID (REST) and can start linking to it.
In the background, a node representing May (labelled `Month`) and a node representing 2014 (labelled `Year`) will be created,
if they do not exist. Links between nodes on the same level as well as between levels are automatically maintained.

There are 4 types of relationships (hopefully self-explanatory):
* CHILD
* NEXT
* FIRST
* LAST

The root of the tree is labelled `TimeTreeRoot'.

The graph above, if generated by GraphAware TimeTree, would thus look like this:

![GraphAware TimeTree generated time tree](https://github.com/graphaware/neo4j-timetree/raw/master/docs/image3.pdf)

You can select the "resolution" of the time instant you will get from TimeTree. For instance, you only know that a certain
event happened in April, nothing more. In that case, you can request the node representing April. Equally, you can request
nodes representing concrete millisecond instants, if you so desire.

Finally, you can provide a time-zone to the TimeTree APIs in order to create correctly labelled nodes for specific time instants.

### REST API

When deployed in server mode, there are two URLs that you can issue POST requests to:
* `http://your-server-address:7474/graphaware/timetree/{time}` to get a node representing a time instant, where time must be replaced by a `long` number representing the number of milliseconds since 1/1/1970. The default resolution is Day and the default time zone is UTC
* `http://your-server-address:7474/graphaware/timetree/now` to get a node representing now. Defaults are the same as above.

You two query parameters:
* `resolution`, which can take on the following values:
    * `Year`
    * `Month`
    * `Day`
    * `Hour`
    * `Minute`
    * `Second`
    * `Millisecond`
* `timezone`, which can be a String representation of any `java.util.TimeZone`

For instance, issuing the following request, asking for the hour node representing 5th April 2014 1pm (UTC time) in the
GMT+1 time zone

    POST http://your-server-address:7474/graphaware/timetree/1396706182123?resolution=Hour&timezone=GMT%2B1

on an empty database will result in the following graph being generated. The response body will contain the Neo4j node ID
of the node representing the hour. You can then use it in order to link to it.

![GraphAware TimeTree generated time tree](https://github.com/graphaware/neo4j-timetree/raw/master/docs/image4.pdf)

### Java API

Java API has the same functionality as the rest API. Please refer to <a href="http://graphaware.com/site/timetree/latest/apidocs/" target="_blank">its Javadoc</a> (look at the `TimeTree` interface).

License
-------

Copyright (c) 2014 GraphAware

GraphAware is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License
as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.
This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied
warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.
You should have received a copy of the GNU General Public License along with this program.
If not, see <http://www.gnu.org/licenses/>.