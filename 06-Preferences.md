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
