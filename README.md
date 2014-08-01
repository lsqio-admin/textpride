SwyftMedia
=========
------------------
### Android

#### Setting up Webview

1. Add permission to android app (in the manifest) to use internet:<br>
  ```
  <uses-permission android:name="android.permission.INTERNET" />
  ```

2. Create an activity (SwyftmediaWebViewActivtity.class)
  - Activity Title String (strings.xml) (can use cusomt title what app provider desires):
  ```
  <resources>
      ...
      <string name=“swyftmedia_webview_activity”>SwyftMedia Content</string>
  </resources>
  ```
  - Declare it in the manifest
  ```
  <application ... >
    ...
    <activity
        android:name= {package}.”SwyftmediaWebViewActivtity”
        android:label="@string/swyftmedia_webview_activity"
            android:parentActivityName=“{parent activity name}” >
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="{package name}.{activity name}" />
    </activity>
  </application>
  ```

3. Create a layout (named: “swyftmedia_webview_activty”);
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
  setContentView(R.layout.swyftmedia_webview_activty);
  ```

5. Get the Url that will be provided by the intent object when activity is started (below):
  ```
  Intent intent = getIntent();
  String swyftmediaprideUrl = intent.getStringExtra({ParentMainActivity}.SWYFTMEDIA_URL);
  //ex: "ads.swyftmedia.com/ad/:appName/:contentId/[:userid]?callback=true"
  String contentId = intent.getStringExtra({ParentMainActivity}.CONTENTID);
  String userId = intent.getStringExtra({ParentMainActivity}.USERID);
  ```

6. Get the WebView in the activity by id
  ```
  WebView swyftmediaWebView = (WebView) findViewById(R.id.webview);
  ```
  - Setup the web view:
  ```
  WebSettings webSettings = swyftmediaWebView.getSettings();
  webSettings.setJavaScriptEnabled(true);

  ```
  - Setup page redirect for callback (** this is where app provider content unlock function goes):
    - this is where the page redirect is checked to see if the URL tells the app to unlock the current
    	- if so run app-providers content unlock function
    	- stop further page loading
    	- close webivew if required
    	- else wait for user to close webview
  ```
  swyftmediaWebView.setWebViewClient(new WebViewClient() {  
    @Override  
    public void onPageStarted(WebView view, String url, Bitmap favicon){  
      if (url.startsWith(“swyftmediaUnlockContent/time=number/complete=boolean")) {  
        	view.stopLoading();  
  	// in app sticker unlock function
  	// close web view
      }  
    }  
  }  
  ```

7. load the Url (with params for post), some params taken from intent (above):
  ```
  String url = swyftmediaUrl;
  String postData = “appName={app providers name}&contentId=contentId&userId=userId&callback={true/false}";
  webview.postUrl(url,EncodingUtils.getBytes(postData, “BASE64”));
  ```
===========

#### Setting up Content to Unlock Screen
1. Generate Stickers (However the app provider chooses);

2. Create fields for intent to send parameters to the new activity
  ```
  public static String SWYFTMEDIA_URL = “{package name}.SWYFTMEDIAURL”;
  public static String CONTENT_ID = “{package name}.CONTENTID”;
  public static String USER_ID = “{package name}.USERID”;

  ```
3. Attach swyftmedia Url to stickers
4. If stickers have swyftmedia Url, add/show unlock button as seem fit
  - button must run a method, ex: `swyftmediaViewContent(View view);`
    - therefor onclick of unlock button, run swyftmediaViewContent(View view); 
  - method definition:
  ```
  swyftmediaViewContent(View view){
    String swyftmediaUrl = {url from client for webview, attached to the sticker} from intent;
    String stickerId = { the id of the sticker clicked on} from intent
    String userId = {id of the user in the app} from intent
    int code = 404;
    HttpURLConnection connection = null;
    try{         
        URL myurl = new URL(swyftmediaUrl);        
        connection = (HttpURLConnection) myurl.openConnection();     
        connection.setRequestMethod("HEAD");         
        code = connection.getResponseCode();        
        System.out.println("" + code); 
    } catch {
    //Handle invalid URL
    }
    
    // if code = 200 (not 404 or 500)
    Intent viewSwyftMediaContent = new Intent(this, swyftmediaWebViewActivtity.class);
    viewSwyftMediaContent.putExtra(SYFTMEDIA_URL, swyftmediaUrl);
    viewSwyftMediaContent.putExtra(CONTENT_ID, stickerId);
    viewSwyftMediaContent.putExtra(USER_ID, userId);
    startActivity(viewSwyftMediaContent);
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
  
5. On sticker (with swyftmediaUrl) click Load uiwebview with post data 
  ```
  NSURL *url = [NSURL URLWithString: @swyftmediaUrl];
  NSString *body = [NSString stringWithFormat: @"appName=%@&contentId=%@&userId=%@&callback=%@",
  	@"{app providers name}",@contentId, @userId, @"{true|false}";

  NSMutableURLRequest *request = [[NSMutableURLRequest alloc]initWithURL: url];

  [request setHTTPMethod: @“POST"];

  [request setHTTPBody: [body dataUsingEncoding: NSUTF8StringEncoding]];

  [webView loadRequest: request];
  ```
=============

##### Alternatively 
- To check if content is unlocked, you can watch for a value change in function unlock or hijack it
  - function = swyftmedia.unlock = false -> true;
  - the function will call automaticly when the the value changes
  - allowing you to overwrite the function and making logic based on your needs.

- Or html5 call a function on that using 
  ```
  window.swyftmedia.unlock = function(){
   // js unlock functions by app
  };
  ````
  
-----------------
=================
-----------------

     | End User | Client | Swyftmedia | App Partners
-----| -------- | ------ | ---------- | ------------
Client Signup | | | SwyftMedia uses the admin panel to setup the Client | 
Creating a Campaign | | The client creates  a new story or campaign. | Swyftmedia uses the information/settings entered by the client and sets up a unique URL | 
The Stickers | | The client provides the generated unique URL to the app partner for the stickers (or other in app content) | | app partners integrate the stickers with a clickable link
The Call | The end user wants to download the sticker, if told to unlock, they must complete the campaign event. User clicks to continue, buy or cancel | | the generated url is used by SwyftMedia to confirm the campaign and link is active. the confirmation returns event details for the in-app browser experience to the app | the app hits the link information to confirm link is active and use the returning info to start the browser and generate the content
The Callback | The User is provided with share, unlock sticker, close buttons. | | - share allows them to share the link to the campaign (in and out of app) <br> - unlock sticker provides a callback function back in the app to unlock the sticker <br> - close just cancels the event, nothing is delegated except closing the browser. event must be completed again | - app must allow in-app link sharing to other users <br> - app must integrate callback javascript interface function to complete the event and unlock the feature <br> - callback to close the browser
