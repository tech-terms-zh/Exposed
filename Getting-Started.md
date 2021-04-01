## 下載

### Maven

```xml
<repositories>
  <repository>
    <id>jcenter</id>
    <name>jcenter</name>
    <url>https://jcenter.bintray.com</url>
  </repository>
</repositories>

<dependencies>
    <dependency>
      <groupId>org.jetbrains.exposed</groupId>
      <artifactId>exposed-core</artifactId>
      <version>0.29.1</version>
    </dependency>
    <dependency>
      <groupId>org.jetbrains.exposed</groupId>
      <artifactId>exposed-dao</artifactId>
      <version>0.29.1</version>
    </dependency>
    <dependency>
      <groupId>org.jetbrains.exposed</groupId>
      <artifactId>exposed-jdbc</artifactId>
      <version>0.29.1</version>
    </dependency>
</dependencies>

```

### Gradle Kotlin Script

如果你使用舊版的 Gradle，需要在 `build.gradle` 內加上以下內容：

```
repositories {
  jcenter()
}
dependencies {
  compile("org.jetbrains.exposed", "exposed-core", "0.29.1")
  compile("org.jetbrains.exposed", "exposed-dao", "0.29.1")
  compile("org.jetbrains.exposed", "exposed-jdbc", "0.29.1")
}
```

如果你使用舊版的 Gradle，可以在 `build.gradle.kts` 內加上以下內容：

```
val exposedVersion: String by project
dependencies {
    implementation("org.jetbrains.exposed:exposed-core:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-dao:$exposedVersion")
    implementation("org.jetbrains.exposed:exposed-jdbc:$exposedVersion")
}
```

`gradle.properties` 內調整版本號：

```
exposedVersion=0.29.1
```

- 備註：還有其他的模組可使用。細節詳見 [模組相關文件|LibDocumentation]]

## 開始

### 開始交易

每個使用 Exposed 存取資料庫的行為，都從取得連線並建立交易開始。

取得連線：

```kotlin
Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")
```

要處理進階的操作，像是連線匯集池（connection pooling），也可以提供 `javax.sql.DataSource` 進行操作：

```kotlin
Database.connect(dataSource)
```

到 [[資料庫和資料來源|DataBase-and-DataSource]] 看更多細節

在連線後，所有的 SQL 敘述（statement）都需被放在一個交易處理（transaction）中

```kotlin
transaction {
  // Statements here
}
```

要看實際呼叫資料庫的內容，可以加上一個 logger：

```kotlin
transaction {
  // print sql to std-out
  addLogger(StdOutSqlLogger)
}
```

### 領域專用語言和資料存取物件

Exposed 兩種使用方式：領域專用語言（DSL，Domain Specific Language）和資料存取物件（DAO，Data Access Object）  
簡而言之，領域專用語言，是用型別安全（type-safe）的語法來操作 SQL。資料存取物件，則是用元件的方式執行 CRUD 操作。
先看看下面的範例，然後到各個 API 專屬的章節，看更多的細節。

### 第一個 Exposed 領域專用語言

```kotlin


fun main(args: Array<String>) {
  //an example connection to H2 DB
  Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")

  transaction {
    // print sql to std-out
    addLogger(StdOutSqlLogger)

    SchemaUtils.create (Cities)

    // insert new city. SQL: INSERT INTO Cities (name) VALUES ('St. Petersburg')
    val stPeteId = Cities.insert {
      it[name] = "St. Petersburg"
    } get Cities.id

    // 'select *' SQL: SELECT Cities.id, Cities.name FROM Cities
    println("Cities: ${Cities.selectAll()}")
  }
}

object Cities: IntIdTable() {
    val name = varchar("name", 50)
}

```

到 [[領域專用語言 API|DSL]] 看更多教學

### 第一個 Exposed 資料存取物件

```kotlin


fun main(args: Array<String>) {
  //an example connection to H2 DB
  Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")

  transaction {
    // print sql to std-out
    addLogger(StdOutSqlLogger)

    SchemaUtils.create (Cities)

    // insert new city. SQL: INSERT INTO Cities (name) VALUES ('St. Petersburg')
    val stPete = City.new {
            name = "St. Petersburg"
    }

    // 'select *' SQL: SELECT Cities.id, Cities.name FROM Cities
    println("Cities: ${City.all()}")
  }
}

object Cities: IntIdTable() {
    val name = varchar("name", 50)
}

class City(id: EntityID<Int>) : IntEntity(id) {
    companion object : IntEntityClass<City>(Cities)

    var name by Cities.name
}
```

到 [[資料存取物件 API|DAO]] 看更多教學

或者，回到 [[介紹|Home]]
