### apache-cayenne
---
https://cayenne.apache.org/

```sh
mvn cayenne:cdbimport
mvn cayenne:cgen
```

```py
buildscript {
  repositories {
    mavenCantral()
  }
  dependencies {
    classpath group: 'org.apache.cayenne.plugins', name: 'org.apache.cayenne.plugin', version: 'cayenne-gradle-plugin', version: '4.1.B2'
    classpath 'mysql:mysql-connector-java:8.0.13'
  }
}

apply plugin: 'org.apache.cayenne'
cayenne.defaultDataMap 'demo.map.xml'

cdbimport {
  cayenneProject 'cayenne-demo.xml'
  
  dataSource {
    driver 'com.mysql.cj.jdbc.Driver'
    url 'jdbc:mysql://127.0.0.1:3306/cayenne_demo'
    username: 'user'
    password: 'password'
  }
  
  dbImport {
    defaultPackage = 'org.apache.cayenne.demo.model'
  }
}

cgen.dependsOn cdbimport
compileJava.dependesOn cgen


Serverruntime cayenneRuntime = ServerRuntime.builder()
  .addConfig("cayenne-demo.xml")
  .dataSource(DataSourceBuilder
    .url("jbc:mysql://localhost::3306/cayenne_demo")
    .driver("com.mysql.cj.jdbc.Driver")
    .userName("username")
    .password("password")
    .build())
  .build();


ObjectContext context = cayenneRuntime.newContext();

Artist picasso = context.newObject(Artist.class);
picasso.setName("Pablo Picasso");
picasso.setDataOfBirth(LocalData.of(1881, 10, 25));

Gallery metropolitan = context.newObject(Gallery.class);
metropolitan.setName("Metropolitan Museum of Art");

Painting girl = context.newObject(Painting.class);
girl.setName("Girl Reading at a Table");

Painting stein = context.newObject(Painting.class);
stein.setName("Gertrude Stein");

picasso.addToPaintings(girl);
picasso.addToPaintings(stein);

girl.setGallery(metropolitan);
stein.setGallery(metropolitan);

context.commitChanges();


List<Painting> paintings = ObjectSelect.query(Painting.class)
  .where(Painting.ARTIST.dot(Artist.DATE_OF_BIRTH).lt(LocalDate.of(1900, 1, 1)))
  .prefetch(Painting.ARTIST.joint())
  .select(context);

Property<Artist> artistProperty = Property.createSelf(Artist.class);

List<Object[]> artistAndPaintingCount = ObjectSelect.columnQuery(Artist.class, artistProperty, Artist.PAINTING_ARRAY.count())
  .where(Artist.ARTIST_NAME.like("a%"))
  .having(Artist.PAINTING_ARRAY().lt(5L))
  .orderBy(Artist.PAINTING_ARRAY.count().desc(), Artist.ARTIST_NAME.asc())
  .select(context);
for (Object[] next : artistAndPaintingCount) {
  Artist artist = (Artist)next[0];
  long paintingsCount = (Long)next[1];
  System.out.println(artist.getArtistName() + " has " + paintingsCount + " painting(s)");
}

List<Painting> paintings = SQLSelect
  .query(Painting.class, "SELECT * FROM PAINTING WHERE PAINTING_TILTE LIKE #bind($title)")
  .params("title", "painting%")
  .upperColumnNames()
  .localCache()
  .limit(100)
  .select(context);

List<String> paintingNames = SQLSelect
  .scalarQuery(String.class, "SELECT PAINTING_TILTE FROM PAINTING WHERE ESTIMATED_PRICE > #bind($price)")
  .params("price", 100000)
  .select(context);

int inserted = SQLExec
  .query("INSERT INTO ARTIST (ARTIST_ID, ARTIST_NAME) VALUES (#bind($id), #bind(#name))")
  .paramsArray(55, "Picasso")
  .update(context);

```

```gradle
compile group: 'org.apache.cayenne', name: 'cayenne-server', version: '4.1.B2'
compile cayenne.dependency('server')
```
