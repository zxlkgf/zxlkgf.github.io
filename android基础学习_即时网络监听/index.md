# Android基础学习_即时网络监听


---

# 1.即时网络监听
## 1.1 使用BroadCast监听网络变化
---

### 1.1.1 创建广播接受类
1. 创建广播接受类并继承BroadcastReceiver  
2. 重写onReciver方法  
3. 根据intent的action判断是不是自己需要反应的动作(本次反应对象为ConnectivityManager.CONNECTIVITY_ACTION)  
```java
public class NetworkChangeBroadCast extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if(action.equals(ConnectivityManager.CONNECTIVITY_ACTION)){
            Toast.makeText(context,"CONNECTIVITY_ACTION_CHANGED",Toast.LENGTH_SHORT).show();
            Log.d("ZHAO","CONNECTIVITY_ACTION_CHANGED");
        }
    }
}

```
### 1.1.2 在MainActivity内部注册广播
1. 在onCreate内部注册广播
2. 在ondestroy内部注销广播

```java
public class BroadCastActivity extends AppCompatActivity {
    private NetworkChangeBroadCast networkChangeBroadCast;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //注册过滤器
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(ConnectivityManager.CONNECTIVITY_ACTION);
        //广播接受者
        networkChangeBroadCast = new NetworkChangeBroadCast();
        //注册广播
        registerReceiver(networkChangeBroadCast,intentFilter);
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        //解除广播
        unregisterReceiver(networkChangeBroadCast);
    }
}
```

---

## 1.2 使用ConnectivityManager.NetworkCallback监控网络

[官方文档](https://developer.android.google.cn/reference/android/net/ConnectivityManager.NetworkCallback.html)

---

### 1.2.1 新建NetworkCallbackImpl类
1. 新建NetworkCallbackImpl类继承ConnectivityManager.NetworkCallback
2. 重写onAvailable、onLost、onCapabilitiesChanged
```java
public class NetworkCallbackImpl extends ConnectivityManager.NetworkCallback {
    private  static final String TAG = "ZHAO";
    //网络可用时
    @Override
    public void onAvailable(@NonNull Network network) {
        super.onAvailable(network);
        Log.d("ZHAO","onAvailable: 网络已连接");
    }
    //网络不可用时
    @Override
    public void onLost(@NonNull Network network) {
        super.onLost(network);
        Log.d("ZHAO","onLost: 网络已断开");
    }


    @Override
    public void onCapabilitiesChanged(@NonNull Network network, @NonNull NetworkCapabilities networkCapabilities) {
        super.onCapabilitiesChanged(network, networkCapabilities);
        if(networkCapabilities.hasCapability(NetworkCapabilities.NET_CAPABILITY_VALIDATED)){
            if(networkCapabilities.hasTransport(networkCapabilities.TRANSPORT_WIFI)){
                Log.d(TAG, "onCapabilitiesChanged: 网络类型为wifi");
            }else if (networkCapabilities.hasTransport(networkCapabilities.TRANSPORT_CELLULAR)){
                Log.d(TAG, "onCapabilitiesChanged: 蜂窝网络");
            }else {
                Log.d(TAG, "其他网络网络");
            }
        }
    }
}

```

---

### 1.2.2 注册NetworkCallbackImpl
```java
        NetworkCallbackImpl networkCallbackImpl = new NetworkCallbackImpl();
        NetworkRequest.Builder builder = new NetworkRequest.Builder();
        NetworkRequest request = builder.build();
        ConnectivityManager connectivityManager = (ConnectivityManager)getSystemService(Context.CONNECTIVITY_SERVICE);
        if(connectivityManager!=null){
            connectivityManager.registerNetworkCallback(request,networkCallbackImpl);
        }
```

---

