# 3. RecyclerView

Even though a certain part of the layout is viewable at any given time, the app loads additional views that may be seen later on. This can cause the app to run out of memory.\

## How RecyclerView works
Rather than creating each ListItem as a list scrolls, we keep them in a queue (or recycling bin) for use and reuse. Views that are taken off the queue are bound with new content, and then shown.

## Using RecyclerView
An Adapter takes items from a Data Source and sends them to the RecyclerView in a ViewHolder, which contains a reference to the root view object for the item, caching the view objects represented in the layout to make it less costly to populate the view with new data. LayoutManager tells the RecyclerView how to distribute those views on the screen.

## Create a RecyclerView:
```xml
<FrameLayout
   xmlns:android="http://schemas.android.com/apk/res/android"
   android:layout_width="match_parent"
   android:layout_height="match_parent">
   <android.support.v7.widget.RecyclerView
       android:id="@+id/rv_numbers"
       android:layout_width="match_parent"
       android:layout_height="match_parent"/>
</FrameLayout>
```

# Create a list item layout and ViewHolder
Create a custom view holder layout (number_list_item.xml) to represent a single list item.
```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
   android:layout_width="match_parent"
   android:layout_height="wrap_content"
   android:padding="16dp">
 <TextView
     android:id="@+id/tv_item_number"
     android:layout_width="wrap_content"
     android:layout_height="wrap_content"
     android:gravity="start"
     android:layout_gravity="center"
     android:fontFamily="monospace"
     android:textSize="42sp"/>
</FrameLayout>
```

Create a custom view holder class that corresponds to this layout.
```java
class NumberViewHolder extends RecyclerView.ViewHolder {
   private TextView mItemNumberTextView;

   public NumberViewHolder(View itemView) {
       super(itemView);
       mItemNumberTextView = (TextView) itemView.findViewById(R.id.tv_item_number);
   }

   public void bind(int listIndex) {
       mItemNumberTextView.setText(String.valueOf(listIndex));
   }
}
```

# Add a RecyclerView adapter
An adapter creates a ViewHolder object for each RecyclerView item, returns the number of items in the data source, binds each view item with data from the data source, and inflates each item view that will be displayed.

```java
public class GreenAdapter extends RecyclerView.Adapter<GreenAdapter.NumberViewHolder> {
   private int mNumberItems;

   public GreenAdapter(int numberOfItems) {
       mNumberItems = numberOfItems;
   }

   @Override
   public NumberViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

       return new NumberViewHolder(LayoutInflater.from(parent.getContext())
           .inflate(R.layout.number_list_item, parent, false));
   }

   @Override
   public void onBindViewHolder(NumberViewHolder holder, int position) {
       holder.bind(position);
   }

   @Override
   public int getItemCount() {
       return mNumberItems;
   }

   class NumberViewHolder extends RecyclerView.ViewHolder {
    // Defined above ...
   }
}
```

# Add a LayoutManager and connect everything else
LayoutManager determines how a collection of items is displayed, and when to recycle item views that are no longer visible. There are three general types of LayoutManager: LinearLayoutManager (scroll vertically or horizontally), GridLayoutManager (grid, can also scroll), StaggeredGridLayoutManager (for views with content of varying dimensions). You can also extend from LayoutManager and create your own.

```java
public class MainActivity extends AppCompatActivity {
   private static final int NUM_LIST_ITEMS = 100;
   private GreenAdapter mAdapter;
   private RecyclerView mNumbersList;

   @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);

       mNumbersList = (RecyclerView) findViewById(R.id.rv_numbers);
       LinearLayoutManager layoutManager = new LinearLayoutManager(this);
       mNumbersList.setLayoutManager(layoutManager);
       mNumbersList.setHasFixedSize(true);
      
       mAdapter = new GreenAdapter(NUM_LIST_ITEMS);
       mNumbersList.setAdapter(mAdapter);
   }
}
```

# Implementing click handling on custom ViewHolders
Create an interface with a clickable method. Implement View.OnClickListener in the ViewHolder. Pass a listener to the Adapter.

```java
class GreenAdapter extends RecyclerView.Adapter<GreenAdapter.NumberViewHolder> {
    ...
    private final ListItemClickListener mOnClickListener;

    public interface ListItemClickListener {
       void onListItemClick(int clickedItemIndex);
    }

    public GreenAdapter(int numberOfItems, ListItemClickListener listener) {
       mNumberItems = numberOfItems;
       viewHolderCount = 0;
       mOnClickListener = listener;
    }
    ...
    class NumberViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener {
        ...
        @Override
        public void onClick(View view) {
           mOnClickListener.onListItemClick(getAdapterPosition());
        }
    }
}
```


Lastly, implement your custom interface in some listener view and pass `this` to the Adapter.
```java
public class MainActivity extends AppCompatActivity implements GreenAdapter.ListItemClickListener {
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ...
        mAdapter = new GreenAdapter(NUM_LIST_ITEMS, this);
    }


    @Override
    public void onListItemClick(int clickedItemIndex) {
        ...
    }
}
```
