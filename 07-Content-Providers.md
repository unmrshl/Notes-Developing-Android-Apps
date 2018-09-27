# 7. Content Providers

A way to read from & write to a user's contacts, calendar, and storage. A **Content Provider** is a class that sits between an application and its data source and provides managed access. If you don't use a content provider, the data you use is isolated to your app alone. 

General steps for using a Content provider:
1. Get permission to use the content provider
2. Get the ContentResolver
3. Pick one of four basic actions on the data (query, instert, update, delete)
4. Identify the data you are reading or manipulating to create a URI
5. In the case of reading, display the info in the UI

## Content Provider Permissions

You can request permission from the user to access a Content provider as normal, by adding a line in the Android manifest.

```xml
<uses-permission android:name="com.example.udacity.droidtermsexample.TERMS_READ"/>
```

## Content Resolvers

A content resolver sits between an application and the many content providers it may want to access. Resolvers act as intermediaries between all content providers and all content requesters.

```java
Content Resolver resolver = getContentResolver();
```

Once gotten, a resolver has four CRUD methods you can use: query(), insert(), update(), delete(). A URI is passed into the method to locate the resource(s) in question. Content provider URI's take the form of: `content://com.example.contentauthority.identifier/resource`.

The Docs of the Content Provider you are trying to access will likely have instructions on what URI to use to access their data.

Database operations can take a pretty long time (like disk operations). These operations should be done off the main thread, using an AsyncTask or a Loader.
```java
public class MainActivity extends AppCompatActivity {
    private Cursor mData;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        new ContentResolverAsyncTask().execute();
    }

    public class ContentResolverAsyncTask extends AsyncTask<Void, Void, Cursor> {

        @Override
        protected Cursor doInBackground(Void... voids) {
            ContentResolver resolver = getContentResolver();
            return resolver.query(DroidTermsExampleContract.CONTENT_URI, null, null, null, null)
        }

        @Override
        protected void onPostExecute(Cursor cursor) {
            mData = cursor;
            super.onPostExecute(cursor);
        }
    }
}
```
### Working with Cursors

**Cursors** are iterators that provide read/write access to a ContentProvider's data source. Cursor data is tabular. 

A cursor has a position, which is the row it's currently pointing to. 
* `cursor.moveToNext()` - moves to the next row, returning true/false if the operation was possible/impossible.
* `cursor.moveToFirst()` - moves to first row
* `cursor.getColumnIndex(String heading)` - given a column heading, returns the index of the column. Headings can be found in the content provider's docs
* `cursor.get<Type>(int columnIndex)` - returns the value at the column index. Type can be found in the docs
* `cursor.getCount()` - returns the number of rows in cursor
* `cursor.close()` - always close the cursor to prevent memory leaks

## CursorLoader

Instead of running an AsyncTask to load from a ContentResolver, we should be using a Loader. The proper Loader to use here is a CursorLoader.

```java
public class MainActivity extends AppCompatActivity implements
        LoaderManager.LoaderCallbacks<Cursor> {

    public static final String[] MAIN_FORECAST_PROJECTION = {
            WeatherContract.WeatherEntry.COLUMN_DATE,
            WeatherContract.WeatherEntry.COLUMN_MAX_TEMP,
            WeatherContract.WeatherEntry.COLUMN_MIN_TEMP,
            WeatherContract.WeatherEntry.COLUMN_WEATHER_ID,
    };
    
    private static final int ID_FORECAST_LOADER = 44;

    private ForecastAdapter mForecastAdapter;
    private RecyclerView mRecyclerView;
    private int mPosition = RecyclerView.NO_POSITION;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_forecast);
        getSupportActionBar().setElevation(0f);
        FakeDataUtils.insertFakeData(this);
        mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview_forecast);
        LinearLayoutManager layoutManager =
                new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false);

        mRecyclerView.setLayoutManager(layoutManager);
        mRecyclerView.setHasFixedSize(true);

        mForecastAdapter = new ForecastAdapter(this, this);
        mRecyclerView.setAdapter(mForecastAdapter);

        getSupportLoaderManager().initLoader(ID_FORECAST_LOADER, null, this);
    }

    @Override
    public Loader<Cursor> onCreateLoader(int loaderId, Bundle bundle) {
        switch (loaderId) {
            case ID_FORECAST_LOADER:
                /* URI for all rows of weather data in our weather table */
                Uri forecastQueryUri = WeatherContract.WeatherEntry.CONTENT_URI;
                /* Sort order: Ascending by date */
                String sortOrder = WeatherContract.WeatherEntry.COLUMN_DATE + " ASC";
                /*
                 * A SELECTION in SQL declares which rows you'd like to return. In our case, we
                 * want all weather data from today onwards that is stored in our weather table.
                 * We created a handy method to do that in our WeatherEntry class.
                 */
                String selection = WeatherContract.WeatherEntry.getSqlSelectForTodayOnwards();

                return new CursorLoader(this,
                        forecastQueryUri,
                        MAIN_FORECAST_PROJECTION,
                        selection,
                        null,
                        sortOrder);

            default:
                throw new RuntimeException("Loader Not Implemented: " + loaderId);
        }
    }

    @Override
    public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
        mForecastAdapter.swapCursor(data);
        if (mPosition == RecyclerView.NO_POSITION) mPosition = 0;
        mRecyclerView.smoothScrollToPosition(mPosition);
        if (data.getCount() != 0) showWeatherDataView();
    }

    @Override
    public void onLoaderReset(Loader<Cursor> loader) {
        mForecastAdapter.swapCursor(null);
    }
}

```
