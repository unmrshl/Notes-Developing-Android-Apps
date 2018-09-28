# 8. Android Architecture Components

ContentProviders are great, but sometimes we don't need the advantages it provides. In those cases, Room can come in handly. Room is an ORM, Object-relational Mapping library, which creates a database in code and maps database objects to Java objects. 

## Room 

Some advantages that Room provides:
* Less boilerplate
* SQL validation at compile-time
* Built to work with LiveData and RxJava for data observation

Limitations of Room:
* Room only allows for a single constructor. You can use the `@Ignore` annotation to tell Room to ignore methods or parameters

Room dependencies:
```xml
dependencies {
    ...
    implementation "android.arch.persistence.room:runtime:1.0.0"
    annotationProcessor "android.arch.persistence.room:compiler:1.0.0"
}
```
## Room Components 

Room has three main components:
1. **Entity** - The simple addition of the Entity annotation can transform a POJO into a database table. With `@Entity(tableName = task)`, we can give a table a different name than the POJO class name. Room also requires a primary key (`@PrimaryKey`(autoGenerate = true) annotation above a single parameter)

The following is an example of a valid Entity class, and its corresponding "task" data table.

```java
@Entity(tableName = "task")
public class TaskEntry {
    @PrimaryKey(autoGenerate = true)
    private int id
    private String description;
    @ColumnInfo(name = "updated_at")
    private Date updatedAt;
    
    @Ignore
    public TaskEntry(String description, int priority, Date updatedAt) {
        this.description = description;
        this.priority = priority;
        this.updatedAt = updatedAt;
    }
    
    public TaskEntry(int id, String description, int priority, Date updatedAt) {
        this.id = id;
        this.description = description;
        this.priority = priority;
        this.updatedAt = updatedAt;
    }
    
    // ... Getters and Setters
}
```

| id | description     | priority | updated_at|
|----|-----------------|----------|-----------|
| 1  | sample descript | 1        | today     |
| 2  | sample descript | 2        | yesterday |
| 3  | sample descript | 3        | tuesday   |


2. **Dao** - A Dao is an interface with the `@Dao` annotation. The interface has a method for the four CRUD methods (@Query, @Insert, @Update, @Delete annotations).

The following is an example of a valid Data Access Object class.

```java
@Dao
public interface TaskDao {
    @Query("SELECT * FROM task ORDER BY priority")
    List<TaskEntry> loadAllTasks();
    
    @Query("SELECT * FROM task WHERE id = :id")
    TaskEntry loadTaskById(int id);
    
    @Insert
    void insertTask(TaskEntry taskEntry);
    
    @Update(onConflict = OnConflictStrategy.REPLACE)
    void updateTask(TaskEntry taskEntry);
    
    @Delete
    void deleteTask(TaskEntry taskEntry);
}
```


3. **Database** - A Singleton class with the `@Database` annotation that stores table Entities that Dao's can interact with.

The following is an example of a valid Database class.

```java
@Database(entities = {TaskEntry.class}, version = 1, exportSchema = false)
@TypeConverters(DateConverter.class)
public abstract class AppDatabase extends RoomDatabase {
    private static final Object LOCK = new Object();
    private static final String DB_NAME = "todolist";
    private static AppDatabase sInstance;
    
    public static AppDatabase getInstance(Context context) {
        if (sInstance == null) {
            synchronized (LOCK) {
                sInstance = Room.databaseBuilder(context.getApplicationContext(),
                        AppDatabase.class, AppDatabase.DB_NAME)
                        .build();
            }
        }
        return sInstance;
    }
}
```

If your custom Entity has a field with a non-primitive data type, (like Date in TaskEntry above), Room needs a Type converter to map the custom data type to one of the following data types: NULL, INTEGER, REAL, TEXT, BLOB. 

The following is an example of a type converter:

```java
public class DateConverter {
    @TypeConverter
    public static Date toDate(Long timestamp) {
        return timestamp == null ? null : new Date(timestamp);
    }
    
    @TypeConverter
    public static Long toTimestamp(Date date) {
        return date == null ? null : date.getTime();
    }
}
```
As with other kinds of loads, Room database loads on the main thread can lock and kill the UI. So, our database methods must be run on a secondary thread. 

## Executors

**Executors** are useful for Room DB operations, because, were we to just use Runnables, we would need to spawn a new thread every time we wanted to run a new DB operation. Executors allow us to create a set number of threads, and then continually dispatch Runnable tasks to that thread as we see fit. Executors, like AsyncTask and Runnables, are a way of performing asynchronous operations outside of the UI thread.

As a rule, if you need _simplicity_, use AsyncTask, but if you need **speed**, use traditional Java threads (Runnable).

One implementation of this "AppExecutors" concept (not the _best_ way to handle prespecified multithreading operations, but a workable one) is as follows:

```java
/**
 * Global executor pools for the whole application.
 * <p>
 * Grouping tasks like this avoids the effects of task starvation (e.g. disk reads don't wait behind
 * webservice requests).
 */
public class AppExecutors {

    // For Singleton instantiation
    private static final Object LOCK = new Object();
    private static AppExecutors sInstance;
    private final Executor diskIO;
    private final Executor mainThread;
    private final Executor networkIO;

    private AppExecutors(Executor diskIO, Executor networkIO, Executor mainThread) {
        this.diskIO = diskIO;
        this.networkIO = networkIO;
        this.mainThread = mainThread;
    }

    public static AppExecutors getInstance() {
        if (sInstance == null) {
            synchronized (LOCK) {
                sInstance = new AppExecutors(Executors.newSingleThreadExecutor(),
                        Executors.newFixedThreadPool(3),
                        new MainThreadExecutor());
            }
        }
        return sInstance;
    }

    public Executor diskIO() {
        return diskIO;
    }

    public Executor mainThread() {
        return mainThread;
    }

    public Executor networkIO() {
        return networkIO;
    }

    private static class MainThreadExecutor implements Executor {
        private Handler mainThreadHandler = new Handler(Looper.getMainLooper());

        @Override
        public void execute(@NonNull Runnable command) {
            mainThreadHandler.post(command);
        }
    }
}
```

A single thread, like diskIO, can then be called on the main thread as such:

```java
AppExecutors.getInstance().diskIO().execute(new Runnable() {
    @Override
    public void run() {
        mDb.taskDao().insertTask(taskEntry);
        finish();
        }
    });
```

## LiveData

An Observer pattern implementation of a data holder class. LiveData sits between the database and the UI, monitoring changes in the database, and notifying the MainActivity when a change should prompt a re-query. The LiveData object is the subject, and the activities are the observers.

Dependencies:
```xml
dependencies {
    implementation "android.arch.lifecycle:extensions:1.1.0"
    annotationProcessor "android.arch.lifecycle:compiler:1.1.0"
}
```

Dao interface:
```java
@Dao
public interface TaskDao {
    ...
    @Query("SELECT * FROM task WHERE id = :id")
    LiveData<TaskEntry> loadTaskById(int id);
}
```

To incorporate LiveData into a Room database, and be notified whenever the underlying data changes, we can replace the following snippet:
```java
AppExecutors.getInstance().diskIO().execute(new Runnable() {
    @Override
    public void run() {
        final TaskEntry task = mDb.taskDao().loadTaskById(mTaskId);
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                populateUI(task);
            }
        });
    }
});
```

with the following code:
```java
final LiveData<TaskEntry> task = mDb.taskDao().loadTaskById(mTaskId);
task.observe(this, new Observer<TaskEntry>() {
    @Override
    public void onChanged(@Nullable TaskEntry taskEntry) {
        populateUI(taskEntry);
    }
});
```

LiveData updates do not occur on the main thread, so we don't need to manually dispatch operations that return LiveData to a secondary thread. Additionally, LiveData's onChanged() method occurs back on the main thread, so we don't need to use the runOnUiThread() method either.

## ViewModel

Thanks to LiveData, we can be notified of changes to our data without having to requery the Database incessantly. However, we will still requery the database every time the device is rotated. ViewModel asks as a cache that persists across configuration changes, since onSavedInstanceState() is only meant for saving small amounts of data across configuration changes. ViewModel lasts from when the activity is created until it is finished.

Also, traditionally, if we made Async calls before the Activity was destroyed, we would have a memory leak. If we make those calls in ViewModel, we don't have to worry abou those leaks.

Implementation of ViewModel:

```java
public class MainViewModel extends AndroidViewModel {
  LiveData<List<TaskEntry>> tasks;

  public MainViewModel(@NonNull Application application) {
    super(application);
    tasks = AppDatabase.getInstance(this.getApplication()).taskDao().loadAllTasks();
  }

  public LiveData<List<TaskEntry>> getTasks() {
    return tasks;
  }
}
```

And to augment the LiveData code snippet above:

```java
MainViewModel viewModel = ViewModelProviders.of(this).get(MainViewModel.class);
viewModel.getTasks().observe(this, new Observer<List<TaskEntry>>() {
    @Override
    public void onChanged(@Nullable List<TaskEntry> taskEntries) {
        mAdapter.setTasks(taskEntries);
    }
});
```

Additionally, to pass parameters to a ViewModel, you must make use a ViewModelFactory and override its create() function to initialize a ViewModel with your parameters in its constructor.

```java
public class AddTaskViewModelFactory extends ViewModelProvider.NewInstanceFactory {
    private AppDatabase appDatabase;
    private int taskId;

    public AddTaskViewModelFactory(AppDatabase appDatabase, int taskId) {
      this.appDatabase = appDatabase;
      this.taskId = taskId;
    }

    @Override
    public <T extends ViewModel> T create(Class<T> modelClass) {
        return (T) new AddTaskViewModel(appDatabase, taskId);
    }
}

public class AddTaskViewModel extends ViewModel {
    LiveData<TaskEntry> task;

    AddTaskViewModel(AppDatabase appDatabase, int taskId) {
        task = appDatabase.taskDao().loadTaskById(taskId);
    }

    LiveData<TaskEntry> getTask() {
        return task;
    }
}
```

Finally, make use of the ViewModelFactory as follows:

```java
AddTaskViewModelFactory factory = new AddTaskViewModelFactory(mDb, mTaskId);
final AddTaskViewModel viewModel = ViewModelProviders.of(this, factory).get(AddTaskViewModel.class);
viewModel.getTask().observe(this, new Observer<TaskEntry>() {
    @Override
    public void onChanged(@Nullable TaskEntry taskEntry) {
        viewModel.getTask().removeObserver(this);
        populateUI(taskEntry);
    }
});
```

**_Another_** benefit of LiveData: It is a **_Lifecycle aware component_**. This means LiveData is able to know the state of its associated component, and it queues updates for its observers, waiting until they are active. Plus, if they are destroyed, they will be unsubscribed, and the pending updates squashed, so we don't need to worry about memory leaks.

LifeCycle and LiveData are so well intertwined because Android Architecture Components were all designed to work hand in hand.
