# Android基础学习_MediaPlayer(音乐播放器)


# 基于原生安卓的音乐播放器
## 1.Ui界面
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/ll_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/bg"
    android:gravity="center"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        >
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="30dp"
            android:layout_weight="1"
            android:textSize="17sp"
            android:textColor="@color/black"
            android:background="@color/white"
            android:text="选取歌曲"/>
        <Spinner
            android:id="@+id/spinner"
            android:layout_weight="1"
            android:background="@color/white"
            android:layout_width="wrap_content"
            android:layout_height="30dp"
            />
    </LinearLayout>

    <ImageView
        android:id="@+id/lv_music"
        android:layout_width="240dp"
        android:layout_height="240dp"
        android:layout_gravity="center_horizontal"
        android:layout_margin="15dp"
        android:src="@drawable/music_disk"
        />
    <SeekBar
        android:id="@+id/sb"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:progressBackgroundTint="#fff"/>
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="8dp"
        android:paddingRight="8dp"
        >
        <TextView
            android:id="@+id/tv_progress"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="#fff"
            android:text="00:00"
            />
        <TextView
            android:id="@+id/tv_total"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:textColor="#fff"
            android:text="00:00"
            />
    </RelativeLayout>
    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        >
        <Button
            android:id="@+id/btn_play"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:paddingLeft="10dp"
            android:paddingRight="10dp"
            android:text="播放音乐"/>
        <Button
            android:id="@+id/btn_pause"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:paddingLeft="10dp"
            android:paddingRight="10dp"
            android:text="暂停播放"/>
        <Button
            android:id="@+id/btn_continue"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:paddingLeft="10dp"
            android:paddingRight="10dp"
            android:text="继续播放"/>
        <Button
            android:id="@+id/btn_exit"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:paddingLeft="10dp"
            android:paddingRight="10dp"
            android:text="退出播放"/>
    </LinearLayout>

</LinearLayout>
```

## MainActivity部分

```java
public class MainActivity extends AppCompatActivity {
    private static final String TAG = "MainActivity";
    private static final boolean DEBUG = true;
    private LinearLayout ll_main ;
    private Spinner spinner;
    private ImageView lv_music;
    private static SeekBar seekBar;
    private static TextView tv_progress;
    private static TextView tv_total;
    private Button btn_play,btn_pause,btn_continue,btn_exit;
    private ObjectAnimator objectAnimator;
    private String[] music_name;
    private HashMap<String,Integer[]> map;

    private MusicControllerService.musicController musicController;


    public static Handler mHandler = new Handler(){
        @Override
        public void handleMessage(@NonNull Message msg) {
            super.handleMessage(msg);
            Bundle data = msg.getData();
            int duration = data.getInt("duration");//获取从子线程发来的音乐总时长
            int currentPosition = data.getInt("currentPosition");//获取从子线程发来的播放进度
            seekBar.setMax(duration);//设置seekBar的最大歌曲时长
            seekBar.setProgress(currentPosition);//设置seekBar的进度
            String sDuration = msToMinSec(duration);
            String currentPos = msToMinSec(currentPosition);
            tv_progress.setText(currentPos);
            tv_total.setText(sDuration);
        }
    };

    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            musicController = (MusicControllerService.musicController)service;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            musicController = null;
        }
    };

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        init();
    }
    private void init(){
        ll_main = findViewById(R.id.ll_main);
        spinner = findViewById(R.id.spinner);
        lv_music = findViewById(R.id.lv_music);
        seekBar = findViewById(R.id.sb);
        tv_progress = findViewById(R.id.tv_progress);
        tv_total = findViewById(R.id.tv_total);
        btn_play =findViewById(R.id.btn_play);
        btn_pause = findViewById(R.id.btn_pause);
        btn_continue = findViewById(R.id.btn_continue);
        btn_exit = findViewById(R.id.btn_exit);
        music_name = getResources().getStringArray(R.array.music_tag);
        //构建哈希表
        map = new HashMap<String ,Integer[]>(){
            {
                put(music_name[0],new Integer[]{R.raw.jjay_xuebuhui,R.drawable.bg_xuebuhui});
                put(music_name[1],new Integer[]{R.raw.jjay_shengsheng,R.drawable.bg_shengsheng});
                put(music_name[2],new Integer[]{R.raw.jjay_jiaohuanyusheng,R.drawable.bg_jiaohuanyusheng});
            }
        };

        onClick onClick = new onClick();
        btn_play.setOnClickListener(onClick);
        btn_pause.setOnClickListener(onClick);
        btn_continue.setOnClickListener(onClick);
        btn_exit.setOnClickListener(onClick);

        //对象是lv_music,动作是rotation
        objectAnimator = ObjectAnimator.ofFloat(lv_music,"rotation",0,360.0F);
        //设置持续时间，10s一周
        objectAnimator.setDuration(10000);
        //设置旋转为匀速旋转
        objectAnimator.setInterpolator(new LinearInterpolator());
        //设置重复次数,-1为持续
        objectAnimator.setRepeatCount(-1);

        //设置spinner
        //简历adapter并绑定数据源
        ArrayAdapter adapter = new ArrayAdapter<>(getApplicationContext(), android.R.layout.simple_spinner_item,music_name);
        adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
        //绑定adapter控件
        spinner.setAdapter(adapter);
        //设置点击事件
        spinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
            @Override
            public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
                log("onItemSelected , position:"+position+",music src:"+map.get(music_name[position])[0]);
                ll_main.setBackgroundResource(map.get(music_name[position])[1]);
                musicController.setMusicSrc(map.get(music_name[position])[0]);
                objectAnimator.pause();
            }

            @Override
            public void onNothingSelected(AdapterView<?> parent) {

            }
        });

        Intent intent = new Intent(getApplicationContext(), MusicControllerService.class);
        bindService(intent,conn,BIND_AUTO_CREATE);

        seekBar.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
                //当音乐停止
                if(progress == seekBar.getMax()){
                    objectAnimator.pause();
                }
                //判断是不是用户拖动
                if(fromUser){
                    musicController.seekTo(progress);
                }
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {
                musicController.pause();
            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {
                musicController.resume();
            }
        });
    }

    class onClick implements View.OnClickListener{
        @Override
        public void onClick(View v) {
            if(v.getId()==R.id.btn_play){
                log("play");
                //动画开始
                objectAnimator.start();
                //音乐播放
                musicController.play();
            } else if (v.getId() == R.id.btn_pause) {
                log("pause");
                //动画暂停
                objectAnimator.pause();
                //音乐暂停
                musicController.pause();
            } else if (v.getId() == R.id.btn_continue) {
                log("continue");
                //动画继续
                objectAnimator.resume();
                //音乐继续
                musicController.resume();
            } else if (v.getId() == R.id.btn_exit) {
                log("exit");
                //退出活动
                finish();
            }
        }
    }

    @Override
    protected void onDestroy() {
        musicController.stop();
        unbindService(conn);
        super.onDestroy();
    }

    private void log(String str){
        if(DEBUG){
            Log.d(TAG,str);
        }
    }
    public static String msToMinSec(int ms){
        int sec = ms/1000;
        int min = sec/60;
        sec -= min * 60;
        return String.format("%02d:%02d",min,sec);
    }
}

```

## service部分
```java
public class MusicControllerService extends Service {
    private static final String TAG = "MusicController";
    private static final boolean DEBUG = true;

    private MediaPlayer mediaPlayer;
    private Timer timer;
    private  Integer music_src;

    public  void setMusic_src(Integer music_src) {
        music_src = music_src;
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return new musicController();
    }

    @Override
    public void onCreate() {
        super.onCreate();
        mediaPlayer = new MediaPlayer();
        music_src = R.raw.jjay_xuebuhui;
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return super.onStartCommand(intent, flags, startId);
    }
    //增加计时器
    private void addTimer(){
        if(timer == null){
            timer = new Timer();
            TimerTask timerTask  = new TimerTask() {
                @Override
                public void run() {
                    int duration = mediaPlayer.getDuration();//获取歌曲总时长
                    int currentPos = mediaPlayer.getCurrentPosition();//获取当前进度
                    Message msg = new Message();
                    //将音乐的总时长和播放进度封装到消息对象中
                    Bundle bundle = new Bundle();
                    bundle.putInt("duration",duration);
                    bundle.putInt("currentPosition",currentPos);
                    msg.setData(bundle);
                    //将消息发送到主线程的消息队列
                    MainActivity.mHandler.sendMessage(msg);
                }
            };
            //开启计时任务后的5ms,第一次执行task之后，每500ms执行一次
            timer.schedule(timerTask,5,500);
        }
    }

    public class musicController extends Binder{
        //播放音乐
        public void play(){
            log("musicController play");
            mediaPlayer.reset();
            mediaPlayer = MediaPlayer.create(getApplicationContext(),music_src);
            mediaPlayer.start();
            addTimer();
        }
        //暂停
        public void pause(){
            log("musicController pause");
            mediaPlayer.pause();
        }
        //继续
        public void resume(){
            log("musicController resume");
            mediaPlayer.start();//播放 不会被重置
        }
        //停止
        public void stop(){
            log("musicController stop");
            mediaPlayer.stop();//停止音乐
            mediaPlayer.release();
            try{
                timer.cancel();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        //打带
        public void seekTo(int ms){
            mediaPlayer.seekTo(ms);
        }
        public void setMusicSrc(int musicSrc){
            music_src = musicSrc;
            mediaPlayer.stop();
            mediaPlayer.release();
            mediaPlayer = MediaPlayer.create(getApplicationContext(),musicSrc);
            log(String.valueOf(musicSrc));
        }
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }

    private void log(String str){
        if(DEBUG){
            Log.d(TAG,str);
        }
    }
}

```


## 思路
### UI界面
创建一个黑胶圆盘，提供播放动画效果  
创建一个SeekBar，提供音乐进度条效果  
创建四个Button，提供播放，暂停，继续，退出的点击事件  

### 主界面
使用ObjectAnimetor,设置黑胶圆盘的动画效果  

提供一个spinner供选择歌曲  
在Spinner选择item的时候，调用onItemSelete切换背景和歌曲源  

提供一个Service,提供播放，暂停，继续，退出的方法实现  
需要重写ServiceConnetion  
并调用bindservice  
记住 在onDestory的时候调用unbindService方法  

### Service
Service内部维护三个成员变量  
1.MediaPlayer 音乐播放器  
2.Timer类  
3.音乐源节点  
并在内部创建一个内部类继承iBInder  
内部类内提供调用的播放，暂停，继续，退出的方法。  

Timer类作为SeekBar的设置，提供一个TimerTask，在每0.5s的时候构建一个Message，将播放的进度返回，主界面接受消息并设置SeekBar。  


