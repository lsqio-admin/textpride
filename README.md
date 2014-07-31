textpride
=========

------------
### Android

#### Setting up Webview

1. Add permission to android app (in the manifest) to use internet:<br>
  ```
  <uses-permission android:name="android.permission.INTERNET" />
  ```

2. Create an activity (TextprideWebViewActivtity.class)
  - Activity Title String (strings.xml) (can use cusomt title what app provider desires):
  ```
  <resources>
      ...
      <string name=“textpride_webview_activity”>Textpride Content</string>
  </resources>
  ```
  - Declare it in the manifest
  ```
  <application ... >
    ...
    <activity
        android:name= {package}.”TextprideWebViewActivtity”
        android:label="@string/textpride_webview_activity"
            android:parentActivityName=“{parent activity name}” >
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.MainActivity" />
    </activity>
  </application>
  ```

3. Create a layout (named: “textpride_webview_activty”);
  - Add the WebView to the activity layout
  ```
  <WebView  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/webview"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
  />
  ```

4. Add the layout (created above) to the onCreate() method of the activity:<br>
  setContentView(R.layout.textpride_webview_activty);

5. Get the Url that will be provided by the intent object when activity is started (below):


===========
