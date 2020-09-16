# Creating the Android application

The configuration we need was configured by the KMM Android Studio plugin, so we have our KMM module already connected to our Android application. Before implementing the UI and presentation logic, let's add all the required dependencies to the `androidApp/build.gradle.kts`:

```kotlin
// ... 

dependencies {
    implementation(project(":shared")) 
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9")
    implementation("androidx.core:core-ktx:1.3.1")
    implementation("com.google.android.material:material:1.2.0")
    implementation("androidx.appcompat:appcompat:1.2.0")
    implementation("androidx.constraintlayout:constraintlayout:2.0.0")
    implementation("androidx.recyclerview:recyclerview:1.1.0")
    implementation("androidx.swiperefreshlayout:swiperefreshlayout:1.1.0")
    implementation("androidx.cardview:cardview:1.0.0")
}

// ... 
```

## Implementing the UI: display the list of rocket launches

Let's modify `activity_main.xml` to implement the UI we need. The screen is based on the `ConstraintLayout` with the `SwipeRefreshLayout` inside it, which contains `RecyclerView` and `FrameLayout` with a background with `ProgressBar` across its center. You can view all the layout code in the [commit](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/commit/27fcb711582ba3b069650c3d385663f521e426be) for this stage.

Now let's add the properties for the UI elements to the `MainActivity` class: 

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var launchesRecyclerView: RecyclerView
    private lateinit var progressBarView: FrameLayout
    private lateinit var swipeRefreshLayout: SwipeRefreshLayout

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        title = "SpaceX Launches"
        setContentView(R.layout.activity_main)

        launchesRecyclerView = findViewById(R.id.launchesListRv)
        progressBarView = findViewById(R.id.progressBar)
        swipeRefreshLayout = findViewById(R.id.swipeContainer)
    }
}
```

For the `RecyclerView` element to work, we need to create an adapter (as a subclass of `RecyclerView.Adapter`) that will convert raw data into list item views. For this purpose we'll create the `LaunchesRvAdapter` class and `item_launch.xml` with the items view layout:

```kotlin
class LaunchesRvAdapter(var launches: List<RocketLaunch>) : RecyclerView.Adapter<LaunchesRvAdapter.LaunchViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): LaunchViewHolder {
        return LayoutInflater.from(parent.context)
            .inflate(R.layout.item_launch, parent, false)
            .run(::LaunchViewHolder)
    }

    override fun getItemCount(): Int = launches.count()

    override fun onBindViewHolder(holder: LaunchViewHolder, position: Int) {
        holder.bindData(launches[position])
    }

    inner class LaunchViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        // ...

        fun bindData(launch: RocketLaunch) {
            // ...
        }
    }
}
```

You can view the full XML layout code and the full implementation of the RecyclerView adapter in the [commit](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/commit/27fcb711582ba3b069650c3d385663f521e426be) for this stage.

Now we need to create an instance of `LaunchesRvAdapter` in the `MainActivity` class, configure the `RecyclerView` component, add a listener to the `SwipeRefreshLayout` to catch the screen refresh gesture, and implement all the LaunchesListView interface functions:

```kotlin
class MainActivity : AppCompatActivity() {
    // ...

    private val launchesRvAdapter = LaunchesRvAdapter(listOf())

    override fun onCreate(savedInstanceState: Bundle?) {
        // ...

        launchesRecyclerView.adapter = launchesRvAdapter
        launchesRecyclerView.layoutManager = LinearLayoutManager(this)

        swipeRefreshLayout.setOnRefreshListener {
            swipeRefreshLayout.isRefreshing = false
            displayLaunches(true) 
        }

        displayLaunches()
    }

    private fun displayLaunches(needReload: Boolean) {
        // TODO: Presentation logic
    }
}
```

## Implementing presentation logic

First, we need to create an instance of the `SpaceXSDK` class from our KMM module and inject an instance of `DatabaseDriverFactory` in it:

```kotlin
class MainActivity : AppCompatActivity() {
    // ...
    private val sdk = SpaceXSDK(DatabaseDriverFactory(this)) 
}
```

Then let's implement the private function `displayLaunches(needReload: Boolean)`. It will run the `getLaunches` function inside the coroutine launched in the main CoroutineScope, handle exceptions, and display the error text in the toast:

```kotlin
class MainActivity : AppCompatActivity() {
    private val mainScope = MainScope()

    // ...

    override fun onDestroy() {
        super.onDestroy()
        mainScope.cancel()
    }

    // ...

    private fun displayLaunches(needReload: Boolean) {
        progressBarView.isVisible = true
        mainScope.launch {
            kotlin.runCatching {
                sdk.getLaunches(needReload)
            }.onSuccess {
                launchesRvAdapter.launches = it
                launchesRvAdapter.notifyDataSetChanged()
            }.onFailure {
                Toast.makeText(this@MainActivity, it.localizedMessage, Toast.LENGTH_SHORT).show()
            }
            progressBarView.isVisible = false
        }
    }
}
```

All done! Select "androidApp" from the run configurations menu, choose emulator, and press the run button:

<img alt="Android application" src="./assets/android-application.png" width="300">

Congratulations! You've just created an Android application, which has its business logic implemented in Kotlin Multiplatform Mobile module. To earn the title of "cross-platform Kotlin mobile developer" let’s take it even further and implement an iOS application!

You can find the state of the project after this section in [this commit](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/commit/355ce6e4ddb662e9fc978e10bfcc73c075bdaa5b) on the final branch in the GitHub repository.

