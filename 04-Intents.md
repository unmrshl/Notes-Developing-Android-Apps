# 4. Intents

## Intents
Intents let an app know that an action should take place. Intents are a little more complicated than just passing in a class name. Think of them as the means of communicating between activities.

```xml
<activity android:name="com.example.android.explicitintent.MainActivity">
   *****
   <intent-filter>
       <action android:name="android.intent.action.MAIN"/>

       <category android:name="android.intent.category.LAUNCHER"/>
   </intent-filter>
   *****
</activity>
```

The intent-filter item above tells the application to open the MainActivity when the app launches.

## Creating an explicit intent
Intents are like envelopes, packaging bits of information between the start activity and the end activity (and the reverse, if you want).

Since we know the destination, we can use the explicit constructor Intent(Context packageContext, Class<?> clazz).
```java
Context context = MainActivity.this;
Class clazz = ChildActivity.class;
startActivity(new Intent(context, clazz));
```

## Passing data between activities
In the main activity, within some action, create an intent and add some data to that intent as an Extra. Then, in the child activity, grab the source intent with getIntent() and retrieve that data.

```java
public class MainActivity extends AppCompatActivity {
    ...
    mDoSomethingCoolButton.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            Intent startChildActivityIntent = new Intent(MainActivity.this, ChildActivity.class)
                .putExtra(Intent.EXTRA_TEXT, "Some text to pass.");
            startActivity(startChildActivityIntent);
        }
    });
}

public class ChildActivity extends AppCompatActivity {
   private TextView mDisplayText;
   @Override
   protected void onCreate(Bundle savedInstanceState) {
       mDisplayText = (TextView) findViewById(R.id.tv_display);
       Intent sourceIntent = getIntent();
       if (sourceIntent.hasExtra(Intent.EXTRA_TEXT)) {
           mDisplayText.setText(sourceIntent.getStringExtra(Intent.EXTRA_TEXT));
       }
   }
}
```

## Implicit intents

See https://developers.google.com/maps/documentation/urls/android-intents

* How to open a web page

Convert the desired URL into a Uri. Create an ACTION_VIEW intent, and check to see if the user’s device has an app that can handle that intent. If they can, run startActivity() with that intent.
```java
public void onClickOpenWebpageButton(View v) {
   final String url = "https://www.udacity.com";
   openWebPage(url);
}

public void openWebPage(String url) {
   Uri uri = Uri.parse(url);
   Intent viewWebPageIntent = new Intent(Intent.ACTION_VIEW, uri);
   if (viewWebPageIntent.resolveActivity(getPackageManager()) != null) {
       startActivity(viewWebPageIntent);
   }
}
```

This illustrates using the PackageManager to check whether a user can handle a certain intent.

* How to open a map

Be sure to follow the expected schema for Geo Uri’s. Use Uri.Builder to create the Uri.

```java
public void onClickOpenAddressButton(View v) {
   showMap(new Uri.Builder().scheme("geo").path("0,0").query("2000 Broadway St, San Francisco CA").build());
}

public void showMap(Uri geoUri) {
   Intent showMapIntent = new Intent(Intent.ACTION_VIEW).setData(geoUri);
   if (showMapIntent.resolveActivity(getPackageManager()) != null) {
       startActivity(showMapIntent);
   }
}
```

* How to ‘share’ data

Specify content type, then create a “Chooser” from the type and the shareable data so the user can decide how to share their content.

```java
public void onClickShareTextButton(View v) {
   shareText("HeLlO wOrLd!");
}

public void shareText(String shareableText) {
   String mimeType = "text/plain";
   String chooserTitle = "How do you want to share?";

   Intent shareIntent = ShareCompat.IntentBuilder.from(this)
       .setChooserTitle(chooserTitle)
       .setType(mimeType)
       .setText(shareableText)
       .getIntent();
   startActivity(shareIntent);
}
```
