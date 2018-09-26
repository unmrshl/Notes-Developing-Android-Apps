# 1. Views, Layouts, etc

## R?
```java
mSearchResults = (TextView) findViewById(R.id.tv_github_search_results);
```

The R class is a way of accessing the res files in an Android Studio project.

## Layouts

Activities create views that show information to users and let users interact with the activity. An activity learns what views to create by reading an xml file (stored in res/layout). 

## Views

**Types of UI Views:** TextView, EditText, ImageView, Button, Chronometer, ProgressBar
The android.widget package contains a list of most of the UI View classes available.

**Types of Container Views:** LinearLayout, RelativeLayout, FrameLayout, ScrollView, ConstraintLayout
Container views are primarily responsible for containing groups of other views and determining their locations on screen.

Properties can be set on views similarly to css properties by adding props to views in xml
```xml
<TextView
  android:id="@+id/text_view_display" />
  ```
  
## Getting references to layouts in Java
{}.xml
```xml
<TextView
  android:id="@+id/text_view_display" />
```

{}Activity.java
```java
TextView mTextView = (TextView) findViewById(R.id.text_view_display);
```

## Logging
**Error levels:**
* v - Verbose
* d - Debug
* i - Info
* w - Warn
* e - Error
* wtf - WTF

Log statements accept two parameters: TAG and message. TAG is usually the class name.
```java
Log.v(TAG, "This is a verbose log statement.");
```

## Working with strings.xml

In an activity, just call 
```java
getString(R.id.string_id); 
```

In a layout file, make use of @string
```xml
<TextView text="@string/string_id" />
```

## Menus
Android has the concept of menus. Menus can be created in xml like any other resource by creating a menu/ folder in res/, and creating menu_name.xml files in the menu directory.

Sample menu:
```xml
<menu xmlns:android="///"
  xmlns:app="///">
    android:id="@id+/menu_id"
    android:orderInCategory="1"
    app:showAsAction="ifRoom"
    android:title="Menu Title"
</menu>
```
To create a menu in the activity, override onCreateOptionsMenu. To handle menu item selections, override onOptionsItemSelected.

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.menu_id, menu);
    return true;
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
    int itemId = item.getItemId();
    switch (itemId) {
        case R.id.menu_item_id:
            // do something
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}
```

## Toasts
Toasts are floating messages that appear at the bottom of the screen for a short or long period of time. The current activity remains visible and interactive.

```java
Context context = MainActivity.this;
String message = "Popup message.";
Toast.makeText(context, message, Toast.LENGTH_LONG).show();
```
