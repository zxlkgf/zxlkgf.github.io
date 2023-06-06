# TelePhonyManager


# TelephonyManager类笔记
提供对有关设备上的电话服务的信息的访问。应用程序可以使用此类中的方法来确定电话服务和状态，以及访问某些类型的用户信息。应用程序还可以注册一个侦听器以接收电话状态更改的通知。  

```java
//TelePhonyManager对象的获取
mTelephonyManager = (TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE);
```
需要注意的是,需要在AndroidManifest.xml内部声明权限。
```xml
<!--允许读取电话状态SIM的权限-->
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<!-- 这个权限用于进行网络定位 -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

部分TelephonyManager由于权限问题，无法输出显示。

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private static final String TAG = "zhao";
    private Button btn_click;
    private TelephonyManager mTelephonyManager;
    private StringBuilder sb;
    private TextView tv_message;


    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        Log.d("MainActivity", "MainActivity onCreate");
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        tv_message = findViewById(R.id.tv_message);
        btn_click = findViewById(R.id.btn_click);
        sb = new StringBuilder();

        mTelephonyManager = (TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE);
        btn_click.setOnClickListener(this);
    }

        private void setMessage(){
        //获取手机当前位置
        //<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
//        String cellLocation = mTelephonyManager.getCellLocation().toString();
//        sb.append("CellLocation Info : ").append(cellLocation).append("\n");

            /*
            获取手机的卡槽个数
             */
        int phoneCount = mTelephonyManager.getPhoneCount();
        sb.append("phoneCount : ").append(phoneCount).append("\n");
            /*
         * 获取数据连接状态
         * DATA_CONNECTED 数据连接状态：已连接
         * DATA_CONNECTING 数据连接状态：正在连接
         * DATA_DISCONNECTED 数据连接状态：断开
         * DATA_SUSPENDED 数据连接状态：暂停
         */
        int dataState = mTelephonyManager.getDataState();
        setDataState(dataState);

        /*
         * 返回唯一的设备ID
         * 如果是GSM网络，返回IMEI；如果是CDMA网络，返回MEID；如果设备ID是不可用的返回null
         */
//        String deviceId = mTelephonyManager.getDeviceId();
//        sb.append("deviceId : ").append(deviceId).append("\n");
//
//
        /*
         * 返回设备的软件版本号
         * 例如：GSM手机的IMEI/SV码，如果软件版本是返回null，如果不可用返回null
         */
//        String deviceSoftwareVersion = mTelephonyManager.getDeviceSoftwareVersion();
//        sb.append("deviceSoftwareVersion : ").append(deviceSoftwareVersion).append("\n");
//
        /*
         * 返回手机号码
         * 对于GSM网络来说即MSISDN，如果不可用返回null
         *
         */
//        @SuppressLint("MissingPermission") String line1Number = mTelephonyManager.getLine1Number();
//        sb.append("line1Number : ").append(line1Number).append("\n");
//
//
//        /*
//         * 返回当前设备附近设备的信息
//         * getNeighboringCellInfo() 在>=29 被删除
//         */
//
//        /*
//         * 返回ISO标准的国家码，即国际长途区号
//         */
        String networkCountryIso = mTelephonyManager.getNetworkCountryIso();
        sb.append("networkCountryIso : ").append(networkCountryIso).append("\n");
//
//        /*
//         * 返回MCC+MNC代码 (SIM卡运营商国家代码和运营商网络代码)(IMSI)
//         */
        String networkOperator = mTelephonyManager.getNetworkOperator();
        sb.append("networkOperator : ").append(networkOperator).append("\n");
//
//        /*
//         * 返回移动网络运营商的名字(SPN)
//         */
        String networkOperatorName = mTelephonyManager.getNetworkOperatorName();
        sb.append("networkOperatorName : ").append(networkOperatorName).append("\n");
//
//
//        /*
//         * 获取网络类型
//         * NETWORK_TYPE_CDMA 网络类型为CDMA
//         * NETWORK_TYPE_EDGE 网络类型为EDGE
//         * NETWORK_TYPE_EVDO_0 网络类型为EVDO0
//         * NETWORK_TYPE_EVDO_A 网络类型为EVDOA
//         * NETWORK_TYPE_GPRS 网络类型为GPRS
//         * NETWORK_TYPE_HSDPA 网络类型为HSDPA
//         * NETWORK_TYPE_HSPA 网络类型为HSPA
//         * NETWORK_TYPE_HSUPA 网络类型为HSUPA
//         * NETWORK_TYPE_UMTS 网络类型为UMTS
//         *
//         * 在中国，联通的3G为UMTS或HSDPA，移动和联通的2G为GPRS或EGDE，电信的2G为CDMA，电信的3G为EVDO
//         */
//        @SuppressLint("MissingPermission") int networkType = mTelephonyManager.getNetworkType();
//        setNetWorkType(networkType);
//
//        /*
//         * 返回设备的类型
//         *
//         * PHONE_TYPE_CDMA 手机制式为CDMA，电信
//         * PHONE_TYPE_GSM 手机制式为GSM，移动和联通
//         * PHONE_TYPE_NONE 手机制式未知
//         */
        int phoneType = mTelephonyManager.getPhoneType();
        setPhoneType(phoneType);
//
//        /*
//         * 返回SIM卡提供商的国家代码
//         */
        String simCountryIso = mTelephonyManager.getSimCountryIso();
        sb.append("simCountryIso : ").append(simCountryIso).append("\n");
//
//        /*
//         * 返回MCC+MNC代码 (SIM卡运营商国家代码和运营商网络代码)(IMSI)
//         */
        String simOperator = mTelephonyManager.getSimOperator();
        sb.append("simOperator : ").append(simOperator).append("\n");
//
//        /*
//         * 返回服务提供者的名称(SPN)
//         */
        String simOperatorName = mTelephonyManager.getSimOperatorName();
        sb.append("simOperatorName : ").append(simOperatorName).append("\n");
//
//        /*
//         * 返回SIM卡的序列号(IMEI),如果是返回null为不可用。
//         */
//        String simSerialNumber = mTelephonyManager.getSimSerialNumber();
//        sb.append("simSerialNumber : ").append(simSerialNumber).append("\n");
//
        /*
         * 返回一个常数表示默认的SIM卡的状态。
         * SIM_STATE_ABSENT SIM卡未找到
         * SIM_STATE_NETWORK_LOCKED SIM卡网络被锁定，需要Network PIN解锁
         * SIM_STATE_PIN_REQUIRED SIM卡PIN被锁定，需要User PIN解锁
         * SIM_STATE_PUK_REQUIRED SIM卡PUK被锁定，需要User PUK解锁
         * SIM_STATE_READY SIM卡可用
         * SIM_STATE_UNKNOWN SIM卡未知
         */
        int simState = mTelephonyManager.getSimState();
        setSimState(simState);

//        /*
//         * 返回唯一的用户ID,例如,IMSI为GSM手机。
//         */
//        String subscriberId = mTelephonyManager.getSubscriberId();
//        sb.append("subscriberId : ").append(subscriberId).append("\n");
//
//        /*
//         * 获取语音信箱号码关联的字母标识
//         */
//        @SuppressLint("MissingPermission") String voiceMailAlphaTag = mTelephonyManager.getVoiceMailAlphaTag();
//        sb.append("voiceMailAlphaTag : ").append(voiceMailAlphaTag).append("\n");
//
//        /*
//         * 返回语音邮件号码
//         */
//        @SuppressLint("MissingPermission") String voiceMailNumber = mTelephonyManager.getVoiceMailNumber();
//        sb.append("voiceMailNumber : ").append(voiceMailNumber).append("\n");
//
//        /*
//         * 返回手机是否处于漫游状态
//         */
        boolean networkRoaming = mTelephonyManager.isNetworkRoaming();
        sb.append("networkRoaming : ").append(networkRoaming==true?"true":"false").append("\n");
//
    }
    private void setSimState(int simState) {
        sb.append("simState : ");
        switch (simState) {
            case TelephonyManager.SIM_STATE_ABSENT:
                sb.append("SIM_STATE_ABSENT");
                break;
            case TelephonyManager.SIM_STATE_NETWORK_LOCKED:
                sb.append("SIM_STATE_NETWORK_LOCKED");
                break;
            case TelephonyManager.SIM_STATE_PIN_REQUIRED:
                sb.append("SIM_STATE_NETWORK_LOCKED");
                break;
            case TelephonyManager.SIM_STATE_PUK_REQUIRED:
                sb.append("SIM_STATE_PUK_REQUIRED");
                break;
            case TelephonyManager.SIM_STATE_READY:
                sb.append("SIM_STATE_READY");
                break;
            case TelephonyManager.SIM_STATE_UNKNOWN:
                sb.append("SIM_STATE_UNKNOWN");
                break;
            default:
                break;
        }
        sb.append("\n");
    }

    private void setPhoneType(int phoneType) {
        sb.append("phoneType : ");
        switch (phoneType) {
            case TelephonyManager.PHONE_TYPE_CDMA:
                sb.append("PHONE_TYPE_CDMA");
                break;
            case TelephonyManager.PHONE_TYPE_GSM:
                sb.append("PHONE_TYPE_GSM");
                break;
            case TelephonyManager.PHONE_TYPE_NONE:
                sb.append("PHONE_TYPE_NONE");
                break;
            case TelephonyManager.PHONE_TYPE_SIP:
                sb.append("PHONE_TYPE_SIP");
                break;
            default:
                break;
        }
        sb.append("\n");
    }

    private void setNetWorkType(int netWorkType) {
        sb.append("networkType : ");
        switch (netWorkType) {
            case TelephonyManager.NETWORK_TYPE_CDMA:
                sb.append("NETWORK_TYPE_CDMA");
                break;
            case TelephonyManager.NETWORK_TYPE_EDGE:
                sb.append("NETWORK_TYPE_EDGE");
                break;
            case TelephonyManager.NETWORK_TYPE_EVDO_0:
                sb.append("NETWORK_TYPE_EVDO_0");
                break;
            case TelephonyManager.NETWORK_TYPE_EVDO_A:
                sb.append("NETWORK_TYPE_EVDO_A");
                break;
            case TelephonyManager.NETWORK_TYPE_GPRS:
                sb.append("NETWORK_TYPE_GPRS");
                break;
            case TelephonyManager.NETWORK_TYPE_HSDPA:
                sb.append("NETWORK_TYPE_HSDPA");
                break;
            case TelephonyManager.NETWORK_TYPE_HSPA:
                sb.append("NETWORK_TYPE_HSDPA");
                break;
            case TelephonyManager.NETWORK_TYPE_HSUPA:
                sb.append("NETWORK_TYPE_HSUPA");
                break;
            case TelephonyManager.NETWORK_TYPE_UMTS:
                sb.append("NETWORK_TYPE_UMTS");
                break;
            default:
                break;
        }
        sb.append("\n");
    }

    private void setDataState(int dataState) {
        sb.append("dataState : ");
        switch (dataState) {
            case TelephonyManager.DATA_DISCONNECTED:
                sb.append("DATA_DISCONNECTED");
                break;
            case TelephonyManager.DATA_CONNECTING:
                sb.append("DATA_CONNECTING");
                break;
            case TelephonyManager.DATA_CONNECTED:
                sb.append("DATA_CONNECTED");
                break;
            case TelephonyManager.DATA_SUSPENDED:
                sb.append("DATA_SUSPENDED");
                break;
            case TelephonyManager.DATA_DISCONNECTING:
                sb.append("DATA_DISCONNECTING");
                break;
            case TelephonyManager.DATA_HANDOVER_IN_PROGRESS:
                sb.append("DATA_HANDOVER_IN_PROGRESS");
                break;
            default:
                break;
        }
        sb.append("\n");
    }

    @Override
    public void onClick(View v) {
        if (v.getId() == R.id.btn_click) {
             setMessage();
             tv_message.setText(sb.toString());
         }
    }
}

```

下面一些解释：
(1)IMSI是国际移动用户识别码的简称(International Mobile Subscriber Identity)  
IMSI共有15位，其结构如下：  
MCC+MNC+MIN  
MCC：Mobile Country Code，移动国家码，共3位，中国为460;  
MNC:Mobile NetworkCode，移动网络码，共2位  
在中国，移动的代码为电00和02，联通的代码为01，电信的代码为03  
合起来就是（也是Android手机中APN配置文件中的代码）：  
中国移动：46000 46002  
中国联通：46001  
中国电信：46003  

(2)IMEI是International Mobile Equipment Identity （国际移动设备标识）的简称  
IMEI由15位数字组成的”电子串号”，它与每台手机一一对应，而且该码是全世界唯一的  
其组成为：  
1. 前6位数(TAC)是”型号核准号码”，一般代表机型  
2. 接着的2位数(FAC)是”最后装配号”，一般代表产地  
3. 之后的6位数(SNR)是”串号”，一般代表生产顺序号  
4. 最后1位数(SP)通常是”0″，为检验码，目前暂备用  


(3)slotId和subId
slotId即为手机SIM卡槽的个数，通常从以0,1表示。
subId即为手机插入的SIM卡个数，通常从1,2表示。
