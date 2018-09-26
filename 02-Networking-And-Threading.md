# 2. Network & Threading

## URLs
How to build URLs for network requests.

```java
Uri builtUri = Uri.parse(BASE_URL).buildUpon()
    .appendQueryParameter(PARAM_QUERY, searchQuery)
    .appendQueryParameter(PARAM_SORT, sortBy)
    .build();
URL url = null;
try {
    Url = new URL(builtUri.toString());
} catch (MalformedURLException e) {
    e.printStackStrace();
}
```

## Reading HTTP Response
How to return the entire result from an HTTP response.

```java
public static String getResponseFromHttpUrl(URL url) throws IOException { 
    HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection(); 
    try { 
        InputStream in = urlConnection.getInputStream(); 
        Scanner scanner = new Scanner(in); 
        scanner.useDelimiter("\\A"); 
        boolean hasInput = scanner.hasNext(); 
        if (hasInput) { 
            return scanner.next(); 
        } else { 
            return null; 
        } 
    } finally { 
        urlConnection.disconnect(); 
    } 
}
```

## The Internet Permission
Android apps need to explicitly list which permissions they require from the user. It is best to request the minimum number of permissions needed to perform your function. In the AndroidManifest.xml, add the line:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```
## AsyncTask
Android throws an exception when you try to make network requests on the main (UI) thread. In order to guarantee 60 fps, operations on the main thread must take < 17ms. Network tasks must be run on a secondary execution thread. AsyncTask allows you to run a task on a background thread while updating your progress on the UI thread.

The AsyncTask constructor takes three types: Params (type sent to the task upon execution), Progress (type published to update progress during the background computation), and Result (type of the result of the background task). The following functions can be overridden within a custom AsyncTask class: doInBackground(), onProgressUpdate(), onPostExecute(), and onPreExecute(), publishProgress(). A task in submitted by calling execute();

Creating a custom AsyncTask:
```java
String[] urls = {"first_url"};
new CustomAsyncTask().execute(urls);
...
public class CustomAsyncTask extends AsyncTask<URL, int, String> {
    @Override // Happens on 2nd thread, can network but no UI updates
    protected String doInBackground(URL... urls) {
        return null;
    }

    @Override // Happens on UI thread, UI change but no networking
    protected int onProgressUpdate() {
        return null;
    }

    @Override // Happens on UI thread, UI change but no networking
    protected void onPostExecute(String executionResult) {
        return;
    }
}
```
