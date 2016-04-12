---
title: actionBar
date: 2016-04-10 19:56:26
categories: Android
tags: 77
comments: true

---
## 利用actionBar做拍照/从相册选取
### 1. 设置ActionBar

在AndroidManifest.xml中指定Application或Activity的theme是Theme.Holo或其子类。在Android 3.0及更高的版本中，Activity中都默认包含有ActionBar组件。

    <style name="protraitBarTheme" parent="@android:style/Theme.Holo.Light">
        <item name="android:actionBarStyle">@style/MyActionBar</item>
        <item name="android:actionBarTabTextStyle">@style/MyActionBarTabText</item>
    </style>
其中protraitBarTheme的主题是自定义的继承于Theme.Holo的子类Theme.Holo.Light，在styles.xml中可以自定义主题，代码如下：   

    <style name="protraitBarTheme" parent="@android:style/Theme.Holo.Light">
        <item name="android:actionBarStyle">@style/MyActionBar</item>
        <item name="android:actionBarTabTextStyle">@style/MyActionBarTabText</item>
    </style>
    <style name="MyActionBar" parent="@android:style/Widget.Holo.Light.ActionBar">
        <item name="android:background">#f4842d</item>
        <item name="android:actionBarSize">150dip</item>
    </style>
    <style name="MyActionBarTabText" parent="@android:style/Widget.Holo.ActionBar.TabText">
        <item name="android:textColor">#fff</item>
    </style>
    
    
 在AndroidManifest.xml设置actionBar的logo和文字显示：
 
    <activity android:theme="@style/protraitBarTheme"
            android:name=".editProtraitActivity"
            android:label="Ta"
            android:logo="@mipmap/main_icon">
    </activity>
     
### 2.设置Action按钮
在menu下创建一个新的.xml布局文件，布局item选项按钮：

    <menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context="com.example.seven.peoplehub.MainActivity">
    <item
        android:id="@+id/takePic"
        android:title="拍照"/>
    <item
        android:id="@+id/album"
        android:title="从相册中选择"/>
    </menu>
  
  
### 3.Activity中menu处理

  Activity启动的时候，系统会调用Activity的onCreateOptionsMenu()方法来取出所有的Action按钮，我们只需要在这个方法中去加载一个menu资源，并把所有的Action按钮都定义在资源文件里面就可以了。
  
  
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.menu_protrait, menu);
        return super.onCreateOptionsMenu(menu);
    }

### 4.Action按钮的点击事件

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.album: {
                Toast.makeText(this, "“进入相册模式”！", Toast.LENGTH_SHORT).show();
                Intent intent1 = new Intent(Intent.ACTION_PICK, null);
                intent1.setDataAndType(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, "image/*");
                startActivityForResult(intent1, 1);
                return true;
            }
            case R.id.takePic: {
                Toast.makeText(this, "“进入拍照选择”！", Toast.LENGTH_SHORT).show();
                Intent intent2 = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
                intent2.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(new File(Environment.getExternalStorageDirectory(),
                        "head.jpg")));
                startActivityForResult(intent2, 2);//采用ForResult打开
                return true;
            }
            default:
            return super.onOptionsItemSelected(item);
        }
    }
    
### 5.头像显示控件初始化
在activity中需要先对头像图片的显示控件做初始化：

    private Bitmap head;//头像Bitmap
    private static String path="/sdcard/myHead/";//sd路径
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_edit_protrait);
        initView();
    }

    private void initView() {
        //初始化控件
        ivHead = (ImageView) findViewById(R.id.iv_head);
        Bitmap bt = BitmapFactory.decodeFile(path + "head.jpg");//从Sd中找头像，转换成Bitmap
        if(bt!=null){
            @SuppressWarnings("deprecation")
            Drawable drawable = new BitmapDrawable(bt);//转换成drawable
            ivHead.setImageDrawable(drawable);
        }else{
            /**
             *	如果SD里面没有 则需要从服务器取头像，取回来的头像再保存在SD中
             *
             */
        }
    }
    
 *最后要记得设置权限:*
 
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    
 *参考链接：*
 
 [http://www.mincoder.com/article/1847.shtml](http://www.mincoder.com/article/1847.shtml)
 [http://www.cnblogs.com/yc-755909659/p/4290784.html](http://www.cnblogs.com/yc-755909659/p/4290784.html
)
