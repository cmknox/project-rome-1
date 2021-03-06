---
title: Publishing and reading User Activities (Android)
description: This guide shows how to create, publish, and read Windows-based User Activities in your Android app.
keywords: microsoft, windows, project rome, Android api reference 
---

# Publishing and reading User Activities (Android)

> **Important:** Before you use this guide, make sure you have completed all of the preliminary steps laid out in [Getting started with Connected Devices](getting-started-rome-android.md). Additionally, you will need to have completed the "Preliminary setup" section of the [Hosting guide](hosting-android.md).

User Activities are data constructs that represent a user's tasks within an application. They make it possible to save a snapshot of a current task to be continued at a later time. The [Windows Timeline](https://blogs.windows.com/windowsexperience/2018/04/27/make-the-most-of-your-time-with-the-new-windows-10-update/) feature presents Windows users with a scrollable list of all their recent activities, represented as cards with text and graphics. For more information about User Activities in general, see [Continue user activity, even across devices](https://docs.microsoft.com/windows/uwp/launch-resume/useractivities).

With the Project Rome SDK, your Android app can not only publish User Activities for use in Windows features such as Timeline, but it can also act as an endpoint and read Activities back to the user just as Timeline does. This allows cross-device apps to completely transcend their platforms and present experiences that follow users rather than devices.

> Important: As this is a preview release, there are some known bugs in the Project Rome platform. Currently, Windows apps cannot read Activities created by Android/iOS apps, and the same holds true for the reverse. We are working on a timely fix. 

## Initialize a User Activity channel

To implement User Activity features in your app, you will first need to initialize the user activity feed by creating a **UserActivityChannel**. You should treat this like the Platform initialization step in the [Getting started guide](getting-started-rome-android.md): it should be checked and possibly re-done whenever the app comes to the foreground (but not before Platform initialization). 

Note that you will need a signed-in user account. This guide will use the `mSignInHelper` instance created in the Getting started guide. You will also need your cross-platform app ID, which was retrieved through the Microsoft Developer Dashboard registration in the [Hosting guide](hosting-android.md).

The following methods initialize a **UserActivityChannel**.

```Java
private UserActivityChannel mActivityChannel;
private UserDataFeed mUserDataFeed;

// ...

/**
 * Initializes the UserActivityFeed.
 */
public void initializeUserActivityFeed() {

    SyncScope[] scopes = { UserActivityChannel.getSyncScope(), UserNotificationChannel.getSyncScope() };
    
    // defined below
    mUserDataFeed = getUserDataFeed(scopes, new EventListener<UserDataFeed, Void>() {
        @Override
        public void onEvent(UserDataFeed userDataFeed, Void aVoid) {
            if (userDataFeed.getSyncStatus() == UserDataSyncStatus.SYNCHRONIZED) {
                // log synchronized.
            } else {
                // log synchronization not completed.
            }
        }
    });

    mActivityChannel = getUserActivityChannel();
}

// define the 
private UserDataFeed getUserDataFeed(SyncScope[] scopes, EventListener<UserDataFeed, Void> listener) {
    UserAccount[] accounts = AccountProviderBroker.getSignInHelper().getUserAccounts();
    if (accounts.length <= 0) {
        // notify user that sign-in is required
        return null;
    }

    // use the initialized Platform instance, along with the cross-device app ID.
    UserDataFeed feed = UserDataFeed.getForAccount(accounts[0], PlatformBroker.getPlatform(), Secrets.APP_HOST_NAME);
    feed.addSyncStatusChangedListener(listener);
    feed.addSyncScopes(scopes);
    // sync data with the server
    feed.startSync();
    return feed;
}

@Nullable
private UserActivityChannel getUserActivityChannel() {

    UserActivityChannel channel = null;
    try {
        // create a UserActivityChannel for the signed in account
        channel = new UserActivityChannel(mUserDataFeed);

    } catch (Exception e) {
        e.printStackTrace();
        // handle exception
    }
    return channel;
}


```

At this point, you should have a **UserActivityChannel** reference in `mActivityChannel`.

## Create and publish a User Activity

Next, set the ID, DisplayText and ActivationURI data of what will be a new **UserActivity**. The ID should be a unique String. The DisplayText will be shown on other devices when they view the Activity (in Windows Timeline, for example). The ActivationUri will determine what action is taken when the UserActivity is activated (when it is selected in Timeline, for example). The following code fills in sample data for these fields.


```Java
private UserActivity mActivity;
private String mActivityId;
private String mDisplayText;
private String mActivationUri;

// ...

mActivityId = UUID.randomUUID().toString();
mDisplayText = "Created by OneSDK Sample App";
mActivationUri = "http://contoso.com");

```

Next, provide a method that creates a new **UserActivity** instance.

```Java
// Create the UserActivity with a custom method. 
mActivity = createUserActivity(mActivityChannel, mActivityId);

// ...

/**
* Custom method for creating a new UserActivity
*/
@Nullable
private UserActivity createUserActivity(UserActivityChannel channel, String activityId)
{
    UserActivity activity = null;
    AsyncOperation<UserActivity> activityOperation = channel.getOrCreateUserActivityAsync(activityId);
    try {
        activity = activityOperation.get();
        Log.d("UserActivityFragment","Created user activity successfully");
    } catch (Exception e) {
        e.printStackTrace();
        Log.d("UserActivityFragment","Created user activity successfully");
    }
    return activity;
}
```
Once you have the **UserActivity** instance, populate it with the data defined earlier.

```Java
//set the properties of the UserActivity
//Display Text will be shown when the UserActivity is viewed on other devices
mActivity.getVisualElements().setDisplayText(mDisplayText);
//ActivationURI will determine what is launched when your UserActivity is activated from other devices
mActivity.setActivationUri(mActivationUri);
```

For more information on the different ways that a UserActivity can be customized, see the **[UserActivity](https://docs.microsoft.com/java/api/com.microsoft.connecteddevices.useractivities._user_activity)**, **[UserActivityVisualElements](https://docs.microsoft.com/java/api/com.microsoft.connecteddevices.useractivities._user_activity_visual_elements)**, and **[UserActivityAttribution](https://docs.microsoft.com/java/api/com.microsoft.connecteddevices.useractivities._user_activity_attribution)** classes.

Finally, publish the Activity to the cloud. 

```Java
// This code saves & publishes the activity
AsyncOperation<Void> operation = mActivity.saveAsync();
operation.whenCompleteAsync(new AsyncOperation.ResultBiConsumer<Void, Throwable>() {
    @Override
    public void accept(Void aVoid, Throwable throwable) throws Throwable {
        if (throwable != null)
        {
            Log.d("UserActivityFragment", "Failed to save");
        } else {
            Log.d("UserActivityFragment", "User activity saved");
        }
    }
});
```

## Update an existing User Activity

If you have an existing activity and wish to update its information (in the event of a new engagement, changed page, and so on), you can do so by using a **UserActivitySession**.

```Java
private UserActivitySession mActivitySession;

// ...

if (mActivity != null)
{
    mActivitySession = mActivity.createSession();
    Log.d("UserActivityFragment", "Starting");
```

Once you've created a session, your app can make any desired changes to the properties of the **UserActivity**. When you're done making changes, close the session. 

```Java
mActivitySession.close();
mActivitySession = null;
Log.d("UserActivityFragment", "Stopping");
```

A **UserActivitySession** can be viewed as a way to create a **UserActivitySessionHistoryItem** (covered in the next section). Rather than create a new **UserActivity** every time a user moves to a new page, you can simply create a new session for each page. This will make for a more intuitive and organized activity reading experience.

## Read User Activities

Your app can read User Activities and present them to the user just as the Windows Timeline feature does. To set up User Activity reading, you use the same **UserActivityChannel** as explained at the beginning of this guide. The app receives **UserActivitySessionHistoryItem** instances, which represent a user's engagement in a particular Activity during a specific period of time.

```Java
private ArrayList<UserActivitySessionHistoryItem> mHistoryItems; 

// ...

mHistoryItems.clear();

Log.d("UserActivityFragment", "Read");
//Async method to read activities. Last (most recent) 5 activities will be returned
AsyncOperation<UserActivitySessionHistoryItem[]> operation = mActivityChannel.getRecentUserActivitiesAsync(5);
operation.whenCompleteAsync(new AsyncOperation.ResultBiConsumer<UserActivitySessionHistoryItem[], Throwable>() {
    @Override
    public void accept(UserActivitySessionHistoryItem[] result, Throwable throwable) throws Throwable {
        if (throwable != null)
        {
            Log.d("UserActivityFragment", "Failed to read user activity!" + throwable.getMessage() + " " + throwable.getStackTrace());
        } else {
            for (UserActivitySessionHistoryItem item : result)
            {
                mHistoryItems.add(item);
            }
            mActivityStatus.setText("Read user activity successfully");
        }
    }
});
```

Now your app should have a populated list of **UserActivitySessionHistoryItem**s. Each of these can deliver the underlying **UserActivity** (see [UserActivitySessionHistoryItem](https://docs.microsoft.com/java/api/com.microsoft.connecteddevices.useractivities._user_activity_session_history_item) for details), which you can then display to the user.
