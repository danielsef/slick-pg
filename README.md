Slick-pg
========
[Slick](https://github.com/slick/slick "Slick") extensions for PostgreSQL, to support a series of pg data types and related operators/functions.

####Currently supported data types:
- ARRAY
- Range
- Hstore
- Search(tsquery+tsvector)
- Geometry

** tested on `postgreSQL 9.2` with `Slick 1.0`.

Install
-------
To use slick-pg in [sbt](http://www.scala-sbt.org/ "slick-sbt") project, add the following to your project file:
```scala
libraryDependencies += "com.github.tminglei" % "slick-pg_2.10.1" % "0.1.1"
```

Or, in [maven](http://maven.apache.org/ "maven") project, you can add `slick-pg` to your `pom.xml` like this:
```
<dependency>
    <groupId>com.github.tminglei</groupId>
    <artifactId>slick-pg_2.10.1</artifactId>
    <version>0.1.1</version>
</dependency>
```

Usage
------
Before using it, you need integrate it with PostgresDriver maybe like this:
```scala
import slick.driver.PostgresDriver
import com.github.tminglei.slickpg._

trait MyPostgresDriver extends PostgresDriver
                          with PgArraySupport
                          with PgRangeSupport
                          with PgHStoreSupport
                          with PgSearchSupport
                          with PostGISSupport {

  override val Implicit = new ImplicitsPlus {}
  override val simple = new SimpleQLPlus {}

  //////
  trait ImplicitsPlus extends Implicits
                        with ArrayImplicits
                        with RangeImplicits
                        with HStoreImplicits
                        with SearchImplicits
                        with PostGISImplicits

  trait SimpleQLPlus extends SimpleQL
                        with ImplicitsPlus
                        with SearchAssistants
}

object MyPostgresDriver extends MyPostgresDriver

```

then in your codes you can use it like this:
```scala
import MyPostgresDriver.simple._

object TestTable extends Table[Test](Some("xxx"), "Test") {
  def id = column[Long]("id", O.AutoInc, O.PrimaryKey)
  def during = column[Range[Timestamp]]("during")
  def location = column[Point]("location")
  def text = column[String]("text", O.DBType("varchar(4000)"))
  def props = column[Map[String,String]]("props_hstore")
  def tags = column[List[String]]("tags_arr")

  def * = id ~ during ~ location ~ text ~ props ~ tags <> (Test, Test unapply _)

  ///
  def byId(ids: Long*) = TestTable.where(_.id inSetBind ids).map(t => t)
  // will generate sql like: select * from test where tags && ?
  def byTag(tags: String*) = TestTable.where(_.tags @& tags.toList.bind).map(t => t)
  // will generate sql like: select * from test where during && ?
  def byTsRange(tsRange: Range[Timestamp]) = TestTable.where(_.during @& tsRange.bind).map(t => t)
  // will generate sql like: select * from test where case(props -> ? as [T]) == ?
  def byProperty[T](key: String, value: T) = TestTable.where(_.props.>>[T](key.bind) === value.bind).map(t => t)
  // will generate sql like: select * from test where ST_DWithin(location, ?, ?)
  def byDistance(point: Point, distance: Int) = TestTable.where(r => r.location.dWithin(point.bind, distance.bind)).map(t => t)
  // will generate sql like: select id, text, ts_rank(to_tsvector(text), to_tsquery(?)) from test where to_tsvector(text) @@ to_tsquery(?) order by ts_rank(to_tsvector(text), to_tsquery(?))
  def search(queryStr: String) = TestTable.where(tsVector(_.text) @@ tsQuery(queryStr.bind)).map(r => (r.id, r.text, tsRank(tsVector(r.text), tsQuery(queryStr.bind)))).sortBy(_._3)
}

...
 
```

Data types/operators/functions
------------------------------
- [Array's operators/functions](https://github.com/tminglei/slick-pg/tree/master/src/main/scala/com/github/tminglei.slickpg#array "Array's operators/functions")
- [Range's operators/functions](https://github.com/tminglei/slick-pg/tree/master/src/main/scala/com/github/tminglei.slickpg#range "Range's operators/functions")
- [HStore's operators/functions](https://github.com/tminglei/slick-pg/tree/master/src/main/scala/com/github/tminglei.slickpg#hstore "HStore's operators/functions")
- [Search's operators/functions](https://github.com/tminglei/slick-pg/tree/master/src/main/scala/com/github/tminglei.slickpg#search "Search's operators/functions")
- [Geometry's operators/functions](https://github.com/tminglei/slick-pg/tree/master/src/main/scala/com/github/tminglei.slickpg#geometry "Geometry's operators/functions")


Version history
---------------
v0.1.1 (8-Jul-2013):  
1) supplement pg array support  

v0.1.0 (20-May-2013):  
1) support pg array  
2) support pg range  
3) support pg hstore  
4) support pg search  
5) support pg geometry  


License
-------
Licensing conditions (BSD-style) can be found in LICENSE.txt.
