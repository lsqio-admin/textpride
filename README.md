textpride
=========
------------------
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
  ```
  setContentView(R.layout.textpride_webview_activty);
  ```

5. Get the Url that will be provided by the intent object when activity is started (below):
  ```
  Intent intent = getIntent();
  String textprideUrl = intent.getStringExtra({ParentMainActivity}.TEXTPRIDE_URL);
  //ex: "ads.swyftmedia.com/ad/:appName/:contentId/[:userid]?callback=true"
  String contentId = intent.getStringExtra({ParentMainActivity}.CONTENTID);
  String userId = intent.getStringExtra({ParentMainActivity}.USERID);
  ```

6. Get the WebView in the activity by id
  ```
  WebView textprideWebView = (WebView) findViewById(R.id.webview);
  ```
  - Setup the web view:
  ```
  WebSettings webSettings = textprideWebView.getSettings();
  webSettings.setJavaScriptEnabled(true);

  ```
  - Setup page redirect for callback (** this is where app provider content unlock function goes):
    - this is where the page redirect is checked to see if the URL tells the app to unlock the current
    	- if so run app-providers content unlock function
    	- stop further page loading
    	- close webivew if required
    	- else wait for user to close webview
  ```
  textprideWebView.setWebViewClient(new WebViewClient() {  
    @Override  
    public void onPageStarted(WebView view, String url, Bitmap favicon){  
      if (url.startsWith(“textprideUnlockContent/time=number/complete=boolean")) {  
        	view.stopLoading();  
  	// in app sticker unlock function
  	// close web view
      }  
    }  
  }  
  ```

7. load the Url (with params for post), some params taken from intent (above):
  ```
  String url = textprideUrl;
  String postData = “appName={app providers name}&contentId=contentId&userId=userId&callback={true/false}";
  webview.postUrl(url,EncodingUtils.getBytes(postData, “BASE64”));
  ```
===========

#### Setting up Content to Unlock Screen
1. Generate Stickers (However the app provider chooses);

2. Create fields for intent to send parameters to the new activity
  ```
  public static String TEXTPRIDE_URL = “{package name}.TEXTPRIDEURL”;
  public static String CONTENT_ID = “{package name}.CONTENTID”;
  public static String USER_ID = “{package name}.USERID”;

  ```
3. Attach textpride Url to stickers
4. If stickers have text pride Url, add/show unlock button as seem fit
  - button must run a method, ex: `textprideViewContent(View view);`
    - therefor onclick of unlock button, run textprideViewContent(View view); 
  - method definition:
  ```
  textprideViewContent(View view){
    String textprideUrl = {url from client for webview, attached to the sticker};
    String stickerId = { the id of the sticker clicked on}
    String userId = {id of the user in the app}
    int code = 404;
    HttpURLConnection connection = null;
    try{         
        URL myurl = new URL(textprideUrl);        
        connection = (HttpURLConnection) myurl.openConnection();     
        connection.setRequestMethod("HEAD");         
        code = connection.getResponseCode();        
        System.out.println("" + code); 
    } catch {
    //Handle invalid URL
    }
    
    // if code = 200 (not 404 or 500)
    Intent viewTextPrideContent = new Intent(this, textprideWebViewActivtity.class);
    viewTextPrideContent.putExtra(TEXTPRIDE_URL, textprideUrl);
    viewTextPrideContent.putExtra(CONTENT_ID, stickerId);
    viewTextPrideContent.putExtra(USER_ID, userId);
    startActivity(viewTextPrideContent);
  }
  ```
---------------

### Some iOS (Similar to Android)

1. Create UIWebView object in the window (not inside scrollview)

2. Enable caching and other required settings

3. Scale to fit page (scalePageToFit = true)
 
4. Overload Url Request, to check for url change for the callback (ex: to unlock content)
  ```
  - (BOOL)webView:(UIWebView *)webView 
      shouldStartLoadWithRequest:(NSURLRequest *)request 
      navigationType:(UIWebViewNavigationType)navigationType {

    NSLog(@"%@",[[request URL] query]);

    return NO;
  }

  ```
  
5. On sticker (with textprideUrl) click Load uiwebview with post data 
  ```
  NSURL *url = [NSURL URLWithString: @textprideUrl];
  NSString *body = [NSString stringWithFormat: @"appName=%@&contentId=%@&userId=%@&callback=%@",
  	@"{app providers name}",@contentId, @userId, @"{true|false}";

  NSMutableURLRequest *request = [[NSMutableURLRequest alloc]initWithURL: url];

  [request setHTTPMethod: @“POST"];

  [request setHTTPBody: [body dataUsingEncoding: NSUTF8StringEncoding]];

  [webView loadRequest: request];
  ```
=============

##### Alternatively 
- To check if content is unlocked, you can watch for a value change to see if event is unlocked
  - valueName = textprideContentUnlocked = false -> true;
  - Upon value change to true, make a request to the same textprideUrl
    - With params: `userId and confirm="true"`
    - look for response value: confirmed = `"true"|"false"`
    - if reponse is true, unlock content

-----------------
=================
-----------------

     | End User | Client | Text Pride | App Partners
-----| -------- | ------ | ---------- | ------------
Client Signup | | | Textpride uses the admin panel to setup the Client | 
Creating a Campaign | | The client creates  a new story or campaign. | Textpride uses the information/settings entered by the client and sets up a unique URL | 
The Stickers | | The client provides the generated unique URL to the app partner for the stickers (or other in app content) | | app partners integrate the stickers with a clickable link
The Call | The end user wants to download the sticker, if told to unlock, they must complete the campaign event. User clicks to continue, buy or cancel | | the generated url is used by text pride to confirm the campaign and link is active. the confirmation returns event details for the in-app browser experience to the app | the app hits the link information to confirm link is active and use the returning info to start the browser and generate the content
The Callback | The User is provided with share, unlock sticker, close buttons. | | - share allows them to share the link to the campaign (in and out of app) <br> - unlock sticker provides a callback function back in the app to unlock the sticker <br> - close just cancels the event, nothing is delegated except closing the browser. event must be completed again | - app must allow in-app link sharing to other users <br> - app must integrate callback javascript interface function to complete the event and unlock the feature <br> - callback to close the browser
