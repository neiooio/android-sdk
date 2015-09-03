開始使用 - Android
=

### 安裝 Neioo SDK   
Neioo SDK 需要 Android 4.3 以上，支援 BLE 的裝置才可以正常運作。

#### Android Studio 使用者

##### 1. 加入 .jar 檔和 .aar 檔
在專案根目錄下建立一個 `libs` 資料夾，並將所有 `.jar` 檔和 `.aar` 檔加入其中。

##### 2. 調整 build.gradle 檔案
在 `build.gradle` 中的 `repositories` 區塊中加入 `flatDir`。
```
repositories {
    flatDir {
        dirs 'libs'
    }
}
```
##### 3. 添加依賴
在 `dependencies` 區塊中指定 Neioo SDK 及其他 `.jar` 檔。
```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile(name:'neioo-sdk', ext:'aar')
}
```
##### 4. 完成！
現在你的 Android 專案已經可以引用 Neioo 的類別和方法了。

#### Eclipse 使用者
Mark Murphy 的[這篇文章](https://commonsware.com/blog/2014/07/03/consuming-aars-eclipse.html)解釋了如何在 eclipse 使用 `.aar` 檔。  
不過更推薦的方式是：開始改用 Android Studio 吧！

***


### 設定 Neioo Cloud
Neioo 的一切事件都定義在 Cloud 上，所以安裝完 SDK 之後，我們先來設定 Neioo Cloud。

##### 1. 建立 Beacon
實體的 Beacon 必須先登錄到 Cloud 上，才能為 Beacon 附加事件。有兩個管道可以將 Beacon 登錄到 Cloud 上：  

* 手動輸入資料  
    在 Beacon 頁面點擊 `Add Beacon` 按鈕可以看到畫面如下  
    ![](/screenshot/add_beacon.png)

* 使用 [Neioo Officer]()

##### 2. 建立 Space
Space 代表一個場域，也是事件的容器。要辦活動得先有場地才行。  
在 Space 頁面點擊 `Add Space` 按鈕可以看到畫面如下   
![](/screenshot/space_settings.png)

##### 3. 建立 Action
Action  就是 Neioo SDK 在事件觸發時該做的動作，Neioo 定義了 10 種類別。  
在 Action 頁面點擊 `Add Action` 按鈕可以看到畫面如下   
![](/screenshot/show_image.png)  
這裏我們先建一個 type 為 `Show Image on user's App` 的 Action 來練習。  

##### 4. 完成！
現在你的 Neioo App 上已經有足夠的素材了，接下來的實作就會把這些素材給兜在一起。  

*更多關於 Neioo 物件的說明，請參考 [Overview](/)。*

***


### 實作基本事件觸發
這個實作將一步一步帶你實現一個最簡單的觸發事件。完成後你的 App 在接近 Beacon 時就會依照 Cloud 上所定義的反應。
##### 1. 建立 Campaign
Campaign 就是一個觸發事件，定義了觸發的 Beacon、距離、時間...等條件。這個範例我們建立一個最簡單的。  
在 Space 按下 `Add Campaign to Space` 按鈕會看到這樣的畫面：  
![](/screenshot/add_campaign01.png)  

* 在 Activate beacons 選擇事件由哪顆 Beacon 觸發，就指定之前登錄的 `Demo Beacon`   
* 設定 Trigger Proximity ，表示事件將在距離 Beacon 多近時觸發，有近、中、遠三種可以選
* 設定 Repeat Setting ，表示事件是否可以重複觸發。如果設定為重複觸發的話，可以設定兩次觸發之間必須間隔多久時間
* 在 Activate actions 選擇觸發時要做什麼 Action，指定剛才建立的 `Show Image`
* 按下 Add New Rule，Campaign 就建立完成了。  

可以在 Space 頁面看到你所建立的 Campaign 列表  
![](/screenshot/add_campaign03.png)  

##### 2. 實作 NeiooCallback - 進出 Space 的事件
現在回到 Android 專案上，實作 Neioo 的事件接口 - NeiooCallback。包含了五個回調函式：
```java
NeiooCallback neiooCallback = new NeiooCallback() {
    @Override
    public void onCampaignTriggered(NeiooBeacon neiooBeacon, NeiooCampaign neiooCampaign) {
        // 當裝置經過符合條件的 campaign 時被呼叫。
    }
    @Override
    public void onEnterSpace(NeiooSpace neiooSpace) {
        // 當裝置離開進入 Space 時被呼叫，返回進入的 Space 資料。
    }
    @Override
    public void onExitSpace(NeiooSpace neiooSpace) {
        // 當裝置離開離開 Space 時被呼叫，返回離開的 Space 資料。
    }
    @Override
    public void inShakeRange(NeiooCampaign neiooCampaign) {
        // 當裝置進入符合條件的 campaign 範圍，且 campaign 有「搖一搖」屬性時被呼叫。
    }
    @Override
    public void outShakeRange(NeiooCampaign neiooCampaign) {
        // 當裝置離開有「搖一搖」屬性的 campaign 的範圍時被呼叫。
    }
};
```
這裏先忽略 `inShakeRange()`和 `outShakeRange()`，之後的實作會提到。  
`onEnterSpace()` 會在裝置已經正確取得 Space 的資料時被呼叫，表示 SDK 已經準備好觸發這個 Space 的事件了。  
我們可以在 `onEnterSpace()` 和 `onExitSpace()` 用 Toast 來提示使用者，修改一下 neiooCallback。  
```java
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
特別注意 NeiooCallback 的回調函式都不是在 UI 線程上被調用，所以跟 UI 相關的任務，別忘了用 `runOnUiThread()` 或是 `Handler.post()` 來執行。

##### 3. 實作 NeiooCallback - Campaign 觸發的事件
當有事件觸發時， NeiooCallback 的 `onCampaignTriggered()` 就會被呼叫，並回傳 NeiooCampagin 物件，包含了你在 Neioo Cloud 上定義的所有資訊；還有觸發 Campaign 的 Beacon，封裝在 NeiooBeacon 裡。     
可以用 NeiooCampagin.getActions() 取得 Campaign 需要做的 Action，並做出反應。修改一下 neiooCallback：
```java
@Override
public void onCampaignTriggered(NeiooBeacon neiooBeacon, NeiooCampaign neiooCampaign) {
    for(NeiooAction action : neiooCampaign.getActions()){
        if(action.getType().equals("show_image")){
            Log.e("onCampaignTriggered",action.getDetail().toString());
        }
    }
}
```
這裏用 Log.e() 來展示 Action 內容，Action.getDetail() 可以拿到在 Cloud 上定義的所有內容，以 JSONObject 包裝。  
取得這些資料之後，要怎麼在你的 App 上呈現就任你發揮。

##### 4. 初始化並啟動 Neioo
Neioo 採獨體模式(Singleton)設計，只能透過 Neioo.setUp() 取得實體，建議在 Application 類中初始化。  
初始化有三個參數，依序是：

* Application Context  

* Neioo App Key  
你可在 Cloud 頁面的最上方找到它。  
![](/screenshot/appkey.png)  

* NeiooCallback  
就傳入我們前兩個步驟寫的 neiooCallback 吧

```java
try {
    Neioo neioo = Neioo.setUp(context,"YourAppKey",neiooCallback);
    neioo.enable();
} catch (NeiooException e) {
    e.printStackTrace();
}
```
啟動之後，就靜待 NeiooCallback 被呼叫吧。

##### 5. 測試結果
你可以試著遠離、靠近 Beacon 幾次，甚至離開它的訊號範圍再回來。Campaign 觸發的條件和動作將跟著 Cloud 的定義變動。  
恭喜～現在你已經學會怎麼創造一個基本的事件觸發了！
***


### 實作條件事件觸發
接著是稍微進階點的功能，為 Campaign 設定觸發的用戶條件，精準地找到想要觸發的特定客群。  
舉個例，一個 `18禁`的 Campaign。

##### 1. 建立 Criteria
Criteria 就是判斷標準，按上面 `18禁` 的例子來建立就會像這樣：  
![](/screenshot/add_criteria.png)

##### 2. 在 Campaign 中使用 Criteria
同上一節寫的方式建立 Campaign，不一樣的地方在於這次要為這個 Campaign 指定 Criteria。  
在 Activate criterias 裡找到剛才建立的 criteria，選取它之後建立Campaign。  
![](/screenshot/add_campaign02.png)  
可以在 Campaign 列表上看到這個 Campaign 跟上一個比起來，多了一個淺藍色的 Criteria 方塊，意思就是只有滿足這個 Criteria 的用戶能觸發這個 Campaign。  
![](/screenshot/add_campaign04.png)  


##### 3. 提供 Neioo 標準資料
Cloud 的設定完成之後，回到 Android 專案。Neioo SDK 必須靠 App 主動提供當前用戶的數值，才有足夠的資料判斷用戶是不是符合 Criteria。所以在設計 Criteria 時，必須考慮到你的 App 是不是能取得對應的資料。  
假設我們的 App 要求使用者輸入年齡才能開始，且當前使用的是一個 13 歲的用戶。在輸入完後就用 `Neioo.addCriteriaData()` 來提供資料給 Neioo 像這樣：
```java
neioo.addCriteriaData("age","13");
```

##### 4. 測試結果
照上面的例子，一個 13 歲的用戶不會觸發 `18禁` 的 Campaign。你也可以試著把 age 設為 18 以上，看看 Campaign 是不是能正常觸發。


***


### 實作搖一搖觸發
搖一搖的情境是，使用者進入 Beacon 範圍後，事件不立即觸發，而是等到使用者搖晃手機之後才觸發。給使用者更多跟 App 互動的機會。

##### 1. 建立搖一搖屬性的 Campaign
Campaign 的基本屬性同上兩節所寫。要使 Campaign 具備藉由搖一搖觸發必須在 Customize setting 中加上 `shake|`的字樣。像這樣：  
![](/screenshot/add_shake_campaign.png)

##### 2. 實作 NeiooCallback - 進出「搖一搖」範圍的事件
為了適時提醒使用者何時該搖手機， NeiooCallback 提供了 `inShakeRange()`和 `outShakeRange()` 兩個回調函式，會在進入/離開搖一搖屬性的 Campaign 時被呼叫。我們一樣用 Toast 來提醒使用者像這樣：
```java
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

##### 3. 偵測搖晃事件
Neioo 只能告訴你搖晃時會觸發什麼事件，不會主動偵測搖晃事件何時發生。所以我們首先需要的是一個「搖晃偵測者」。關於這類的教學網路上很多，也有開源專案像是 [Seismic](https://github.com/square/seismic),  [ShakeDetector](https://github.com/tbouron/ShakeDetector)，這裏就不詳述怎麼實作。

##### 4. 取得搖晃時應觸發的 Campaign
現在我們得以判斷裝置是不是正在搖晃，假設上面的搖晃偵測者有個 `OnShake()` 的 callback，表示搖晃發生。我們就必須在 `OnShake()` 裡使用 `Neioo.getShakeCampaigns()` ，Neioo 將直接返回應觸發的 Campaign。
```java
@Override
public void onShake() {
    for (NeiooCampaign campaign : neioo.getShakeCampaigns()) {
        Toast.makeText(MainActivity.this,"Trigger shake campaign "+campaign.getId(),Toast.LENGTH_LONG).show();
    }
}
```

##### 5.測試結果
試著遠離、靠近 Beacon，觀察 `inShakeRange()`和 `outShakeRange()` 是否如預期發生。再分別在範圍內、範圍外搖晃手機試試 Campaign 是否只有裝置在範圍內時會觸發。
