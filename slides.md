---
theme: apple-basic
background: https://source.unsplash.com/collection/94734566/1920x1080
lineNumbers: true
drawings:
  persist: false
routerMode: history
selectable: true
remoteAssets: true
colorSchema: 'dark'
layout: intro
canvasWidth: 800
---

#  Advanced Kotlin Techniques for Spring Developers


## Pasha Finkelshteyn <logos-kotlin-icon /><logos-spring-icon />

<img src="/bellsoft.png" width="200px" class="absolute right-10px bottom-10px"/>


---
layout: image-right
image: 'avatar.jpg'
---
# `whoami`

<v-clicks>

- <div v-after>Pasha Finkelshteyn</div>
- Dev <noto-v1-avocado /> at BellSoft
- ≈10 years in JVM. Mostly <logos-java /> and <logos-kotlin-icon />
- And <logos-spring-icon />
- <logos-twitter /> asm0di0
- <logos-mastodon-icon /> @asm0dey@fosstodon.org

</v-clicks>

---
layout: two-cols-header
---

# BellSoft

::left::

* Vendor of Liberica JDK
* Contributor to the OpenJDK
* Author of ARM32 support in JDK

Liberica is the JDK officially recommended by <logos-spring-icon />

<v-click><b>We know our stuff!</b></v-click>

::right::

<img src="/news.png" class="invert rounded self-center"/>



---
layout: statement
---

# I hope to showcase something you don't know yet!

---

# My application

<v-clicks>

- Simple nano-service
- MVC
- Validation
- JPA
- JDBC
- Tests

</v-clicks>

---

# Where do I start?

https://start.spring.io

![](/settings.png)

---

# Minimum dependencies

<img src="/deps.png" class="max-h-310px"/>

[Full config](https://start.spring.io/#!type=gradle-project-kotlin&language=kotlin&platformVersion=3.0.2&packaging=jar&jvmVersion=17&groupId=com.github.asm0dey&artifactId=sample&name=sample&description=Demo%20project%20for%20Spring%20Boot&packageName=com.github.asm0dey.sample&dependencies=data-jpa,postgresql,validation,web,security,testcontainers)


---

# 2 files are generated

- `build.gradle.kts`
- `SpringKotlinStartApplication.kt`

---

# What happens

```kotlin 
plugins {
	kotlin("jvm") version "2.0.20"
	kotlin("plugin.spring") version "2.0.20"
	id("org.springframework.boot") version "3.3.3"
	id("io.spring.dependency-management") version "1.1.6"
	kotlin("plugin.jpa") version "2.0.20"
}

group = "com.example"
version = "0.0.1-SNAPSHOT"

java {
	toolchain {
		languageVersion = JavaLanguageVersion.of(21)
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation("org.springframework.boot:spring-boot-starter-data-jpa")
	implementation("org.springframework.boot:spring-boot-starter-data-r2dbc")
	implementation("org.springframework.boot:spring-boot-starter-security")
	implementation("org.springframework.boot:spring-boot-starter-validation")
	implementation("org.springframework.boot:spring-boot-starter-web")
	implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
	implementation("io.projectreactor.kotlin:reactor-kotlin-extensions")
	implementation("org.jetbrains.kotlin:kotlin-reflect")
	implementation("org.jetbrains.kotlinx:kotlinx-coroutines-reactor")
	runtimeOnly("org.postgresql:postgresql")
	runtimeOnly("org.postgresql:r2dbc-postgresql")
	testImplementation("org.springframework.boot:spring-boot-starter-test")
	testImplementation("io.projectreactor:reactor-test")
	testImplementation("org.jetbrains.kotlin:kotlin-test-junit5")
	testImplementation("org.springframework.security:spring-security-test")
	testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

kotlin {
	compilerOptions {
		freeCompilerArgs.addAll("-Xjsr305=strict")
	}
}

tasks.withType<Test> {
	useJUnitPlatform()
}
```

---
layout: section
---

# The main class

---

# Main class

```kotlin {all|4,6|8}
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class SampleApplication

fun main(args: Array<String>) {
  runApplication<SampleApplication>(*args)
}
```

---
canvasWidth: 600
---

# `runApplication`

```kotlin {all|1|2}
inline fun <reified T : Any> runApplication(vararg args: String) =
		SpringApplication.run(T::class.java, *args)
```

<v-click>

The first goodie of Spring for Kotlin

</v-click>

---
layout: statement
---

# Let's start implementing

## Chapter 1. MVC + Validation

---

# First controller

```kotlin {all|2|4|5}
@RestController
@RequestMapping("/person")
class PersonController {
  @PostMapping
  fun createPerson(@RequestBody @Valid person: Person) {}
}
```

<v-click>

`Person.kt`:
```kotlin {at:'+1'}
data class Person(
  val name: String,
  val age: Int
)
```

</v-click>

---

# Make an empty `POST`…

<div v-click.hide="'2'">

```http {all|1|2}{at:'0'}
POST localhost:8080/person
Content-Type: application/json
```

</div>
<div v-click="'2'">

```http {all|5-7}{at:'3'}
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
  "timestamp": 1674735741056,
  "status": 400,
  "error": "Bad Request",
  "path": "/person"
}
```

</div>
<div v-click="'4'">

Since `Person` is non-nullable — it's validated without `@NotNull` annotation

</div>

---

# Why? How?

`build.greadle.kts`:
```kotlin {all|1|3}
tasks.withType<KotlinCompile> {
	kotlinOptions {
		freeCompilerArgs = listOf("-Xjsr305=strict")
		jvmTarget = "21"
	}
}
```
<v-click>

## `JSR 305: Annotations for Software Defect Detection`:

> Nullness annotations (e.g., `@NonNull` and `@CheckForNull`)

> Internationalization annotations, such as `@NonNls` or `@Nls`

</v-click>

---

# Non-empty `POST` with empty properties

```http
POST localhost:8080/person
Content-Type: application/json

{"name": null, "age": null}
```

<div v-click> On client </div>
<div v-click>

```http {all|1}{at:3}
HTTP/1.1 400 Bad Request
Content-Type: application/json
```

</div>

<v-click> On server </v-click>
<div v-click="'4'">

```plain {all|2}{at:'5'}
…Instantiation of [simple type, class com.github.asm0dey.sample.Person] 
  value failed for JSON property name due to missing
```

</div>



---
layout: two-cols-header
---

# `POST` with non-empty name

::left::

```http {all|4}
POST localhost:8080/person
Content-Type: application/json

{"name": "Pasha", "age": null}
```

<div v-click="'2'">

```http {all|1}{at:'3'}
HTTP/1.1 200 
Content-Length: 0
```

</div>

::right::


<img src="/surprised.png" class="h-80 shadow rounded ml-16" v-click="'4'" />


---
layout: two-cols-header
---

::left ::
# Rechecking

```kotlin {all|3}
data class Person(
  val name: String,
  val age: Int
)
```


::right::

<v-click>

<img src="/right.jpg" class="shadow rounded " />

</v-click>


---
layout: statement
---

# In JVM primitive types have default values

---

# These types will be JVM primitives:

- `Double`
- `Int`
- `Float`
- `Char`
- `Short`
- `Byte`
- `Boolean`


---

# Updating class



```kotlin {all|3}{at:'1'}
data class Person(
  val name: String,
  @field:NotNull val age: Int?
)
```

<div v-click="'2'">

```http {all|4}{at:3}
POST localhost:8080/person
Content-Type: application/json

{"name": "Pasha", "age": null}
```

</div>
<v-click at="4">

```http
HTTP/1.1 400 Bad Request
…
{ "timestamp": 1674760360096, "status": 400, "error": "Bad Request", "path": "/person" }
```

</v-click>
<v-click at="5">

```plain
Field error in object 'person' on field 'age': rejected value [null]
```

<h2 class="text-center">Hooray!<twemoji-party-popper /></h2>

</v-click>

---

## Also…

```yaml
spring:
  jackson:
    deserialization:
      FAIL_ON_NULL_FOR_PRIMITIVES: true
```

---

# Quick summary

- `-Xjsr305=strict` will make the validation easier
- For JVM primitive types we have to put `@field:NotNull` and mark them nullable
- Sometimes can work it around with jackson settings


---
layout: section
---

# JPA

## Chapter 2

---
clicks: 3
---

# Nanoentity

```kotlin {all|2|7,9|2,10}
@Entity
data class Person(
  @Id
  @GeneratedValue(strategy = IDENTITY)
  var id: Int? = null,
  @Column(nullable = false)
  val name: String,
  @Column(nullable = false)
  val age: Int,
)
```

<ul>
  <li v-click="1">
  <span><code>data</code> class</span>
  </li>
  <li v-click="2">
  <span><code>val name</code> and <code>val age</code></span>
  </li>
  <li v-click="3">
  <span>No no-arg constructor</span>
  </li>
</ul>

---

# Improving

<div v-click.hide="'2'">

`data` classes have `copy`, `equals`, `hashCode`, `copy`, and `componentX` defined

</div>

````md magic-move
```kotlin {all|2}
@Entity
data class Person(
  @Id
  @GeneratedValue(strategy = IDENTITY)
  var id: Int? = null,
  @Column(nullable = false)
  val name: String,
  @Column(nullable = false)
  val age: Int,
)
```
```kotlin {2|7,9}
@Entity
class Person(
  @Id
  @GeneratedValue(strategy = IDENTITY)
  var id: Int? = null,
  @Column(nullable = false)
  val name: String,
  @Column(nullable = false)
  val age: Int,
)
```
```kotlin {7,9|3,4,6,8}
@Entity
class Person(
  @Id
  @GeneratedValue(strategy = IDENTITY)
  var id: Int? = null,
  @Column(nullable = false)
  var name: String,
  @Column(nullable = false)
  var age: Int,
)
```
````

<v-click at=3>

JPA won's be able to write to `val`

</v-click>


---

# But there is no no-arg constructor!

How to make it work?

Magic:
```kotlin
kotlin("plugin.jpa") version "2.0.20"
```

<v-clicks>

- Puts annotations on the fields
- Adds a default constructor in bytecode*!

<small>* In Kotlin the default constructor would not be possible, but in Java it is</small>

</v-clicks>

---

# Current result

```kotlin
@Entity
class Person(
  @Id
  @GeneratedValue(strategy = IDENTITY)
  var id: Int? = null,
  @Column(nullable = false)
  var name: String,
  @Column(nullable = false)
  var age: Int,
)
```

---

# Is this enough?

Not quite.

At the very least we have to redefine `equals` and `hashCode`.

For example…
```kotlin {all|2-4|4,9|6-8}
@Entity
class Person(
  // properties
) {
  // equals…
  override fun hashCode(): Int {
    return id ?: 0
  }
}
```

---
layout: section
---

# JDBC

## Chapter 3

---

# Obtain user by id

Let's imagine we need to call the following:
```sql
SELECT *
FROM  users
WHERE id = ?
```

---

# Or, for example...

```sql {1-18|18-22}{maxHeight:'340px'}
SELECT DISTINCT book.id
              , (SELECT COALESCE(JSON_GROUP_ARRAY(JSON_ARRAY(t.v0, t.v1, t.v2, t.v3, t.v 4, t.v5, t.v6, t.v7, t.v8,
                                                             t.v9)), JSON_ARRAY())
                 FROM (SELECT b.id AS v0 , b.path AS v1 , b.name AS v2 , b.date AS v3 , b.added AS v4 , b.sequence AS v5
                            , b.sequence_number AS v6 , b.lang AS v7 , b.zip_file AS v8 , b.seqid AS v9 FROM book AS b
                       WHERE b.id = book.id) AS t)                AS book
              , (SELECT COALESCE(JSON_GROUP_ARRAY(JSON_ARRAY(t.v0, t.v1, t.v2, t.v3, t.v4, t.v5, t.v6)), JSON_ARRAY())
                 FROM (SELECT DISTINCT author.id AS v0 , author.fb2id AS v1 , author.first _name AS v2
                                    , author.middle_name AS v3 , author.last_name AS v4 , author.nickname AS v5 , author.added AS v6
                       FROM author
                                JOIN book_author ON book_author.author_id = author.id
                       WHERE book_author.book_id = book.id) AS t) AS authors
              , (SELECT COALESCE(JSON_GROUP_ARRAY(JSON_ARRAY(t.v0, t.v1)), JSON_ARRAY())
                 FROM (SELECT DISTINCT genre.name AS v0, genre.id AS v1
                       FROM genre
                                JOIN book_genre ON book_genre.genre_id = genre.id
                       WHERE book_genre.book_id = book.id) AS t)  AS genres
              , book.sequence
FROM book
         JOIN book_author ON book_author.book_id = book.id
WHERE (book.seqid = 40792 AND book_author.author_id = 34606)
ORDER BY book.sequence_number ASC NULLS LAST, book.name
``` 

---

# In Java

<logos-java /> <span v-click="'9'" v-click.hide="'10'">Let's inline mapper</span>
````md magic-move
```java {all|1|2|5-13|7|8|9|10|11}
public List<Person> findById(int id) {
  return jdbcTemplate.query("SELECT * FROM users WHERE id = ?", new UserRowMapper(), id);
}

private static class UserRowMapper implements RowMapper<Person> {
  @Override
  public Person mapRow(ResultSet resultSet, int i) throws SQLException {
    int id = resultSet.getInt("id");
    String name = resultSet.getString("name");
    Double age = resultSet.getDouble("age");
    return new Person(id, name, age);
  }
}
```
```java {all|2|3-6|7}
public List<Person> findById(int userId) {
  return jdbcTemplate.query("SELECT * FROM users WHERE id = ?", (resultSet, i) -> {
    int id = resultSet.getInt("id");
    String name = resultSet.getString("name");
    Double age = resultSet.getDouble("age");
    return new Person(id1, name, age);
  }, userId);
}
```
````
---

# I don't like it

- Too many mappers
- Parameters are too far from query

<twemoji-loudly-crying-face />


---

# Why should it be so?

<v-clicks>

Let's Look at the signature

```java
public <T> List<T> query(String sql, RowMapper<T> rowMapper, @Nullable Object... args)
```

Because in <logos-java /> vararg can be only the last… <twemoji-sad-but-relieved-face />

</v-clicks>
---

# `JdbcTemplate` in Kotlin <flat-color-icons-entering-heaven-alive />

```kotlin {all|1|2-5}
return jdbcTemplate.query("SELECT * FROM users WHERE id = ?", userId) { rs, _ ->
  val id = rs.getInt("id")
  val name = rs.getString("name")
  val age = rs.getDouble("age")
  Person(id, name, age)
}
```

<v-click>

- `vararg` doesn't have to be in the last position
- unused parameter of a lambda can be named `_`

</v-click>


---

# Extension functions

```kotlin {all|1|2|3|4}
fun <T> JdbcOperations.query(
  sql: String,
  vararg args: Any,
  function: (ResultSet, Int) -> T
): List<T>
```

<v-click>

Which allows
```kotlin
return jdbcTemplate.query("SELECT * FROM users WHERE id = ?", userId) 
{ rs, _ ->
  // TODO: ResultSet → Person
}
```

</v-click>

---

# You can do the same for your own code!

From <logos-java />
```java
public List<String> transformStrings(Function<String, String> mapper, String... args){}
```

<v-click>

To <logos-kotlin-icon />
```kotlin
fun transformStrings(vararg args: String, mapper: (String) -> String): List<String>{}
```

</v-click>

---
layout: two-cols-header
---
# More on extensions for Spring

::left::

- [spring-beans](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-beans/index.html)
- [spring-context](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-context/index.html)
- [spring-core](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-core/index.html)
- [spring-jdbc](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-jdbc/index.html)
- [spring-messaging](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-messaging/index.html)
- [spring-r2dbc](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-r2dbc/index.html)

::right::


- [spring-test](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-test/index.html)
- [spring-tx](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-tx/index.html)
- [spring-web](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-web/index.html)
- [spring-webflux](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-webflux/index.html)
- [spring-webmvc](https://docs.spring.io/spring-framework/docs/6.0.4/kdoc-api/spring-webmvc/index.html)


---
layout: section
---

# Reactive persistence

## Chapter 4

---
layout: two-cols-header
---

# Who knows what is reactive?

::left::

<v-clicks>

- non-blocking
- asynchronous
- handles back-pressure

</v-clicks>
<v-click>

The result is a monad...

</v-click>

::right::

<img src="/jet.png" class="h-80 shadow rounded ml-20"/>

---
layout: two-cols-header
---

# Who knows what is reactive?

::left::


- non-blocking
- asynchronous
- handles back-pressure


The result is either:
- `Mono`: produces 0 to 1 items when it's ready
- `Flux`: Produces 0 to ∞ items

::right::

<img src="/jet.png" class="h-80 shadow rounded ml-20"/>

---

# In Kotlin

Coroutines operate on `suspend` functions

`suspend` can be called from:

- coroutine context
- another `suspend` `fun`

returns usual types - Lists, Strings, etc

```kotlin {1|2|3|4}
suspend fun randomNum(): Int {
  val x = service1.randomNum() // suspend
  val y = service2.randomNum() // suspend
  return x + y
}
``` 

---

<img src="/suspend.jpg" class="h-90 shadow rounded ml-20" />

---

# Spring R2DBC

````md magic-move
```kotlin {none|2|3|all}
@Repository
class Repo(connectionFactory: ConnectionFactory) {
    val client = DatabaseClient.create(connectionFactory)
}
```
```kotlin {4|5|6|7|8}
@Repository
class Repo(connectionFactory: ConnectionFactory) {
    val client = DatabaseClient.create(connectionFactory)
    suspend fun createUserAndReturnId() =
        client
            .sql("INSERT INTO users (name, email, age) VALUES ('Pasha', :email, NULL) RETURNING id")
            .bind("email", RandomStringUtils.randomAlphabetic(20))
            .fetch()
            .awaitSingle()["id"] as? Long ?: error("not long on not returned")
}
```
```kotlin {9-13}
@Repository
class Repo(connectionFactory: ConnectionFactory) {
    val client = DatabaseClient.create(connectionFactory)
    suspend fun createUserAndReturnId() =
        client
            .sql("INSERT INTO users (name, email, age) VALUES ('Pasha', :email, NULL) RETURNING id")
            .bind("email", RandomStringUtils.randomAlphabetic(20))
            .fetch()
            .awaitSingle()["id"] as? Long ?: error("not long on not returned")
    // inside Spring ↓
    suspend fun <T> RowsFetchSpec<T>.awaitSingle(): T {
      return first().awaitSingleOrNull() ?: throw EmptyResultDataAccessException(1)
    } 
}
```
```kotlin {4-9}
@Repository
class Repo(connectionFactory: ConnectionFactory) {
    val client = DatabaseClient.create(connectionFactory)
    suspend fun createUserAndReturnId() =
        client
            .sql("INSERT INTO users (name, email, age) VALUES ('Pasha', :email, NULL) RETURNING id")
            .bind("email", RandomStringUtils.randomAlphabetic(20))
            .fetch()
            .awaitSingle()["id"] as? Long ?: error("not long on not returned")
}
```
````

---

# Transactions

```kotlin {3|4|1|all}
@Transactional
suspend fun failTransactional(): Long {
    val curId =  repo.createUserAndReturnId()
    repo.createUserWithConflictingId(curId) // will throw
    return curId
}
```

<p v-click>Nothing changes from the client standpoint!</p>

---

# Repositories

````md magic-move
```kotlin {1}
interface User : CoroutineCrudRepository<User, Long> {
}
```
```kotlin {1,4|6|7-8}
interface User : CoroutineCrudRepository<User, Long> {
}
// Inside Spring
interface CoroutineCrudRepository<T, ID> : Repository<T, ID> {
  // ...
  suspend fun <S : T> save(entity: S): T
  // Flow is an async unbounded collection
  fun findAll(): Flow<T>
  // ...
}
```
```kotlin
interface User : CoroutineCrudRepository<User, Long> {
}
```
```kotlin {2|3|4}
interface User : CoroutineCrudRepository<User, Long> {
  suspend fun findOne(id: String): User
  fun findByFirstname(firstname: String): Flow<User>
  suspend fun findAllByFirstname(id: String): List<User>
}
```
````

<p v-click>Function is either suspend or returns `Flow`</p>

---

# Transactions with repositories?

```kotlin {none|3|4|5|all}
@Transactional
suspend fun failTransactional(): Long {
  val u = User(null, "me@asm0dey.site", 37)
  val savedId =  repo.save(u)
  repo.save(u.copy(id = savedId)) // will throw
  return curId
}
```


---
layout: section
---

# Configuration

---

# Let's start simple

```kotlin {all|2}
val beans = beans {
  bean { jacksonObjectMapper() }
}
```

<div v-click="'2'">

Modified Jackson's `ObjectMapper` to work with `data` classes from `jackson-module-kotlin`

```kotlin {all|1|3|2}{at:'3'}
@Bean
fun kotlinMapper(): ObjectMapper {
  return jacksonObjectMapper()
}
```

</div>
<div v-click="'6'">

4 lines instead of 1 <twemoji-face-screaming-in-fear />

</div>

---

# Custom bean

```kotlin {1-7|1|3-5|11|11,1}
class JsonLogger(private val objectMapper: ObjectMapper) {
  fun log(o: Any) {
    if (o::class.isData) {
      println(objectMapper.writeValueAsString(o))
    } else println(o.toString())
  }
}

val beans = beans {
  bean { jacksonObjectMapper() }
  bean(::JsonLogger)
}
```

---

# Arbitrary logic

```kotlin {all|4-6|5}
val beans = beans {
  bean { jacksonObjectMapper() }
  bean(::JsonLogger)
  bean("randomGoodThing", isLazyInit = Random.nextBoolean()) {
    if (Random.nextBoolean()) "Norway" else "Well"
  }
}
```

---

# OK How do I use it?

Let's return to our very first file

```kotlin
runApplication<SampleApplication>(*args)
```
```kotlin
val beans = { /* */ }
```

<div v-click="'1'">

Let's change it to
```kotlin {all|2}{at:'2'}
runApplication<SampleApplication>(*args) {
  addInitializers(beans)
}
```

</div>
<div v-click="'3'">

And run it…
```plain
Started SampleApplicationKt in 1.776 seconds (process running for 2.133)
```
<twemoji-party-popper />

</div>

---

# Let's test it
Bean:
```kotlin {all|2|3|all}
@Component
class MyBean(val jsonLogger: JsonLogger) {
  fun test() = jsonLogger.log("Test")
}
```
Test:
```kotlin {all|1|3|5}{at:'3'}
@SpringBootTest
class ConfigTest {
  @Autowired private lateinit var myBean: MyBean
  @Test
  fun testIt() = assertEquals("Test", myBean.test())
}
```

---
layout: two-cols-header
---

# Run it

::left::

```
No qualifying bean of type 
'com.github.asm0dey.sample.JsonLogger'
  available: expected at least 1 
  bean which qualifies as autowire candidate
```

<div v-click="'2'">

That's because our tests do not call `main`!

</div>

::right::

<img src="/explosion.png" v-click="'1'" class="h-80 rounded ml-16">



---

# Requires some glue to work

```kotlin {all|1|2|3}
val beans = { /* */ }
class BeansInitializer : ApplicationContextInitializer<GenericApplicationContext> {
  override fun initialize(context: GenericApplicationContext) = beans.initialize(context)
}
```

<div v-click>

`application.yml`:

```yaml
context.initializer.classes: "com.github.asm0dey.sample.BeansInitializer"
```

</div>
<div v-click="'5'">

`Main.kt`:
```kotlin {all|2}{at:6}
fun main(args: Array<String>) {
  runApplication<SampleApplication>(*args)
}
```

</div>

---
layout: section
---

# Security

---

# Spring Security

```kotlin {none|1|2-6|7|8|9|10|11|12|13|14,15|18|7-19}{maxHeight:'320px'}
val beans = beans {
  bean { jacksonObjectMapper() }
  bean(::JsonLogger)
  bean("random", isLazyInit = Random.nextBoolean()) {
    if (Random.nextBoolean()) "Norway" else "Well"
  }
  bean {
    val http = ref<HttpSecurity>()
    http {
      csrf { disable() }
      httpBasic { }
      securityMatcher("/**")
      authorizeRequests {
        authorize("/auth/**", authenticated)
        authorize(anyRequest, permitAll)
      }
    }
    http.build()
  }
}
```

---
layout: section
---

# So, what did I learn?

---

# So, what did I learn?

<v-clicks>

- Always generate the project with start.spring.io
- Reified generics might make an API better
- Validation is better with Kotlin, but remember about primitives
- `data` classes should not be used for JPA
- JDBC is simpler with Kotlin
- R2DBC is simpler with coroutines
- Bean definition DSL is awesome
- Specifically with security!

</v-clicks>

---
layout: statement
---

# Thank you!

---

# Thank you! Questions?



- <logos-twitter /> asm0di0
- <logos-mastodon-icon /> @asm0dey@fosstodon.org
- <logos-google-gmail /> me@asm0dey.site
- <logos-linkedin-icon /> asm0dey
- <logos-telegram /> asm0dey
- <logos-whatsapp-icon /> asm0dey
- <skill-icons-instagram /> asm0dey
- <logos-facebook /> asm0dey

<img src="/news.png" width="200px" class="absolute right-10px bottom-10px invert rounded"/>

---
layout: end
---
