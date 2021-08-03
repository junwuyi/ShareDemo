# ShareDemo

将现有view生成bitmap，用于分享

# 原理
将布局绘制成位图并保存，然后调用系统自带的分享功能
# 添加存储权限

1. 在AndroidManifest.xml 声明权限
```java
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/0273a849b73a499ab761c040ad292ad2.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzI0MDg2,size_16,color_FFFFFF,t_70)
2. 高版本Android 要需要动态申请权限
```java
    /**
     * 动态申请权限
     */
    private void checkNeedPermissions() {
        //6.0以上需要动态申请权限
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                != PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)
                != PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE)
                != PackageManager.PERMISSION_GRANTED) {
            //多个权限一起申请
            ActivityCompat.requestPermissions(this, new String[]{
                    Manifest.permission.CAMERA,
                    Manifest.permission.WRITE_EXTERNAL_STORAGE,
                    Manifest.permission.READ_EXTERNAL_STORAGE
            }, 1);
        }
    }
```
# 截屏保存
```java
    /**
     * 截屏
     */
    public void captureScreen() {
        Runnable action = new Runnable() {
            @Override
            public void run() {
                /*获取windows中最顶层的view*/
                final View contentView = getWindow().getDecorView();
                try {
                    Log.e("chen", contentView.getHeight() + ":" + contentView.getWidth());
                    bitmap = Bitmap.createBitmap(contentView.getWidth(), contentView.getHeight(), Bitmap.Config.ARGB_4444);
                    contentView.draw(new Canvas(bitmap));
                    ByteArrayOutputStream byteOut = new ByteArrayOutputStream();
                    //将位图的压缩到指定的OutputStream，可以理解成将Bitmap保存到文件中！
                    bitmap.compress(Bitmap.CompressFormat.JPEG, 100, byteOut);
                    File file = new File(Environment.getExternalStorageDirectory().getAbsolutePath() + "/test/");
                    if (!file.exists()) {
                        file.mkdirs();
                    }
                    savePic(bitmap, file.toString() + File.separator + "short.png");
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    try {
                        if (null != byteOut)
                            byteOut.close();
                        if (null != bitmap && !bitmap.isRecycled()) {
//                            bitmap.recycle();
                            bitmap = null;
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        try {
            action.run();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 保存位图到本地
     *
     * @param bitmap
     * @param strFileName
     */
    private void savePic(Bitmap bitmap, String strFileName) {
        Log.i("filepath:", strFileName);
        //Toast.makeText(MainActivity.this, strFileName, Toast.LENGTH_SHORT).show();
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream(strFileName);
            if (null != fos) {
                boolean success = bitmap.compress(Bitmap.CompressFormat.PNG, 100, fos);
                fos.flush();
                fos.close();
                if (success) {
                    //Toast.makeText(MainActivity.this, "截屏成功", Toast.LENGTH_SHORT).show();
                    //分享
                    shareImage(strFileName);
                }
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```
# 分享
```java

    /**
     * 将本地图片分享到第三方
     *
     * @param path 为本地文件绝对路径
     */
    public void shareImage(String path) {
        //解决Android7.0调用 Uri.fromFile() 报错FileUriExposedException
        StrictMode.VmPolicy.Builder builder = new StrictMode.VmPolicy.Builder();
        StrictMode.setVmPolicy(builder.build());
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
            builder.detectFileUriExposure();
        }
        //由文件得到uri
        Uri imageUri = Uri.fromFile(new File(path));
        Log.d("share", "uri:" + imageUri);
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND);
        shareIntent.putExtra(Intent.EXTRA_STREAM, imageUri);
        //图片
        shareIntent.setType("image/*");
        startActivity(Intent.createChooser(shareIntent, "分享到"));
    }
```
# 全部代码 
1. activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main_out_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/bing_pic_img"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scaleType="centerCrop"
        android:src="@mipmap/night" />

    <Button
        android:id="@+id/btn_cut"
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:layout_alignParentRight="true"
        android:layout_marginRight="10dp"
        android:background="@mipmap/share"
        android:button="@null"
        android:scaleType="centerCrop" />

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="50dp"
        android:orientation="vertical">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="12"
            android:textColor="@color/white" />

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Button" />
    </LinearLayout>

</RelativeLayout>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/226147fad9d245dbae71c3d40a1c6b6c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzI0MDg2,size_16,color_FFFFFF,t_70)
2. MainActivity

```java
package com.example.sharedemo;

import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.os.Environment;
import android.os.StrictMode;
import android.util.Log;
import android.view.View;
import android.widget.Button;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

public class MainActivity extends AppCompatActivity {

    private Bitmap bitmap = null;
    static ByteArrayOutputStream byteOut = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        checkNeedPermissions();

        Button button = findViewById(R.id.btn_cut);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                captureScreen();
            }
        });
    }

    /**
     * 动态申请权限
     */
    private void checkNeedPermissions() {
        //6.0以上需要动态申请权限
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
                != PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)
                != PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE)
                != PackageManager.PERMISSION_GRANTED) {
            //多个权限一起申请
            ActivityCompat.requestPermissions(this, new String[]{
                    Manifest.permission.CAMERA,
                    Manifest.permission.WRITE_EXTERNAL_STORAGE,
                    Manifest.permission.READ_EXTERNAL_STORAGE
            }, 1);
        }
    }

    /**
     * 截屏
     */
    public void captureScreen() {
        Runnable action = new Runnable() {
            @Override
            public void run() {
                /*获取windows中最顶层的view*/
                final View contentView = getWindow().getDecorView();
                try {
                    Log.e("chen", contentView.getHeight() + ":" + contentView.getWidth());
                    bitmap = Bitmap.createBitmap(contentView.getWidth(), contentView.getHeight(), Bitmap.Config.ARGB_4444);
                    contentView.draw(new Canvas(bitmap));
                    ByteArrayOutputStream byteOut = new ByteArrayOutputStream();
                    //将位图的压缩到指定的OutputStream，可以理解成将Bitmap保存到文件中！
                    bitmap.compress(Bitmap.CompressFormat.JPEG, 100, byteOut);
                    File file = new File(Environment.getExternalStorageDirectory().getAbsolutePath() + "/test/");
                    if (!file.exists()) {
                        file.mkdirs();
                    }
                    savePic(bitmap, file.toString() + File.separator + "short.png");
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    try {
                        if (null != byteOut)
                            byteOut.close();
                        if (null != bitmap && !bitmap.isRecycled()) {
//                            bitmap.recycle();
                            bitmap = null;
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        };
        try {
            action.run();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 保存位图到本地
     *
     * @param bitmap
     * @param strFileName
     */
    private void savePic(Bitmap bitmap, String strFileName) {
        Log.i("filepath:", strFileName);
        //Toast.makeText(MainActivity.this, strFileName, Toast.LENGTH_SHORT).show();
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream(strFileName);
            if (null != fos) {
                boolean success = bitmap.compress(Bitmap.CompressFormat.PNG, 100, fos);
                fos.flush();
                fos.close();
                if (success) {
                    //Toast.makeText(MainActivity.this, "截屏成功", Toast.LENGTH_SHORT).show();
                    //分享
                    shareImage(strFileName);
                }
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 将本地图片分享到第三方
     *
     * @param path 为本地文件绝对路径
     */
    public void shareImage(String path) {
        //解决Android7.0调用 Uri.fromFile() 报错FileUriExposedException
        StrictMode.VmPolicy.Builder builder = new StrictMode.VmPolicy.Builder();
        StrictMode.setVmPolicy(builder.build());
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
            builder.detectFileUriExposure();
        }
        //由文件得到uri
        Uri imageUri = Uri.fromFile(new File(path));
        Log.d("share", "uri:" + imageUri);
        Intent shareIntent = new Intent();
        shareIntent.setAction(Intent.ACTION_SEND);
        shareIntent.putExtra(Intent.EXTRA_STREAM, imageUri);
        //图片
        shareIntent.setType("image/*");
        startActivity(Intent.createChooser(shareIntent, "分享到"));
    }
}
```
# 效果如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/8aaae94ef6ba43c0b2c16753a11da94c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzI0MDg2,size_16,color_FFFFFF,t_70)
**可以在 Device File Explorer 中查看到图片**
![在这里插入图片描述](https://img-blog.csdnimg.cn/4bb6a4f898ab4d968d99f4bfac19aacf.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzI0MDg2,size_16,color_FFFFFF,t_70)
磁盘位置
**C:\Users\Chen\Documents\AndroidStudio\DeviceExplorer\Pixel_2_API_28 [emulator-5554]\sdcard\test**
![在这里插入图片描述](https://img-blog.csdnimg.cn/088f1eb003e443daa9e2a0ab0aeb9bfa.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyMzI0MDg2,size_16,color_FFFFFF,t_70)
