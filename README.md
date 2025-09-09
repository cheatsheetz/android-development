# Android Development Cheat Sheet

## Quick Reference

Android development involves creating apps for Android devices using Kotlin/Java, Android Studio, and Google's Android SDK.

### Setup and Installation

```bash
# Download Android Studio from https://developer.android.com/studio

# Install Android SDK via Android Studio SDK Manager
# Tools -> SDK Manager

# Set environment variables
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/platform-tools

# Create new project
# Android Studio -> Create New Project

# Run app
./gradlew assembleDebug
adb install app/build/outputs/apk/debug/app-debug.apk

# Or use Android Studio Run button (Shift+F10)
```

## Kotlin Fundamentals

### Variables and Basic Types

```kotlin
// Variables
var mutableVariable = "Can be changed"
val immutableVariable = "Cannot be changed"

// Basic types
val number: Int = 42
val longNumber: Long = 42L
val floatNumber: Float = 3.14f
val doubleNumber: Double = 3.14159
val boolean: Boolean = true
val character: Char = 'A'
val text: String = "Hello, Kotlin"

// Nullable types
var nullableString: String? = null
val length = nullableString?.length ?: 0

// Collections
val list = listOf("item1", "item2", "item3")
val mutableList = mutableListOf("item1", "item2")
val map = mapOf("key1" to "value1", "key2" to "value2")
val mutableMap = mutableMapOf("key1" to "value1")

// Arrays
val array = arrayOf(1, 2, 3, 4, 5)
val intArray = intArrayOf(1, 2, 3, 4, 5)
```

### Functions and Classes

```kotlin
// Functions
fun greet(name: String): String {
    return "Hello, $name!"
}

// Single expression function
fun greet(name: String) = "Hello, $name!"

// Function with default parameters
fun greet(name: String = "World", greeting: String = "Hello") = "$greeting, $name!"

// Extension functions
fun String.addExclamation() = this + "!"

// Classes
class Person(val name: String, var age: Int) {
    // Secondary constructor
    constructor(name: String) : this(name, 0)
    
    // Method
    fun introduce() = "Hi, I'm $name and I'm $age years old"
    
    // Computed property
    val isAdult: Boolean
        get() = age >= 18
}

// Data class
data class User(val id: Int, val name: String, val email: String)

// Enum class
enum class Status {
    PENDING, APPROVED, REJECTED
}

// Sealed class
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val exception: Exception) : Result<Nothing>()
    object Loading : Result<Nothing>()
}

// Object (Singleton)
object ApiClient {
    fun makeRequest() {
        // Implementation
    }
}
```

### Coroutines and Async

```kotlin
import kotlinx.coroutines.*

// Suspend function
suspend fun fetchUserData(userId: String): User {
    delay(1000) // Simulate network call
    return User(userId, "John Doe", "john@example.com")
}

// Coroutine scope
class UserRepository {
    private val scope = CoroutineScope(Dispatchers.IO + SupervisorJob())
    
    suspend fun getUser(id: String): User {
        return withContext(Dispatchers.IO) {
            // Network call
            fetchUserData(id)
        }
    }
}

// In Activity/Fragment
lifecycleScope.launch {
    try {
        val user = userRepository.getUser("123")
        updateUI(user)
    } catch (e: Exception) {
        handleError(e)
    }
}

// Flow (reactive streams)
class DataRepository {
    fun getUserFlow(id: String): Flow<User> = flow {
        while (true) {
            emit(fetchUserData(id))
            delay(5000) // Emit every 5 seconds
        }
    }
}

// Collect flow
lifecycleScope.launch {
    dataRepository.getUserFlow("123")
        .collect { user ->
            updateUI(user)
        }
}
```

## Android Components

### Activities

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Initialize views
        setupViews()
    }
    
    override fun onStart() {
        super.onStart()
        // Activity is visible
    }
    
    override fun onResume() {
        super.onResume()
        // Activity is in foreground and interactive
    }
    
    override fun onPause() {
        super.onPause()
        // Activity is partially obscured
    }
    
    override fun onStop() {
        super.onStop()
        // Activity is no longer visible
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // Activity is being destroyed
    }
    
    private fun setupViews() {
        val button: Button = findViewById(R.id.button)
        button.setOnClickListener {
            startActivity(Intent(this, SecondActivity::class.java))
        }
    }
}

// Activity with result launcher
class MainActivity : AppCompatActivity() {
    
    private val resultLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == RESULT_OK) {
            val data = result.data?.getStringExtra("key")
            // Handle result
        }
    }
    
    private fun startSecondActivity() {
        val intent = Intent(this, SecondActivity::class.java)
        intent.putExtra("data", "Hello from MainActivity")
        resultLauncher.launch(intent)
    }
}
```

### Fragments

```kotlin
class UserFragment : Fragment() {
    
    private var _binding: FragmentUserBinding? = null
    private val binding get() = _binding!!
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentUserBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        setupViews()
        observeData()
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
    
    private fun setupViews() {
        binding.button.setOnClickListener {
            // Handle click
        }
    }
    
    companion object {
        fun newInstance(userId: String): UserFragment {
            return UserFragment().apply {
                arguments = Bundle().apply {
                    putString("userId", userId)
                }
            }
        }
    }
}

// Fragment transaction
supportFragmentManager.beginTransaction()
    .replace(R.id.fragment_container, UserFragment.newInstance("123"))
    .addToBackStack(null)
    .commit()
```

### Services

```kotlin
// Started Service
class BackgroundService : Service() {
    
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val data = intent?.getStringExtra("data")
        
        // Perform background work
        performBackgroundTask(data)
        
        return START_NOT_STICKY
    }
    
    override fun onBind(intent: Intent?): IBinder? = null
    
    private fun performBackgroundTask(data: String?) {
        // Background work here
        stopSelf() // Stop service when done
    }
}

// Foreground Service
class ForegroundService : Service() {
    
    override fun onCreate() {
        super.onCreate()
        createNotificationChannel()
    }
    
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = createNotification()
        startForeground(NOTIFICATION_ID, notification)
        
        // Perform long-running task
        
        return START_NOT_STICKY
    }
    
    override fun onBind(intent: Intent?): IBinder? = null
    
    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                "Service Channel",
                NotificationManager.IMPORTANCE_LOW
            )
            val manager = getSystemService(NotificationManager::class.java)
            manager.createNotificationChannel(channel)
        }
    }
    
    companion object {
        const val CHANNEL_ID = "ServiceChannel"
        const val NOTIFICATION_ID = 1
    }
}

// Bound Service
class BoundService : Service() {
    
    inner class LocalBinder : Binder() {
        fun getService(): BoundService = this@BoundService
    }
    
    private val binder = LocalBinder()
    
    override fun onBind(intent: Intent?): IBinder = binder
    
    fun performTask(): String {
        return "Task completed"
    }
}
```

### Broadcast Receivers

```kotlin
class NetworkReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context?, intent: Intent?) {
        when (intent?.action) {
            ConnectivityManager.CONNECTIVITY_ACTION -> {
                val isConnected = isNetworkAvailable(context)
                // Handle network change
            }
        }
    }
    
    private fun isNetworkAvailable(context: Context?): Boolean {
        val connectivityManager = context?.getSystemService(Context.CONNECTIVITY_SERVICE) 
            as ConnectivityManager
        val activeNetwork = connectivityManager.activeNetwork
        return activeNetwork != null
    }
}

// Register receiver dynamically
class MainActivity : AppCompatActivity() {
    
    private val networkReceiver = NetworkReceiver()
    
    override fun onResume() {
        super.onResume()
        val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
        registerReceiver(networkReceiver, filter)
    }
    
    override fun onPause() {
        super.onPause()
        unregisterReceiver(networkReceiver)
    }
}
```

## UI Development

### Views and ViewGroups

```kotlin
// Programmatic view creation
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val linearLayout = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.MATCH_PARENT
            )
        }
        
        val textView = TextView(this).apply {
            text = "Hello, Android!"
            textSize = 24f
            gravity = Gravity.CENTER
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
        }
        
        val button = Button(this).apply {
            text = "Click me"
            setOnClickListener {
                Toast.makeText(this@MainActivity, "Button clicked!", Toast.LENGTH_SHORT).show()
            }
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            ).apply {
                gravity = Gravity.CENTER_HORIZONTAL
                topMargin = 16.dpToPx()
            }
        }
        
        linearLayout.addView(textView)
        linearLayout.addView(button)
        setContentView(linearLayout)
    }
    
    private fun Int.dpToPx(): Int {
        return (this * resources.displayMetrics.density).toInt()
    }
}
```

### RecyclerView

```kotlin
// Data class
data class Item(val id: Int, val title: String, val description: String)

// ViewHolder
class ItemViewHolder(private val binding: ItemLayoutBinding) : 
    RecyclerView.ViewHolder(binding.root) {
    
    fun bind(item: Item, onItemClick: (Item) -> Unit) {
        binding.titleText.text = item.title
        binding.descriptionText.text = item.description
        
        binding.root.setOnClickListener {
            onItemClick(item)
        }
    }
}

// Adapter
class ItemAdapter(
    private var items: List<Item>,
    private val onItemClick: (Item) -> Unit
) : RecyclerView.Adapter<ItemViewHolder>() {
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ItemViewHolder {
        val binding = ItemLayoutBinding.inflate(
            LayoutInflater.from(parent.context),
            parent,
            false
        )
        return ItemViewHolder(binding)
    }
    
    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) {
        holder.bind(items[position], onItemClick)
    }
    
    override fun getItemCount() = items.size
    
    fun updateItems(newItems: List<Item>) {
        items = newItems
        notifyDataSetChanged()
    }
}

// Usage in Activity/Fragment
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    private lateinit var adapter: ItemAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        setupRecyclerView()
        loadData()
    }
    
    private fun setupRecyclerView() {
        adapter = ItemAdapter(emptyList()) { item ->
            // Handle item click
            Toast.makeText(this, "Clicked: ${item.title}", Toast.LENGTH_SHORT).show()
        }
        
        binding.recyclerView.apply {
            layoutManager = LinearLayoutManager(this@MainActivity)
            adapter = this@MainActivity.adapter
            
            // Add divider
            addItemDecoration(
                DividerItemDecoration(this@MainActivity, DividerItemDecoration.VERTICAL)
            )
        }
    }
    
    private fun loadData() {
        val items = listOf(
            Item(1, "Item 1", "Description 1"),
            Item(2, "Item 2", "Description 2"),
            Item(3, "Item 3", "Description 3")
        )
        adapter.updateItems(items)
    }
}
```

### View Binding

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // Access views through binding
        binding.titleText.text = "Hello, Android!"
        binding.button.setOnClickListener {
            binding.titleText.text = "Button clicked!"
        }
    }
}

// In Fragment
class UserFragment : Fragment() {
    
    private var _binding: FragmentUserBinding? = null
    private val binding get() = _binding!!
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentUserBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

## Data Storage

### SharedPreferences

```kotlin
class PreferencesManager(private val context: Context) {
    
    private val preferences = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE)
    
    fun saveString(key: String, value: String) {
        preferences.edit().putString(key, value).apply()
    }
    
    fun getString(key: String, defaultValue: String = ""): String {
        return preferences.getString(key, defaultValue) ?: defaultValue
    }
    
    fun saveInt(key: String, value: Int) {
        preferences.edit().putInt(key, value).apply()
    }
    
    fun getInt(key: String, defaultValue: Int = 0): Int {
        return preferences.getInt(key, defaultValue)
    }
    
    fun saveBoolean(key: String, value: Boolean) {
        preferences.edit().putBoolean(key, value).apply()
    }
    
    fun getBoolean(key: String, defaultValue: Boolean = false): Boolean {
        return preferences.getBoolean(key, defaultValue)
    }
    
    companion object {
        private const val PREFS_NAME = "app_preferences"
    }
}

// Usage with data classes
inline fun <reified T> PreferencesManager.saveObject(key: String, obj: T) {
    val gson = Gson()
    val json = gson.toJson(obj)
    saveString(key, json)
}

inline fun <reified T> PreferencesManager.getObject(key: String, clazz: Class<T>): T? {
    val json = getString(key)
    return if (json.isNotEmpty()) {
        Gson().fromJson(json, clazz)
    } else null
}
```

### Room Database

```kotlin
// Entity
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Int,
    val name: String,
    val email: String,
    @ColumnInfo(name = "created_at") val createdAt: Long = System.currentTimeMillis()
)

// DAO (Data Access Object)
@Dao
interface UserDao {
    
    @Query("SELECT * FROM users")
    suspend fun getAllUsers(): List<User>
    
    @Query("SELECT * FROM users WHERE id = :userId")
    suspend fun getUserById(userId: Int): User?
    
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertUser(user: User)
    
    @Update
    suspend fun updateUser(user: User)
    
    @Delete
    suspend fun deleteUser(user: User)
    
    @Query("DELETE FROM users WHERE id = :userId")
    suspend fun deleteUserById(userId: Int)
}

// Database
@Database(
    entities = [User::class],
    version = 1,
    exportSchema = false
)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() {
    
    abstract fun userDao(): UserDao
    
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null
        
        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}

// Repository
class UserRepository(private val userDao: UserDao) {
    
    suspend fun getAllUsers(): List<User> = userDao.getAllUsers()
    
    suspend fun getUserById(id: Int): User? = userDao.getUserById(id)
    
    suspend fun insertUser(user: User) = userDao.insertUser(user)
    
    suspend fun updateUser(user: User) = userDao.updateUser(user)
    
    suspend fun deleteUser(user: User) = userDao.deleteUser(user)
}
```

## Architecture Patterns

### MVVM with LiveData

```kotlin
// ViewModel
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    private val _loading = MutableLiveData<Boolean>()
    val loading: LiveData<Boolean> = _loading
    
    private val _error = MutableLiveData<String>()
    val error: LiveData<String> = _error
    
    fun loadUsers() {
        viewModelScope.launch {
            _loading.value = true
            try {
                val userList = repository.getAllUsers()
                _users.value = userList
            } catch (e: Exception) {
                _error.value = e.message
            } finally {
                _loading.value = false
            }
        }
    }
    
    fun addUser(name: String, email: String) {
        viewModelScope.launch {
            try {
                val user = User(0, name, email)
                repository.insertUser(user)
                loadUsers() // Refresh the list
            } catch (e: Exception) {
                _error.value = e.message
            }
        }
    }
}

// ViewModelFactory
class UserViewModelFactory(
    private val repository: UserRepository
) : ViewModelProvider.Factory {
    
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(UserViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return UserViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}

// Activity
class MainActivity : AppCompatActivity() {
    
    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: UserViewModel
    private lateinit var adapter: UserAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        setupViewModel()
        setupRecyclerView()
        observeData()
        
        viewModel.loadUsers()
    }
    
    private fun setupViewModel() {
        val database = AppDatabase.getDatabase(this)
        val repository = UserRepository(database.userDao())
        val factory = UserViewModelFactory(repository)
        viewModel = ViewModelProvider(this, factory)[UserViewModel::class.java]
    }
    
    private fun observeData() {
        viewModel.users.observe(this) { users ->
            adapter.updateUsers(users)
        }
        
        viewModel.loading.observe(this) { isLoading ->
            binding.progressBar.visibility = if (isLoading) View.VISIBLE else View.GONE
        }
        
        viewModel.error.observe(this) { errorMessage ->
            if (!errorMessage.isNullOrEmpty()) {
                Toast.makeText(this, errorMessage, Toast.LENGTH_LONG).show()
            }
        }
    }
}
```

### Dependency Injection with Dagger Hilt

```kotlin
// Application class
@HiltAndroidApp
class MyApplication : Application()

// Network module
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    
    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
    
    @Provides
    @Singleton
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}

// Database module
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {
    
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "app_database"
        ).build()
    }
    
    @Provides
    fun provideUserDao(database: AppDatabase): UserDao {
        return database.userDao()
    }
}

// Repository
@Singleton
class UserRepository @Inject constructor(
    private val apiService: ApiService,
    private val userDao: UserDao
) {
    suspend fun getUsers(): List<User> {
        return try {
            val users = apiService.getUsers()
            userDao.insertAll(users)
            users
        } catch (e: Exception) {
            userDao.getAllUsers()
        }
    }
}

// ViewModel with Hilt
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {
    
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    fun loadUsers() {
        viewModelScope.launch {
            _users.value = repository.getUsers()
        }
    }
}

// Activity with Hilt
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Setup UI and observe ViewModel
    }
}
```

## Networking

### Retrofit

```kotlin
// Data models
data class User(
    val id: Int,
    val name: String,
    val email: String
)

data class ApiResponse<T>(
    val data: T,
    val message: String,
    val success: Boolean
)

// API interface
interface ApiService {
    
    @GET("users")
    suspend fun getUsers(): List<User>
    
    @GET("users/{id}")
    suspend fun getUser(@Path("id") userId: Int): User
    
    @POST("users")
    suspend fun createUser(@Body user: User): User
    
    @PUT("users/{id}")
    suspend fun updateUser(@Path("id") userId: Int, @Body user: User): User
    
    @DELETE("users/{id}")
    suspend fun deleteUser(@Path("id") userId: Int): Response<Void>
    
    @GET("search")
    suspend fun searchUsers(@Query("q") query: String): List<User>
    
    @Multipart
    @POST("upload")
    suspend fun uploadFile(
        @Part("description") description: RequestBody,
        @Part file: MultipartBody.Part
    ): ResponseBody
}

// Network client
object NetworkClient {
    
    private val okHttpClient = OkHttpClient.Builder()
        .addInterceptor(HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        })
        .addInterceptor { chain ->
            val original = chain.request()
            val requestBuilder = original.newBuilder()
                .header("Authorization", "Bearer $authToken")
                .header("Content-Type", "application/json")
            
            val request = requestBuilder.build()
            chain.proceed(request)
        }
        .connectTimeout(30, TimeUnit.SECONDS)
        .readTimeout(30, TimeUnit.SECONDS)
        .build()
    
    val retrofit: Retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .client(okHttpClient)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
    
    val apiService: ApiService = retrofit.create(ApiService::class.java)
}

// Repository with error handling
class NetworkRepository {
    
    private val apiService = NetworkClient.apiService
    
    suspend fun getUsers(): Result<List<User>> {
        return try {
            val users = apiService.getUsers()
            Result.Success(users)
        } catch (e: IOException) {
            Result.Error("Network error: Check your connection")
        } catch (e: HttpException) {
            val errorBody = e.response()?.errorBody()?.string()
            Result.Error("Server error: $errorBody")
        } catch (e: Exception) {
            Result.Error("Unknown error: ${e.message}")
        }
    }
}

// Sealed class for results
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
    object Loading : Result<Nothing>()
}
```

## Testing

### Unit Testing

```kotlin
// Test dependencies in build.gradle
testImplementation 'junit:junit:4.13.2'
testImplementation 'org.mockito:mockito-core:4.6.1'
testImplementation 'org.mockito.kotlin:mockito-kotlin:4.0.0'
testImplementation 'kotlinx-coroutines-test:1.6.0'

// Unit test example
class UserRepositoryTest {
    
    @Mock
    private lateinit var apiService: ApiService
    
    @Mock
    private lateinit var userDao: UserDao
    
    private lateinit var repository: UserRepository
    
    @Before
    fun setup() {
        MockitoAnnotations.openMocks(this)
        repository = UserRepository(apiService, userDao)
    }
    
    @Test
    fun `getUsers should return users from API when network call succeeds`() = runTest {
        // Given
        val expectedUsers = listOf(
            User(1, "John Doe", "john@example.com"),
            User(2, "Jane Smith", "jane@example.com")
        )
        whenever(apiService.getUsers()).thenReturn(expectedUsers)
        
        // When
        val result = repository.getUsers()
        
        // Then
        assertEquals(expectedUsers, result)
        verify(apiService).getUsers()
        verify(userDao).insertAll(expectedUsers)
    }
    
    @Test
    fun `getUsers should return cached users when network call fails`() = runTest {
        // Given
        val cachedUsers = listOf(User(1, "Cached User", "cached@example.com"))
        whenever(apiService.getUsers()).thenThrow(IOException("Network error"))
        whenever(userDao.getAllUsers()).thenReturn(cachedUsers)
        
        // When
        val result = repository.getUsers()
        
        // Then
        assertEquals(cachedUsers, result)
        verify(userDao).getAllUsers()
    }
}

// ViewModel test
class UserViewModelTest {
    
    @get:Rule
    val instantExecutorRule = InstantTaskExecutorRule()
    
    @Mock
    private lateinit var repository: UserRepository
    
    private lateinit var viewModel: UserViewModel
    
    @Before
    fun setup() {
        MockitoAnnotations.openMocks(this)
        viewModel = UserViewModel(repository)
    }
    
    @Test
    fun `loadUsers should update users LiveData`() = runTest {
        // Given
        val expectedUsers = listOf(User(1, "Test User", "test@example.com"))
        whenever(repository.getUsers()).thenReturn(expectedUsers)
        
        // When
        viewModel.loadUsers()
        
        // Then
        assertEquals(expectedUsers, viewModel.users.value)
        verify(repository).getUsers()
    }
}
```

### Instrumentation Testing

```kotlin
// Test dependencies in build.gradle
androidTestImplementation 'androidx.test.ext:junit:1.1.5'
androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
androidTestImplementation 'androidx.test:runner:1.5.2'
androidTestImplementation 'androidx.test:rules:1.5.0'

// Instrumentation test
@RunWith(AndroidJUnit4::class)
class MainActivityTest {
    
    @get:Rule
    val activityRule = ActivityScenarioRule(MainActivity::class.java)
    
    @Test
    fun testButtonClick() {
        // Check if button exists
        onView(withId(R.id.button))
            .check(matches(isDisplayed()))
        
        // Click button
        onView(withId(R.id.button))
            .perform(click())
        
        // Check result
        onView(withId(R.id.textView))
            .check(matches(withText("Button clicked!")))
    }
    
    @Test
    fun testRecyclerViewInteraction() {
        // Check RecyclerView is displayed
        onView(withId(R.id.recyclerView))
            .check(matches(isDisplayed()))
        
        // Click on first item
        onView(withId(R.id.recyclerView))
            .perform(RecyclerViewActions.actionOnItemAtPosition<RecyclerView.ViewHolder>(0, click()))
        
        // Verify navigation or state change
        onView(withText("Item clicked"))
            .check(matches(isDisplayed()))
    }
}

// Database testing
@RunWith(AndroidJUnit4::class)
class UserDaoTest {
    
    private lateinit var database: AppDatabase
    private lateinit var userDao: UserDao
    
    @Before
    fun createDb() {
        val context = ApplicationProvider.getApplicationContext<Context>()
        database = Room.inMemoryDatabaseBuilder(context, AppDatabase::class.java)
            .allowMainThreadQueries()
            .build()
        userDao = database.userDao()
    }
    
    @After
    fun closeDb() {
        database.close()
    }
    
    @Test
    fun insertAndGetUser() = runTest {
        // Given
        val user = User(1, "Test User", "test@example.com")
        
        // When
        userDao.insertUser(user)
        val retrievedUser = userDao.getUserById(1)
        
        // Then
        assertEquals(user, retrievedUser)
    }
}
```

## Deployment and Google Play Store

### Build Configuration

```kotlin
// build.gradle (Module: app)
android {
    compileSdk 34
    
    defaultConfig {
        applicationId "com.example.myapp"
        minSdk 21
        targetSdk 34
        versionCode 1
        versionName "1.0"
    }
    
    buildTypes {
        debug {
            isMinifyEnabled = false
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"
            buildConfigField("String", "API_URL", "\"https://api-dev.example.com\"")
        }
        
        release {
            isMinifyEnabled = true
            proguardFiles(getDefaultProguardFile("proguard-android-optimize.txt"), "proguard-rules.pro")
            buildConfigField("String", "API_URL", "\"https://api.example.com\"")
        }
    }
    
    // Build variants
    flavorDimensions += "version"
    productFlavors {
        create("free") {
            dimension = "version"
            applicationIdSuffix = ".free"
            versionNameSuffix = "-free"
        }
        
        create("paid") {
            dimension = "version"
            applicationIdSuffix = ".paid"
            versionNameSuffix = "-paid"
        }
    }
}
```

### App Signing

```kotlin
// Signing configuration in build.gradle
android {
    signingConfigs {
        create("release") {
            keyAlias = project.findProperty("MYAPP_RELEASE_KEY_ALIAS") as String? ?: ""
            keyPassword = project.findProperty("MYAPP_RELEASE_KEY_PASSWORD") as String? ?: ""
            storeFile = file(project.findProperty("MYAPP_RELEASE_STORE_FILE") as String? ?: "")
            storePassword = project.findProperty("MYAPP_RELEASE_STORE_PASSWORD") as String? ?: ""
        }
    }
    
    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
        }
    }
}

// gradle.properties
MYAPP_RELEASE_KEY_ALIAS=mykey
MYAPP_RELEASE_KEY_PASSWORD=mypassword
MYAPP_RELEASE_STORE_FILE=../keystore/myapp.jks
MYAPP_RELEASE_STORE_PASSWORD=mystorepassword
```

### Build Commands

```bash
# Build debug APK
./gradlew assembleDebug

# Build release APK
./gradlew assembleRelease

# Build App Bundle (recommended for Play Store)
./gradlew bundleRelease

# Install on device
./gradlew installDebug
./gradlew installRelease

# Generate signed APK via Android Studio
# Build -> Generate Signed Bundle / APK
```

## Performance Optimization

### Memory Management

```kotlin
// Use weak references for listeners
class MainActivity : AppCompatActivity() {
    
    private var networkCallback: NetworkCallback? = null
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        networkCallback = object : NetworkCallback {
            override fun onDataReceived(data: String) {
                // Handle data
            }
        }
        
        NetworkManager.addListener(WeakReference(networkCallback))
    }
    
    override fun onDestroy() {
        super.onDestroy()
        networkCallback = null
    }
}

// Use object pools for expensive objects
class BitmapPool {
    private val pool = mutableListOf<Bitmap>()
    
    fun getBitmap(width: Int, height: Int): Bitmap {
        return pool.firstOrNull { 
            it.width == width && it.height == height && !it.isRecycled 
        }?.also { pool.remove(it) } 
            ?: Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888)
    }
    
    fun returnBitmap(bitmap: Bitmap) {
        if (!bitmap.isRecycled && pool.size < MAX_POOL_SIZE) {
            pool.add(bitmap)
        }
    }
    
    companion object {
        private const val MAX_POOL_SIZE = 10
    }
}

// Efficient RecyclerView usage
class EfficientAdapter : RecyclerView.Adapter<ViewHolder>() {
    
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val item = items[position]
        
        // Use Glide for efficient image loading
        Glide.with(holder.itemView.context)
            .load(item.imageUrl)
            .placeholder(R.drawable.placeholder)
            .into(holder.imageView)
        
        // Avoid expensive operations in bind
        holder.textView.text = item.precomputedText
    }
    
    override fun onViewRecycled(holder: ViewHolder) {
        super.onViewRecycled(holder)
        // Clear resources
        Glide.with(holder.itemView.context).clear(holder.imageView)
    }
}
```

## Official Resources

- [Android Developer Documentation](https://developer.android.com/docs)
- [Android Jetpack](https://developer.android.com/jetpack)
- [Material Design](https://material.io/develop/android)
- [Android Architecture Guidance](https://developer.android.com/topic/architecture)
- [Android Studio User Guide](https://developer.android.com/studio/intro)
- [Google Play Console Help](https://support.google.com/googleplay/android-developer)
- [Android Developers Blog](https://android-developers.googleblog.com/)

## Community and Tools

- [Awesome Android](https://github.com/JStumpp/awesome-android)
- [Android Arsenal](https://android-arsenal.com/)
- [Android Weekly](https://androidweekly.net/)
- [Kotlin Official Documentation](https://kotlinlang.org/docs/)
- [Android Developers YouTube Channel](https://www.youtube.com/c/AndroidDevelopers)
- [r/androiddev](https://www.reddit.com/r/androiddev/)
- [Android Dev Summit](https://developer.android.com/events)