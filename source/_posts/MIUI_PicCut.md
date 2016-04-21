---
title: MIUI系统图片获取与裁剪
date: 2016-04-12 13:30:16
categories: Android
tags: 77
comments: true

---
## MIUI系统图片获取与裁剪（衔接上一篇：ActionBar实现拍照/从图片库取图片）
### 1. data.getExtras()取值为空解决方法
裁剪后的图片通过Intent的putExtra("return-data",true)方法进行传递，miui系统问题就出在这里，return-data的方式只适用于小图，miui系统默认的裁剪图片可能裁剪得过大，或对return-data分配的资源不足，造成return-data失败。

```Java
     //Bundle extras = data.getExtras();
     //Bitmap head = extras.getParcelable("data");
```
**当data不为空的时候 这里的extras和head去取值 结果都为空！！**

解决思路是：裁剪后，将裁剪的图片保存在Uri中，在onActivityResult()方法中，再提取对应的Uri图片转换为Bitmap使用。
其实大家直观也能感觉出来，Intent主要用于不同Activity之间通信，是一种动态的小巧的资源占用，类似于Http请求中的GET，并不适用于传递图片之类的大数据。于是当A生成一个大数据要传递给B，往往不是通过Intent直接传递，而是在A生成数据的时候将数据保存到C，B再去调用C，C相当于一个转换的中间件。

```Java
    public void cropPhoto(Uri uri) {
        Intent intent3 = new Intent("com.android.camera.action.CROP");
        intent3.setDataAndType(uri, "image/*");
        intent3.putExtra("crop", "true");
        // aspectX aspectY 是宽高的比例
        intent3.putExtra("aspectX",1);
        intent3.putExtra("aspectY",1);
        // outputX outputY 是裁剪图片宽高
        intent3.putExtra("outputX",150);
        intent3.putExtra("outputY",150);
        intent3.putExtra("return-data", true);
        uritempFile = Uri.parse("file://" + "/" + Environment.getExternalStorageDirectory().getPath() + "/" + "small.jpg");
        intent3.putExtra(MediaStore.EXTRA_OUTPUT, uritempFile);
        intent3.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
        startActivityForResult(intent3,3);
    }   
```   
    
onActivityResult()中处理:

```Java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        // TODO Auto-generated method stub
        case PHOTO_REQUEST_CUT: 
            //将Uri图片转换为Bitmap
            Bitmap bitmap = BitmapFactory.decodeStream(getContentResolver().openInputStream(uritempFile));
	    //TODO，将裁剪的bitmap显示在imageview控件上
            break;
        }
        super.onActivityResult(requestCode, resultCode, data);
    }
```
 **参考链接：**
[http://m.blog.csdn.net/article/details?id=42719217](http://m.blog.csdn.net/article/details?id=42719217/)


### 2. Intent命名问题：

在onOptionsItemSelected()方法中，选择拍照模式和从相册取图片两种模式时分别根据menu选择创建了两个Intent，命名为intent1和intent2，接下来cropPhoto()方法中新建裁剪的Intent，但是这里我起初以下列方式：
       
     Intent intent = new Intent("com.android.camera.action.CROP");
命名，最终裁剪结束后不会       startActivityForResult(intent,3);不会执行onActivityResult()的回调。而我花了8个小时去查错，偶然重命名为intent3:

     Intent intent3 = new Intent("com.android.camera.action.CROP");
后，运行结果正常了。目前我对于Intent的回调机制还不是很清楚，这个问题先记录在此，可能是由于其他页面跳转建立的Intent冲突，日后再补充。

```Java
    private ImageView ivHead;//头像显示
    private Bitmap head;//头像Bitmap
    private Uri uritempFile;//大图转换
    private ImageView ivHead;//头像显示
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
            //如果SD里面没有 则需要从服务器取头像，取回来的头像再保存在SD中
            return;
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.menu_protrait, menu);
        return super.onCreateOptionsMenu(menu);
    }
    
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.album:
                Toast.makeText(this, "进入相册选择！", Toast.LENGTH_SHORT).show();
                Intent intent1 = new Intent(Intent.ACTION_PICK,null);
                intent1.setType("image/*");
                startActivityForResult(intent1,1);
                break;
            case R.id.takePic:
                Toast.makeText(this, "“进入拍照模式”！", Toast.LENGTH_SHORT).show();
                Intent intent2 = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
                intent2.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(new File(Environment.getExternalStorageDirectory(),
                        "head.jpg")));
                startActivityForResult(intent2,2);//采用ForResult打开
                break;
        }
        return super.onOptionsItemSelected(item);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        switch (requestCode) {
            case 1:
                if (resultCode == RESULT_OK&&data!=null) {
                    cropPhoto(data.getData());//裁剪图片
                }
                break;
            case 2:
                if (resultCode == RESULT_OK) {
                    File temp = new File(Environment.getExternalStorageDirectory()
                            + "/head.jpg");
                    cropPhoto(Uri.fromFile(temp));//裁剪图片
                }
                break;
            case 3:
                 try {

                     //Bundle extras = data.getExtras();
                     //head = extras.getParcelable("data");
                     head = BitmapFactory.decodeStream(getContentResolver().openInputStream(uritempFile));
                     if (head != null) {                   
                         setPicToView(head);//保存在SD卡中
                         ivHead.setImageBitmap(head);
                     }
                 }
                    catch (FileNotFoundException e) {
                        e.printStackTrace();
                    }
                break;
            default:
                break;
        }
        super.onActivityResult(requestCode, resultCode, data);
    };
    //调用系统的裁剪  @param uri

    public void cropPhoto(Uri uri) {
        Intent intent3 = new Intent("com.android.camera.action.CROP");
        intent3.setDataAndType(uri, "image/*");
        intent3.putExtra("crop", "true");
        // aspectX aspectY 是宽高的比例
        intent3.putExtra("aspectX",1);
        intent3.putExtra("aspectY",1);
        // outputX outputY 是裁剪图片宽高
        intent3.putExtra("outputX",150);
        intent3.putExtra("outputY",150);
        intent3.putExtra("return-data", true);
        uritempFile = Uri.parse("file://" + "/" + Environment.getExternalStorageDirectory().getPath() + "/" + "small.jpg");
        intent3.putExtra(MediaStore.EXTRA_OUTPUT, uritempFile);
        intent3.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
        startActivityForResult(intent3,3);
    }

     private void setPicToView(Bitmap mBitmap) {
        String sdStatus = Environment.getExternalStorageState();
        if (!sdStatus.equals(Environment.MEDIA_MOUNTED)) { // 检测sd是否可用
            return;
        }
        FileOutputStream b = null;
        File file = new File(path);
        file.mkdirs();// 创建文件夹
        String fileName =path + "head.jpg";//图片名字
        try {
            b = new FileOutputStream(fileName);
            mBitmap.compress(Bitmap.CompressFormat.JPEG, 100, b);// 把数据写入文件
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            try {
                //关闭流
                b.flush();
                b.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```
