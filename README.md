The Navigation Drawer pattern is ubiquitous on Android devices:


![nav_example](readme_images/nav_example.png)

This pattern is easily implemented with the [Design Support Library](https://android-developers.googleblog.com/2015/05/android-design-support-library.html). The best walkthrough on how to build this [tutorial by CodePath](http://guides.codepath.com/android/fragment-navigation-drawer).

On larger devices though, hiding and showing the navigation drawer is unneccessary. With plenty of screen realestate, the menu should just stay open all the time. The [Material Design specs](https://material.io/guidelines/patterns/navigation-drawer.html#navigation-drawer-behavior) even recommend this behavior. I set out to see if I can use Fragments to make this re-use easy.

**TODO insert goal screenshots**

If you're unfamiliar with Navigation Drawers, I suggest you run through the CodePath tutorial first. It'll take you maybe 30 minutes and you'll have a better understanding of what we're going to do here.

Let's get started codeing the building blocks of our app, the re-usable pieces, the Fragments and their layouts. We are going to keep things real simple so the UI is a bit ugly. Let's fist create the layouts for our master and detail fragments:

`fragment_master.xml`:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:background="@color/blue"
              android:orientation="vertical">

    <!-- Replace this simple list with recyclerview or however you
        want to generate your master list -->

    <TextView
        android:id="@+id/master_item_1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp"
        android:text="master list item 1"/>

    <TextView
        android:id="@+id/master_item_2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp"
        android:text="master list item 2"/>

    <TextView
        android:id="@+id/master_item_3"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp"
        android:text="master list item 3"/>

</LinearLayout>
```

We'll set some background color so that we can easily distinguish the boundaries of our fragments. In this example I've got a simple list with three items. You can feel free to use a `RecyclerView` or `ScrollView` here depending on your needs.

Similarly for the detail screen:

`fragment_detail.xml`:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">

    <!-- Replace this simple list with recyclerview or however you
        want to generate your detail list -->

    <TextView
        android:id="@+id/detail_item_1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:text="detail list item 1"/>

    <TextView
        android:id="@+id/detail_item_2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:text="detail list item 2"/>

    <TextView
        android:id="@+id/detail_item_3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:text="detail list item 3"/>

</LinearLayout>
```

Again replace the detail list with whatever you want in the detail screen.

Now let's wire these layouts up to our Fragments:

`MasterFragment.java`:

```
public class MasterFragment extends Fragment {

    public static MasterFragment newInstance() {
        return new MasterFragment();
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, 
                             @Nullable ViewGroup container, 
                             @Nullable Bundle savedInstanceState) {
        
        View view = inflater.inflate(R.layout.fragment_master, container, false);

        TextView textView1 = (TextView) view.findViewById(R.id.master_item_1);
        textView1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // TODO
            }
        });

        TextView textView2 = (TextView) view.findViewById(R.id.master_item_2);
        textView2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // TODO
            }
        });

        TextView textView3 = (TextView) view.findViewById(R.id.master_item_3);
        textView3.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // TODO
            }
        });

        return view;
    }
}

```

`DetailFragment.java`:

```
public class DetailFragment extends Fragment {

    private TextView textView1;
    private TextView textView2;
    private TextView textView3;
    private int nonSelectedColor;
    private int selectedColor;

    public static DetailFragment newInstance() {
        return new DetailFragment();
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
                             
        View view = inflater.inflate(R.layout.fragment_detail, container, false);

        textView1 = (TextView) view.findViewById(R.id.detail_item_1);
        textView2 = (TextView) view.findViewById(R.id.detail_item_2);
        textView3 = (TextView) view.findViewById(R.id.detail_item_3);

        selectedColor = ContextCompat.getColor(getContext(), R.color.red);
        nonSelectedColor = ContextCompat.getColor(getContext(), R.color.black);

        return view;
    }
}
```

For this simple example, we are going to update the *detail* text color based on the selected *master* item. This will just be a simple way to observe sending information from `MasterFragment` to `DetailFragment`.

Now we need to host these two fragments in our `Activity`, but first let's create the layout for this activity:

`activity_main.xml`:

```
<android.support.v4.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@color/white"
            android:fitsSystemWindows="true"
            android:minHeight="?attr/actionBarSize"/>

        <FrameLayout
            android:id="@+id/detail_fragment_container"
            android:layout_width="match_parent"
            android:layout_height="match_parent"/>

    </LinearLayout>

    <!-- The navigation drawer that comes from the left -->
    <!-- Note that `android:layout_gravity` needs to be set to 'start' -->
    <android.support.design.widget.NavigationView
        android:id="@+id/master_fragment_container"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:fitsSystemWindows="true">

    </android.support.design.widget.NavigationView>
    
</android.support.v4.widget.DrawerLayout>
```


There's a lot going on here. If you have used navigateion drawers before, or you read through the CodePath example, some of this should look familiar. We need to set a `DrawerLayout` as our root view. We then need add a `FrameLayout`  as the container to insert our **detail** fragment. Now here' the cool part. Instead of including our `fragment_master.xml` layout inside `NavigationView`, we'll actually use the `NavigationView` as another container, this time for our **master** fragment.

So now we can wire all this up inside our Activity:

`MainActivity.java`:

```
public class MainActivity extends AppCompatActivity {

    private static final String TAG_MASTER_FRAGMENT = "TAG_MASTER_FRAGMENT";
    private static final String TAG_DETAIL_FRAGMENT = "TAG_DETAIL_FRAGMENT";

    private DrawerLayout drawerLayout;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_main);

        // setup toolbar
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        // setup drawer view
        drawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        NavigationView navigationView = (NavigationView) findViewById(R.id.master_fragment_container);
        navigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                return true;
            }
        });

        // setup menu icon
        final ActionBar actionBar = getSupportActionBar();
        if (actionBar != null) {
            actionBar.setHomeAsUpIndicator(R.drawable.ic_menu_black_24dp);
            actionBar.setDisplayHomeAsUpEnabled(true);
        }

        // insert detail fragment into detail container
        DetailFragment detailFragment = DetailFragment.newInstance();
        FragmentManager fragmentManager = getSupportFragmentManager();
        fragmentManager.beginTransaction()
                .add(R.id.detail_fragment_container, detailFragment, TAG_DETAIL_FRAGMENT)
                .commit();

        // insert master fragment into master container (i.e. nav view)
        MasterFragment masterFragment = MasterFragment.newInstance();
        fragmentManager.beginTransaction()
                .add(R.id.master_fragment_container, masterFragment, TAG_MASTER_FRAGMENT)
                .commit();
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // The action bar home/up action should open or close the drawer.
        switch (item.getItemId()) {
            case android.R.id.home:
                drawerLayout.openDrawer(GravityCompat.START);
                return true;
        }

        return super.onOptionsItemSelected(item);
    }
}	
```

Again a lot of this comes from the CodePath example: 

* seting up the toolbar
* setting up the drawer view
* setting up the menu icon
* setting up `onOptionsItemSelected`

The important parts of using fragments is creating our master and detail fragments and using fragment transactions to insert them into their respective containers. We'll also add fragment tags so that we can easily find our fragments later.


**TODO add part about fragment callbacks and passing data to detail**


At this point we've basically reproduced the CodePath example but with Fragments. We don't really have anything to show for our extra work. So now is the time to re-use our master and detail fragments to make this look different on larger screens.

We are going to use resource qualifiers to supply a different version of `activity_main.xml` on screens with a smallest width larger than 600dp. This is a good initial guess at tablet size.

So create a resource directory `src/main/res/layout-sw600dp` and inside it create a new `activity_main.xml`

`layout-sw600dp/activity_main.xml`:

```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/white"
        android:minHeight="?attr/actionBarSize"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="horizontal">

        <FrameLayout
            android:id="@+id/master_fragment_container"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"/>

        <FrameLayout
            android:id="@+id/detail_fragment_container"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="4"/>

    </LinearLayout>
    
</LinearLayout>
```

This is a much simpler layout. Again we still have two containers for master and detail fragments. Note that the `id`s need to match what we had in `layout/activity_main.xml`. We also need the `Toolbar`. Lastly we need to decide the relateive spacing of the master/detail. I've quickly chosen a 1:4 split by using `layout_weight`. This is really up to you. You may decide to just wrap width of the master container and give all the remaining width to the detail. It's up to you.

Now we need to head back to our Activity and make sure it can handle this new layout. We need to make just a few changes

`MainActivity.java`:

<pre>
public class MainActivity extends AppCompatActivity implements MasterFragment.Callbacks {

    private static final String TAG_MASTER_FRAGMENT = "TAG_MASTER_FRAGMENT";
    private static final String TAG_DETAIL_FRAGMENT = "TAG_DETAIL_FRAGMENT";

    private DrawerLayout drawerLayout;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_main);

        // setup toolbar
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        drawerLayout = (DrawerLayout) findViewById(R.id.drawer_layout);
        <b>if (drawerLayout != null) {</b>
            // setup drawer view
            NavigationView navigationView = (NavigationView) findViewById(R.id.master_fragment_container);
            navigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
                @Override
                public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                    return true;
                }
            });

            // setup menu icon
            final ActionBar actionBar = getSupportActionBar();
            if (actionBar != null) {
                actionBar.setHomeAsUpIndicator(R.drawable.ic_menu_black_24dp);
                actionBar.setDisplayHomeAsUpEnabled(true);
            }
        <b>}</b>

        // insert detail fragment into detail container
        DetailFragment detailFragment = DetailFragment.newInstance();
        FragmentManager fragmentManager = getSupportFragmentManager();
        fragmentManager.beginTransaction()
                .add(R.id.detail_fragment_container, detailFragment, TAG_DETAIL_FRAGMENT)
                .commit();

        // insert master fragment into master container (i.e. nav view)
        MasterFragment masterFragment = MasterFragment.newInstance();
        fragmentManager.beginTransaction()
                .add(R.id.master_fragment_container, masterFragment, TAG_MASTER_FRAGMENT)
                .commit();
    }
    
    ...
    
    @Override
    public void onMasterItemClicked(int masterItemId) {
        DetailFragment detailFragment = (DetailFragment) getSupportFragmentManager()
                .findFragmentByTag(TAG_DETAIL_FRAGMENT);
        detailFragment.onMasterItemClicked(masterItemId);

        // Close the navigation drawer
        <b>if (drawerLayout != null) {</b>
            drawerLayout.closeDrawers();
        <b>}</b>
    }
}
</pre>

All we need to do is null check `drawerLayout` before setting up drawer view and menu icon. If Android decides to use our `layout/activity_main.xml`, then the view hierarchy will contain a view with id `drawer_layout`. If Android decides to use `layout-sw600dp/activity_main.xml`, then the view hierarchy will NOT contain a view with that id. In other words, we are able to use the presense of `drawerLayout` as an indication of whether we're on a large screen or not.

**TODO finish this up**


 