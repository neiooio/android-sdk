# Getting Started on Android  


### Install SDK
Neioo SDK requires API level 18+ and Bluetooth Low Energy.

#### Android Studio Users

##### 1. Integrate .jar and .aar files

Build a new libs folder at root directory and unarchive the downloaded SDK to the folder.

##### 2. Edit build.gradle file
In your `build.gradle` add `flatDir` entry to your repositories.
```
repositories {
    flatDir {
        dirs 'libs'
    }
}
```
##### 3. Add dependency
Add dependency to Neioo SDK.
```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile(name:'neioo-sdk', ext:'aar')
}
```
##### 4. Finished！
You should now be able to use Neioo in your app.

#### Eclipse Users
Mark Murphy [on his blog explained](https://commonsware.com/blog/2014/07/03/consuming-aars-eclipse.html) how to use aar format in Eclipse.

***


### Set Up Neioo Cloud
All the information are defined on Neioo Cloud, so after installing Neioo SDK, you can begin to set up the Neioo Cloud.

##### 1. Add Beacon
Firstly, beacon must be added onto the cloud, so that we can attach events on the beacon. There are two ways to add beacon onto the cloud：  

* Enter data manually  
    In Beacon page, click `Add Beacon` button, you can see the picture below
    ![](/screenshot/add_beacon.png)

* Use Neioo Officer

##### 2. Create Space
Space is a container of event.  
In Space page, click `Add Space` button, then you will see the picture below
![](/screenshot/space_settings.png)

##### 3. Create Action
You can decide which action takes place when an event is triggered.
In Action page, click `Add Action` button, then you will see the picture below
![](/screenshot/show_image.png)  
Let's create a `Show Image on user's App` action for practice.

##### 4. Finished！
There are already enough materials on your Neioo Cloud, the next implementation will combine these materials


*To learn more about Neioo objects, please see [Overview](/)。*

***


### Implementation : Basic Event Trigger
This section will take you step by step to achieve a simple trigger event.
##### 1. Create Campaign
Campaign is a trigger event, which contains a lot of trigger-related settings, such as how often the campaign triggered, start time and end time of campaign, the range of triggered.
In Space page, click `Add Campaign to Space` button, you can see the picture below
![](/screenshot/add_campaign01.png)  

* In `Activate beacons` block, choose which beacons to trigger campaign
* Set trigger range in `Trigger Proximity`
* Set whether campaign can be triggered repeatedly in `Repeat Setting`
* In `Activate actions` block, choose which action takes place when an event is triggered.
* Click `Add New Rule` button to create campaign

You can see campaigns you created in space page
![](/screenshot/add_campaign03.png)  

##### 2. Implement NeiooCallback - onEnterSpace() and onExitSpace()
Let's back to the Android project to implements NeiooCallback.  
NeiooCallback is the event interface of Neioo, which contains five callback methods.
```
NeiooCallback neiooCallback = new NeiooCallback() {
    @Override
    public void onCampaignTriggered(NeiooBeacon neiooBeacon, NeiooCampaign neiooCampaign) {
        // will be called when campaign is triggered
    }
    @Override
    public void onEnterSpace(NeiooSpace neiooSpace) {
        // will be called when enter a space
    }
    @Override
    public void onExitSpace(NeiooSpace neiooSpace) {
        // will be called when exit a space
    }
    @Override
    public void inShakeRange(NeiooCampaign neiooCampaign) {
        // will be called when shake campaign is in range
    }
    @Override
    public void outShakeRange(NeiooCampaign neiooCampaign) {
        // will be called when shake campaign is out of range
    }
};
```
Please ignore `inShakeRange ()` and `outShakeRange ()` for now, it will be mentioned later.  
`onEnterSpace()` will be called when sdk correctly get space's data, and is ready to trigger the event of this space.
We can prompt user with Toast in `onEnterSpace ()` and `onExitSpace ()`. Let's adjust neiooCallback  
```
@Override
public void onEnterSpace(final NeiooSpace neiooSpace) {
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            Toast.makeText(MainActivity.this, "Enter " + neiooSpace.getName(), Toast.LENGTH_LONG).show();
        }
    });
}

@Override
public void onExitSpace(final NeiooSpace neiooSpace) {
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            Toast.makeText(MainActivity.this,"Exit " + neiooSpace.getName(),Toast.LENGTH_LONG).show();
        }
    });
}
```
The callback function is run in a non-UI thread. Please execute your code with Handler or Activity.runOnUiThread if necessary

##### 3. Implement NeiooCallback - onCampaignTriggered()
`onCampaignTriggered()` will be called when campaign is triggered, and return with NeiooCampaign object which contains all the information you define on Neioo Cloud.  
You can obtain actions of campaign with `NeiooCampaign.getActions()`. Let's adjust neiooCallback：
```
@Override
public void onCampaignTriggered(NeiooBeacon neiooBeacon, NeiooCampaign neiooCampaign) {
    for(NeiooAction action : neiooCampaign.getActions()){
        if(action.getType().equals("show_image")){
            Log.e("onCampaignTriggered",action.getDetail().toString());
        }
    }
}
```
We use `Log.e()` to display action detail, you can use `Action.getDetail()` to get all the content you define on the Neioo Cloud.  
After obtaining this information, you can decide how to display on your App.

##### 4. Initialized and enabled Neioo
Neioo can only get instance through `Neioo.setUp()`, it is recommended to initialize the Application class.  
There are three params of Neioo.setUp() :

* Application Context  

* Neioo App Key  
You can find it at the top of the space page.
![](/screenshot/appkey.png)  

* NeiooCallback  

```
try {
    Neioo neioo = Neioo.setUp(context,"YourAppKey",neiooCallback);
    neioo.enable();
} catch (NeiooException e) {
    e.printStackTrace();
}
```

##### 5. Testing Result
When you move the phone close to the beacon with campaign, campaign will be triggered by Neioo. Campaign's triggered distance will follow the definitions on the Neioo Cloud.  
Congratulations! Now that you've learned how to create a basic event trigger.
***


### Implementation : Conditional Event Trigger
Now, let's do slightly advanced features - creating the criteria for conditional campaign triggered. We can target certain users to trigger this campaign.

##### 1. Create Criteria
In this example, we use the user's `age` as target value. Campaign will be triggered when user's `age is greater than 18`.  
![](/screenshot/add_criteria.png)

##### 2. Set Criteria for the Campaign
You can choose the criteria you created in `Activate criteria` block. Other settings can be consistent with the previous implementation.
![](/screenshot/add_campaign02.png)  

The difference between this campaign and previous one is that there is a light blue criteria label in this campaign. It is mean that the campaign will only be triggered when user match the criteria.
![](/screenshot/add_campaign04.png)  


##### 3. Provide Criteria Data to SDK
You have to provide the data of current user to Neioo sdk, thus sdk will have enough data to determine whether the user match criteria.  
In this example, we need users to enter their age as criteria data. Assume that the current user is 13 years old.
```
neioo.addCriteriaData("age","13");
```

##### 4. Testing Result
You can set the user's age `> 18` or `< 18` to verify whether the results are correct. In this example, the user is 13 years old, so campaign will not be triggered.  


***


### Implementation : Shake Campaign
In this scenario, when the user enters beacon range, the event will not be triggered immediately until the phone shaking.

##### 1. Create Shake Campaign
To create shake campaign, you need to set the `shake|` tag in campaign's `customize setting` like this:  
![](/screenshot/add_shake_campaign.png)

##### 2. Implement NeiooCallback - inShakeRange() and outShakeRange()
`inShakeRange()` and `outShakeRange()` will be called when the user is inside/outside the shake range, you can show some UI to inform the user to shake his/her phone.
```
@Override
public void inShakeRange(final NeiooCampaign neiooCampaign) {
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            Toast.makeText(MainActivity.this, "In shake range " + neiooCampaign.getId(), Toast.LENGTH_LONG).show();
        }
    });
}

@Override
public void outShakeRange(final NeiooCampaign neiooCampaign) {
    runOnUiThread(new Runnable() {
        @Override
        public void run() {
            Toast.makeText(MainActivity.this, "Out of shake range " + neiooCampaign.getId(), Toast.LENGTH_LONG).show();
        }
    });
}
```

##### 3. Detect Shaking
Neioo can only notify which campaign should be triggered by shaking, so you need to detect shake event by your self.  
There are many tutorials and open-source projects about it like [Seismic](https://github.com/square/seismic),  [ShakeDetector](https://github.com/tbouron/ShakeDetector)

##### 4. Get Shake Campaign
If the user is shaking his / her phone, `onShake()` will be called. Then we can use `Neioo.getShakeCampaigns()` to get shake campaigns directly.
```
@Override
public void onShake() {
    for (NeiooCampaign campaign : neioo.getShakeCampaigns()) {
        Toast.makeText(MainActivity.this,"Trigger shake campaign "+campaign.getId(),Toast.LENGTH_LONG).show();
    }
}
```

##### 5.Testing Result
The campaign will only be triggered when user is shaking their phone in range.
