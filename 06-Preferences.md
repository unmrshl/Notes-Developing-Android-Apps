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

## Create a PreferenceFragment

Add the dependency `compile 'com.andoird.support:preference-v7:*.*.*'` to the build file.

Create an xml resource listing all preferences to include in your Preference menu. To create nested preferences (a second preference window within a preference window), place a PreferenceScreen tag within the root PreferenceScreen.

./res/xml/pref_visualizer.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android">
  <CheckBoxPreference
      android:defaultValue="true"
      android:key="show_bass"
      android:summaryOff="Hidden"
      android:summaryOn="Shows"
      android:title="Show Bass"/>
</PreferenceScreen>
```

Display the resource in a class that extends PreferenceFragmentCompat.

./java/SettingsFragment.java
```java
public class SettingsFragment extends PreferenceFragmentCompat {
  @Override
  public void onCreatePreferences(Bundle savedInstanceState, String rootKey) {
    addPreferencesFromResource(R.xml.pref_visualizer);
  }
}
```

Specify a preference theme in `styles.xml`.

./res/values/styles.xml
```xml
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- ... -->
        <item name="preferenceTheme">@style/PreferenceThemeOverlay</item>
    </style>

</resources>
```

Set the root layout of the pre-existing SettingsActivity to the preference fragment.

./res/layout/activity_settings.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<fragment
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:name="android.example.com.visualizerpreferences.AudioVisuals.SettingsFragment"
    android:id="@+id/activity_settings"
    xmlns:android="http://schemas.android.com/apk/res/android"/>

```

## Reading from & Writing to SharedPreferences

Above, we created a checkbox in the Preferences menu. When that checkbox is toggled, Android automatically assigns the value to the box's corresponding key (above: show_bass). In order to access these stored preferences, we need to read from SharedPreferences.

Traditional reads from SharedPreferences follow the paradigm (assuming we want a boolean value): prefs.getBoolean(key, defaultValue);

```java
SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
view.setShowBass(sharedPreferences.getBoolean("show_bass", true));
```

We can also manually write to SharedPreferences.

```java
SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
SharedPreferences.Editor editor = sharedPreferences.edit();
editor.putBoolean("show_bass", true);
editor.apply(); // Performs the update off the main thread
```

## Preference Change Listener

If you load your preferences from SharedPreferences in the onCreate() method of your main activity, you'll notice that things don't automatically change when you toggle your preferences. This can be fixed by implementing an object that listens to changes in the Preferences and automatically updates when they change.

```java
public class VisualizerActivity extends AppCompatActivity implements SharedPreferences.OnSharedPreferenceChangeListener {
    ...
    @Override
    public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String s) {
        if (s.equals(getString(R.string.pref_show_bass_key))) {
            mVisualizerView.setShowBass(sharedPreferences.getBoolean(s, getResources().getBoolean(R.bool.pref_show_bass_default)));
        }
    }
}
```

Then, register the listener in the Activity's onCreate() method, and de-register the listener in the onDestroy() method.

```java
@Override 
protected void onCreate(Bundle savedInstanceState) {
    ...
    SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
    sharedPreferences.registerOnSharedPreferenceChangeListener(this);
}

@Override
protected void onDestroy() {
    SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
    sharedPreferences.unregisterOnSharedPreferenceChangeListener(this);
    super.onDestroy();
}

```

## More Preference types

**CheckBoxPreference**
```xml
<CheckBoxPreference
      android:defaultValue="true"
      android:key="show_bass"
      android:summaryOff="Hidden"
      android:summaryOn="Shows"
      android:title="Show Bass"/>
```
**ListPreference** 
```xml
  <ListPreference 
      android:defaultValue="@string/pref_units_metric"
      android:entries="@array/pref_units_options"
      android:entryValues="@array/pref_units_values"
      android:key="@string/pref_units_key"
      android:title="@string/pref_units_label"/>
```

**EditTextPreference**
```xml
  <EditTextPreference 
      android:defaultValue="@string/pref_location_default"
      android:inputType="text"
      android:key="@string/pref_location_key"
      android:singleLine="true"
      android:title="@string/pref_location_label"/>
```

## Update Preference Summary

With some preference types, Preference Summary may not update automatically. It will need to be programmatically updated. To do so, implement OnSharedPreferenceChangeListener, iterate over the preferences and set the summaries for all preferences with summaries not automatically set, override onSharedPreferenceChanged() to handle changes as they happen, and register & unregister the change listener.

Example for a ListPreference:
```java
public class SettingsFragment extends PreferenceFragmentCompat implements SharedPreferences.OnSharedPreferenceChangeListener {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getPreferenceScreen().getSharedPreferences().registerOnSharedPreferenceChangeListener(this);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        getPreferenceScreen().getSharedPreferences().unregisterOnSharedPreferenceChangeListener(this);
    }

    @Override
    public void onCreatePreferences(Bundle bundle, String s) {
        addPreferencesFromResource(R.xml.pref_visualizer);
        PreferenceScreen screen = getPreferenceScreen();
        SharedPreferences sharedPreferences = screen.getSharedPreferences();
        for (int i = 0; i < screen.getPreferenceCount(); i++) {
            Preference p = screen.getPreference(i);
            if (!(p instanceof CheckBoxPreference)) {
                setPreferenceSummary(p, sharedPreferences.getString(p.getKey(), ""));
            }
        }
    }
    
    public void setPreferenceSummary(Preference pref, String value) {
        if (pref instanceof ListPreference) {
            int i = ((ListPreference) pref).findIndexOfValue(value);
            if (i >= 0) {
                String label = ((ListPreference) pref).getEntries()[i].toString();
                pref.setSummary(label);
            }
        }
    }

    @Override
    public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String s) {
        Preference preference = findPreference(s);
        if (preference != null) {
            if (!(preference instanceof CheckBoxPreference)) {
                setPreferenceSummary(preference, sharedPreferences.getString(preference.getKey(), ""));
            }
        }
    }
}
```

## Prevent illegal values in preferences

Occasionally, we will have a preference that accepts input from the user, but can only handle values within a specific range. We can use PreferenceChangeListener (**_different from OnSharedPreferenceChangeListener_**) to sieze unacceptable preference values before they are saved to SharedPreferences. The flow proceeds as follows:

1. User updates a preference
2. PreferenceChangeListener is triggered for that preference
3. The new value is saved to the SharedPreferences file
4. onSharedPreferencesChanged listeners are triggered

To use this, implement the Preference.OnPreferenceChangeListener, and override the onPreferenceChange() method, which returns true or false, depending on whether the preference should actually be saved.

```java
// TODO (1) Implement OnPreferenceChangeListener
public class SettingsFragment extends PreferenceFragmentCompat implements
        OnSharedPreferenceChangeListener, Preference.OnPreferenceChangeListener {

    @Override
    public void onCreatePreferences(Bundle bundle, String s) {
        for (int i = 0; i < count; i++) {
            Preference p = prefScreen.getPreference(i);
            ...
            if (p instanceof EditTextPreference) {
                p.setOnPreferenceChangeListener(this);
            }
        }
    }
    
    ...
    
    @Override
    public boolean onPreferenceChange(Preference preference, Object newValue) {
        Toast errorMessage = Toast.makeText(getContext(), "Please enter a number between 0.1 and 3.0.", Toast.LENGTH_LONG);

        try {
            float floatPreference = Float.parseFloat((String) newValue);
            if (floatPreference < 0.1 || floatPreference > 3.0) {
                throw new NumberFormatException();
            }
        } catch (NumberFormatException nfe) {
            errorMessage.show();
            return false;
        }

        return true;
    }
}
```
