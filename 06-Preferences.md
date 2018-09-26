# 6. Preferences

With preferences, we can easily persist data across phases of the lifecycle. Preferences manage things like the Settings of a traditional app.

## Data persistence in Android
There are five typical ways to persist data in Android.
1. **onSavedInstanceState** - Save key-value pairs to persist the state of one of your views. Usually used to save the state during phone rotation, or when the phone destroys the app for memory constraints. This is a temporary place to store the state, and should only be used while a user is still using the app. If you want to save data across app restarts, and phone restarts, you need to save the data to an actual FS.

2. **SharedPreferences** - Specify a file, and save specific key-value pairs to that file. The keys are always strings, and the values are always primitives. SharedPreferences is usually used if you need to save a single text or numerical value about the user. E.g., a game app might save the player's name in SavedPreferences. Sometimes, however, you have more intricate data that can't be saved in key-value pairs.

3. **(Local) SQLite Database** - Stores organized, more complicated data than just text/numbers/booleans. Can access data across app restarts and phone restarts.

4. **Internal Storage** - Android allows you to save larger files to internal storage, or to external storage (like an SD card). Can store multimedia, or large amounts of text.

5. **Cloud Storage**

## Settings = SharedPreferences + PreferenceFragment

There is also a UI class called PreferenceFragment that helps for creating a Settings screen in the user interface.

**Fragament:** A fragment is a class that represents a modular and reusable piece of an Activity

The PreferenceFragment class is specifically designed to display Preferences. PreferenceFragments populate themselves with views defined in XML. When users modify values in the PreferenceFragment, it modifies values for keys in the Androids's SharedPreferences.

## Create a SettingsActivity class

Create a new Activity for your settings as per usual. Here are some Settings-specific things to add to the android manifest:
```xml
  <application
      android:allowBackup="false"
      android:icon="@mipmap/ic_launcher"
      android:label="@string/app_name"
      android:supportsRtl="true"
      android:theme="@style/AppTheme"
      tools:ignore="GoogleAppIndexingWarning">
    <activity android:name=".VisualizerActivity"
        android:launchMode="singleTop">
      <intent-filter>
        <action android:name="android.intent.action.MAIN"/>

        <category android:name="android.intent.category.LAUNCHER"/>
      </intent-filter>
    </activity>
    <activity android:name=".SettingsActivity"
        android:label="Settings"
        android:parentActivityName=".VisualizerActivity">
        <meta-data
          android:name="android.support.PARENT_ACTIVITY"
          android:value=".VisualizerActivity"
            />
    </activity>
  </application>
```
* Change the main activity's launch mode to "SingleTop", which will prevent it from reloading after Settings is closed
* Add the main activity as a parentActivity of the SettingsActivity

And some things to add to SettingsActivity.java:
```java
public class SettingsActivity extends AppCompatActivity {

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_settings);
    
    ActionBar actionBar = this.getSupportActionBar();
    if (actionBar != null) {
      actionBar.setDisplayHomeAsUpEnabled(true);
    }
  }

  @Override
  public boolean onOptionsItemSelected(MenuItem item) {
    int itemId = item.getItemId();
    switch (itemId) {
      case R.id.home:
        NavUtils.navigateUpFromSameTask(this);
        return true;
      default:
        return super.onOptionsItemSelected(item);
    }
  }
}
```
* Replace the back button in SettingsActivity to appear as an up button
* Override the back button's click method to take you back to the main Activity, instead of closing the app
