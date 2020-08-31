# Configuring SQLDelight and implementing cache logic

## Configuring SQLDelight

The SQLDelight library allows us to generate a type-safe Kotlin database API from SQL queries. During compilation, the generator validates the SQL queries and turns them into Kotlin code that can be used in the KMM module.

We already added the library to the project in the previous stage. Now we need to configure it. To do this, we need to add the sqldelight block, which will contain a list of databases and their parameters, to the end of the KMM module `build.gradle` file:

```kotlin
sqldelight {
    database("AppDatabase") {
        packageName = "com.jetbrains.handson.kmm.shared.cache"
    }
}
```

The packageName parameter specifies the package name for the generated Kotlin sources. 

There is an official [plugin](https://cashapp.github.io/sqldelight/multiplatform_sqlite/intellij_plugin/) for working with **.sq** files.

## Generating the database API

First, we need to create the **.sq** file, which will contain all the SQL queries that are needed. By default, SQLDelight plugin reads **.sq** from the `sqldelight` folder.  So in the `kmm-library/commonMain` directory we need to create the `sqldelight` directory. There we need to create the package specified in the packageName parameter (in our case, the `com/jetbrains/handson/kmm/shared/cache` directory path). Inside it, we need to create an **.sq** file with the name of our database (in our case, `AppDatabase.sq`).

Now let's put all the SQL queries that our application will use into this file.
Our database will contain two tables:
*   A table with data about launches
*   A table with data about rockets

To create these tables, we need to add the following code to the `AppDatabase.sq` file:

```sql
CREATE TABLE Launch (
    flightNumber INTEGER NOT NULL,
    missionName TEXT NOT NULL,
    launchYear INTEGER AS Int NOT NULL DEFAULT 0,
    rocketId TEXT NOT NULL,
    details TEXT,
    launchSuccess INTEGER AS Boolean DEFAULT NULL,
    launchDateUTC TEXT NOT NULL,
    missionPatchUrl TEXT,
    articleUrl TEXT
);

CREATE TABLE Rocket (
    id TEXT NOT NULL PRIMARY KEY,
    name TEXT NOT NULL,
    type TEXT NOT NULL
);
```

We need to be able to insert data into these tables. To do this, we declare SQL insert functions:

```sql
insertLaunch:
INSERT INTO Launch(flightNumber, missionName, launchYear, rocketId, details, launchSuccess, launchDateUTC, missionPatchUrl, articleUrl)
VALUES(?, ?, ?, ?, ?, ?, ?, ?, ?);

insertRocket:
INSERT INTO Rocket(id, name, type)
VALUES(?, ?, ?);
```

To clear the tables of data we need to declare SQL delete functions:

```sql
removeAllLaunches:
DELETE FROM Launch;

removeAllRockets:
DELETE FROM Rocket;
```

We can declare functions to retrieve data in the same way. In our case, we want to retrieve data about a rocket using its identifier and select information about all its launches using a JOIN statement:

```sql
selectRocketById:
SELECT * FROM Rocket
WHERE id = ?;

selectAllLaunchesInfo:
SELECT Launch.*, Rocket.*
FROM Launch
LEFT JOIN Rocket ON Rocket.id == Launch.rocketId;
```

When the project is compiled, the generated Kotlin code will be stored in the `/shared/build/generated/sqldelight` directory. The generator will create an interface with the named `AppDatabase`, as we specified in `build.gradle.kts` 

You can learn more about how to work with SQLDelight in the [KMM documentation](https://helpserver.labs.jb.gg/help/kotlin-mobile/configure-sqldelight-for-data-storage.html).

## Creating platform database drivers

To initialize `AppDatabase`, we need to pass an `SqlDriver` instance to it. SQLDelight provides multiple platform-specific implementations of the SQLite driver, so we need to create them for each platform separately. With `expect`/`actual` this can be done easily. First, we can create an abstract factory for database drivers. To do this, create a `com.jetbrains.handson.kmm.shared.cache` package in the `shared/src/commonMain/kotlin` directory and create a `DatabaseDriverFactory` class inside it:

```kotlin
package com.jetbrains.handson.kmm.shared.cache

import com.squareup.sqldelight.db.SqlDriver

expect class DatabaseDriverFactory {
    fun createDriver(): SqlDriver
}
```

Next we need to provide actual implementations for this expected class. 

On Android, the SQLite driver is implemented by the `AndroidSqliteDriver` class. We need to pass the database information and the link to the context to `AndroidSqliteDriver` class constructor. To do this, let's create `com.jetbrains.handson.kmm.shared.cache` package in the `shared/src/androidMain/kotlin` directory and create a `DatabaseDriverFactory` class inside it with the actual implementation:

```kotlin
actual class DatabaseDriverFactory(private val context: Context) {
    actual fun createDriver(): SqlDriver {
        return AndroidSqliteDriver(AppDatabase.Schema, context, "test.db")
    }
}
```

Let's do the same for iOS. On iOS, the SQLite driver implementation is the `NativeSqliteDriver` class. We need to create a `com.jetbrains.handson.kmm.shared.cache` package in the `shared/src/iosMain/kotlin` directory and create a `DatabaseDriverFactory` class inside it with the actual implementation:

```kotlin
import com.jetbrains.handson.kmm.shared.cache.AppDatabase
import com.squareup.sqldelight.db.SqlDriver
import com.squareup.sqldelight.drivers.native.NativeSqliteDriver

actual class DatabaseDriverFactory {
    actual fun createDriver(): SqlDriver {
        return NativeSqliteDriver(AppDatabase.Schema, "test.db")
    }
}
```

We will create instances of these factories later in the code of the Android and iOS projects.

Hint! You can navigate throw `expect` declarations and `actual` realizations with the handy gutter:

<img alt="Expect/Actual gutter" src="./assets/expect-actual.png" width="700">

## Implementing cache 

Now we have platform database drivers and an `AppDatabase` class to perform database operations. Let's create a `Database` class, which will wrap the `AppDatabase` class and contain caching logic.

This will be common to both platform logics, so we need to create a new Database class in the `com.jetbrains.handson.kmm.shared.cache` package in the common source set (`shared/src/commonMain/kotlin` path). To provide a driver for `AppDatabase`, we will pass an abstract `DatabaseDriverFactory` to the `Database` class constructor:

```kotlin
package com.jetbrains.handson.kmm.shared.cache

import com.jetbrains.handson.kmm.shared.cache.AppDatabase

internal class Database(databaseDriverFactory: DatabaseDriverFactory) {
    private val database = AppDatabase(databaseDriverFactory.createDriver())
    private val dbQuery = database.appDatabaseQueries
}
```

This class's visibility is set to internal, which means it is only accessible from within the multiplatform module.

Now we need to implement some data handling operations. First, we'll add a function to clear all the tables in the database in a single SQL transaction:

```kotlin
internal fun clearDatabase() {
    dbQuery.transaction {
        dbQuery.removeAllRockets()
        dbQuery.removeAllLaunches()
    }
}
```

Next, we create a function to get a list of all the rocket launches:

```kotlin
internal fun getAllLaunches(): List<RocketLaunch> {
    return dbQuery.selectAllLaunchesInfo(::mapLaunchSelecting).executeAsList()
}

private fun mapLaunchSelecting(
    flightNumber: Long,
    missionName: String,
    launchYear: Int,
    rocketId: String,
    details: String?,
    launchSuccess: Boolean?,
    launchDateUTC: String,
    missionPatchUrl: String?,
    articleUrl: String?,
    rocket_id: String?,
    name: String?,
    type: String?
): RocketLaunch {
    return RocketLaunch(
        flightNumber = flightNumber.toInt(),
        missionName = missionName,
        launchYear = launchYear,
        details = details,
        launchDateUTC = launchDateUTC,
        launchSuccess = launchSuccess,
        rocket = Rocket(
            id = rocketId,
            name = name!!,
            type = type!!
        ),
        links = Links(
            missionPatchUrl = missionPatchUrl,
            articleUrl = articleUrl
        )
    )
}
```


The argument we pass to `selectAllLaunchesInfo` is a function that maps the database entity class to another type. In this case, we're mapping to the `RocketLaunch` data model class.

Next, we add a function to insert data into the database:

```kotlin
internal fun createLaunches(launches: List<RocketLaunch>) {
    dbQuery.transaction {
        launches.forEach { launch ->
            val rocket = dbQuery.selectRocketById(launch.rocket.id).executeAsOneOrNull()
            if (rocket == null) {
                insertRocket(launch)
            }

            insertLaunch(launch)
        }
    }
}

private fun insertRocket(launch: RocketLaunch) {
    dbQuery.insertRocket(
        id = launch.rocket.id,
        name = launch.rocket.name,
        type = launch.rocket.type
    )
}

private fun insertLaunch(launch: RocketLaunch) {
    dbQuery.insertLaunch(
        flightNumber = launch.flightNumber.toLong(),
        missionName = launch.missionName,
        launchYear = launch.launchYear,
        rocketId = launch.rocket.id,
        details = launch.details,
        launchSuccess = launch.launchSuccess ?: false,
        launchDateUTC = launch.launchDateUTC,
        missionPatchUrl = launch.links.missionPatchUrl,
        articleUrl = launch.links.articleUrl
    )
}
```

All done! We will create an instance of the `Database` class later on in step 7, when we create our SDK facade class.

You can find the state of the project after this section in [this commit](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/commit/565161bcc3726990ff1d8e5f4d67e27e8ec87e84) on the final branch in the GitHub repository.
