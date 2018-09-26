# 5. Android Activity Lifecycle
Activities / apps can be killed at any time by the OS, so it’s important to know when and why an app will be killed. 

## Activity Lifecycle:
**Events:** onCreate -> onStart -> onResume -> onPause -> onStop -> onDestroy  (other: onRestart)
**States:** 	[Created] -> [Visible] ->   [Active] ->   [Paused] ->   [Stopped] ->  [Destroyed]

To make use of a limited amount of resources, you want to adjust your resource load with respect to this lifecycle.
Fun fact: Rotating the device causes the Activity to be destroyed and recreated.

## Persist data between lifecycles
Override the onSaveInstanceState() and make use of the outState object to persist data even when the activity is destroyed.

```java
@Override
public void onSaveInstanceState(Bundle outState) {
   super.onSaveInstanceState(outState);

   String lifecycleContents = mLifecycleDisplay.getText().toString();
   outState.putString(LIFECYCLE_CALLBACKS_TEXT_KEY, lifecycleContents);
}
```

In the onCreate method of the activity, check the saved state and load any persisted parameters.

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   ...
   if (savedInstanceState != null) {
       if (savedInstanceState.containsKey(QUERY_URL_PERSISTENCE_KEY)) {
           mUrlDisplayTextView.setText(savedInstanceState.getString(QUERY_URL_PERSISTENCE_KEY));
       }
      
       if (savedInstanceState.containsKey(QUERY_RESULT_PERSISTENCE_KEY)) {
           mUrlDisplayTextView.setText(savedInstanceState.getString(QUERY_RESULT_PERSISTENCE_KEY));
       }
   }
}
```

## Loaders
Loaders provide a framework to preform Async loading of data. They are registered in the LoaderManager by id, which allows them to life outside of the lifecycle of the Activity they “belong” to. This prevents duplicate loads from acting in parallel.
AsyncTaskLoader implements the same functionality as AsyncTask, but with a different lifecycle.
**Methods:** initLoader() -> onCreateLoader() -> loadInBackground() -> onLoadFinished()

First, create a Loader ID, then override the Loader callbacks, and initialize the loader with the LoaderManager.

```java
public class MainActivity extends AppCompatActivity implements LoaderManager.LoaderCallbacks<String> {

   private static final String SEARCH_QUERY_URL_EXTRA = "query";
   // 1. Create Loader ID
   private static final int GITHUB_SEARCH_LOADER = 73;
   ...
   @Override
   protected void onCreate(Bundle savedInstanceState) {
       ...
       // 3. Initialize Loader
       getSupportLoaderManager().initLoader(GITHUB_SEARCH_LOADER, null, this);
   }

   private void makeGithubSearchQuery() {
       ...
       Bundle queryBundle = new Bundle();
       queryBundle.putString(SEARCH_QUERY_URL_EXTRA, githubSearchUrl.toString());

       LoaderManager loaderManager = getSupportLoaderManager();
       Loader<String> githubSearchLoader = loaderManager.getLoader(GITHUB_SEARCH_LOADER);
       if (githubSearchLoader == null) {
           loaderManager.initLoader(GITHUB_SEARCH_LOADER, queryBundle, this);
       } else {
           loaderManager.restartLoader(GITHUB_SEARCH_LOADER, queryBundle, this);
       }
   }


   // 2. Override Loader callbacks
   @Override
   public Loader<String> onCreateLoader(int i, final Bundle bundle) {
       return new AsyncTaskLoader<String>(this) {
           @Override
           protected void onStartLoading() {
               if (bundle == null) {
                   return;
               }
               forceLoad();
           }

           @Override
           public String loadInBackground() {
               … // Perform load, return result
           }
       };
   }

   @Override
   public void onLoadFinished(Loader<String> loader, String s) {
       … // Perform task with result
   }

   @Override
   public void onLoaderReset(Loader<String> loader) {
       return;
   }
}
```
