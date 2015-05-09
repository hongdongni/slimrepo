# Slim Repo [![Build Status](https://travis-ci.org/slim-gears/slimrepo.svg?branch=master)](https://travis-ci.org/slim-gears/slimrepo) [![Maven Repository](https://img.shields.io/github/release/slim-gears/slimrepo.svg?label=Maven)](https://bintray.com/slim-gears/slimrepo/slimrepo-android/_latestVersion)
### Light-weight modular ORM for Java and Android

##### The library is still under development. Stay tuned for updates.

Background
---

The library was inspired by [GreenDAO](http://greendao-orm.com/ "GreenDAO") and [Microsoft Entity Framework Code First](https://msdn.microsoft.com/en-us/data/ee712907) 

#### Terminology

`Entity` - Data object, POJO 
 
`Repository` - represents abstract working session, *unit-of-work* against ORM

`RepositoryService` - factory, allowing to create `Repository` instances 

#### Features

* **Intuitive syntax** - intuitive, type-safe and highly readable syntax
(underlying persistent storages - e.g. Sqlite, document db, remote RESTful service, etc.)
* **Annotation processing based** - no reflection usage in run-time, *proguard-friendly*
* **Bulk operations support** - *Bulk update* and *bulk delete* are supported
* **Light-weight** - simple and has a low package footprint

---

Installation for Android project
---
**Step 1.** Enable annotation processing for your project (if not enabled yet)
```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}

apply plugin: 'com.neenbedankt.android-apt'

configurations {
    apt
}
```
**Step 2.** Add jCenter repository (if not added yet)
```gradle
repositories {
	jcenter()
}
```
**Step 3.** Add dependencies
```gradle
dependencies {
    compile 'com.slimgears.slimrepo:slimrepo-android:0.7.0'
    apt 'com.slimgears.slimrepo:slimrepo-apt:0.7.0'
}

```

Repository Definition
---

##### Entity definition
**Step 1:** Define one or more entities:
```java
@GenerateEntity
public class AbstractUserEntity {
    protected int userId;
    protected String firstName;
    protected String lastName;
    protected Date lastVisitDate;
}
```
Real entity will be named `UserEntity`, and will be generated by annotation processor.

##### Repository definition
**Step 2:** Define one or more repositories. It's just an interface containing one or more `EntitySets`
```java
@GenerateRepository
@OrmProvider(SqliteOrmServiceProvider.class)
public interface UserRepository extends Repository {
    EntitySet<UserEntity> users();
    EntitySet<CountryEntity> countries();
    EntitySet<OrderEntity> orders();
}
```
Insert
---

First, create `RepositoryService<UserRepository>` instance by instantiating generated `UserRepositoryService`

```java
UserRepositoryService repoService = new GeneratedUserRepositoryService(context);
```

**Option 1** - create repository instance using `RepositoryService.open()`: 

```java
try (UserRepository repo = repoService.open()) {
	EntitySet<UserEntity> users = repo.users();

	// Possible syntax
	users.add(UserEntity.create()
		.setFirstName("John")
		.setLastName("Doe")
		.setLastVisitDate(Dates.now()));

	// Alternative syntax
	users.add(UserEntity.builder()
		.firstName("William")
		.lastName("Shakespeare")
		.lastVisitDate(Dates.now())
		.build());

	// Explicitly saving changes
	repo.saveChanges();
}
```

**Option 2** - using `RepositoryService.update()` method.
In this case changes will be saved automatically:

```java
repoService.update(repo -> {
	EntitySet<UserEntity> users = repo.users();

	// Possible syntax
	users.add(UserEntity.create()
		.setFirstName("John")
		.setLastName("Doe")
		.setLastVisitDate(Dates.now()));

	// Alternative syntax
	users.add(UserEntity.create()
		.firstName("William")
		.lastName("Shakespeare")
		.lastVisitDate(Dates.now())
		.build());
});
```

**Option 3** - for short operations there is a convenient ability to use auto-connecting entity set, performing temporary connections:

```java
repoService.users().add(
	UserEntity.create()
		.setFirstName("John")
		.setLastName("Doe")
		.setLastVisitDate(Dates.now()),
	UserEntity.create()
		.setFirstName("William")
		.setLastName("Shakespeare"));
```

Query
---

**Option 1** - using `RepositoryService.open()`: 

```java
try (UserRepository repo = repoService.open()) {
	long count = repo.users().query()
		.where(UserEntity.LastVisitDate.between(Dates.yesterday(), Dates.today())
		.prepare()
		.count();
}
```

**Option 2** - using `RepositoryService.query()`: 

```java 
UserEntity[] users = repoService.query(repo -> {
	return repo.users().query()
		.where(UserEntity.UserFirstName.contains("a"))
		.prepare()
		.toArray();
});
```

**Option 3** - for short operations there is a convenient ability to use auto-connecting entity set, performing queries using temporary connections:

```java
UserEntity[] users = repoService.users()
		.query()
		.where(UserEntity.UserFirstName.contains("a"))
		.prepare()
		.toArray();
```

**Using and / or condition compositions**

```java
UserEntity[] users = repoService.query(repo -> {
	return repo.users().query()
		.where(Conditions
			.and(
				UserEntity.FirstName.contains("a"),
				UserEntity.LastName.endsWith("e"))
			.or(UserEntity.LastVisitDate.greaterThan(Dates.today()))
		.prepare()
		.toArray();
});
```

Update
---

**Single entity update**: 

```java
repoService.update(repo -> {
	UserEntity user = repo.users().findById(2);
	user.setLastName("Smith");
});
```

**Bulk update**: Updating all entities, matching `where` criteria:

```java 
repoService.update(repo -> {
	return repo.users().update()
		.where(UserEntity.UserFirstName.contains("a"))
		.set(UserEntity.LastVisitDate, Dates.now())
		.prepare()
		.execute();
});
```

Delete
---

**Delete single entity directly**

```java
repoService.update(repo -> {
	UserEntity user = repo.users().findById(2);
	repo.users().remove(user);
});
```

**Bulk delete**: Deleting all entities, matching `where` criteria

```java
repoService.update(repo -> {
	return repo.users().delete()
		.where(UserEntity.UserFirstName.contains("a"))
		.prepare()
		.execute();
});
```
