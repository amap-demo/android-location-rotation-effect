# android-location-rotation-effect
定位图标箭头指向手机朝向示例

## 前述 ##
- [高德官网申请Key](http://lbs.amap.com/dev/#/).
- 阅读
  [地图SDK参考手册](http://a.amap.com/lbs/static/unzip/Android_Map_Doc/index.html). 
  [定位SDK参考手册](http://a.amap.com/lbs/static/unzip/Android_Location_Doc/index.html).
- 工程基于高德地图及定位SDK实现

## 使用方法##
###1:配置搭建AndroidSDK工程###
- [Android Studio工程搭建方法](http://lbs.amap.com/api/android-sdk/guide/creat-project/android-studio-creat-project/#add-jars).
- [通过maven库引入SDK方法](http://lbsbbs.amap.com/forum.php?mod=viewthread&tid=18786).

## 扫一扫安装##
![Screenshot](https://github.com/amap-demo/android-location-rotation-effect/raw/master/resource/download.png)

## 用到产品 ##
 - Android 3D地图 SDK
 - Android 定位 SDK

## 核心类/接口##
### AMap  
 - setLocationSource(LocationSource locationSource)  设置定位资源（V2.0版本起）
 - addMarker(MarkerOptions options) 加一个Marker（标记）到地图上（V2.0版本起）
 - addCircle(CircleOptions options) 添加圆形（circle）覆盖物到地图上（V2.0版本起）
 
### AMapLocationClient
 - startLocation();	启动定位	（V2.0.0版本起）
 - setLocationOption(mLocationOption);	给定位客户端设置参数	（V2.0.0版本起）
 
### AMapLocationListener	
 - onLocationChanged(AMapLocation amapLocation);	监听器回调方法	（V2.0.0版本起）
 
## 核心难点##
 - 获取手机sensor角度值并过滤
 ```java
    @Override
    public void onSensorChanged(SensorEvent event) {
        if (System.currentTimeMillis() - lastTime < TIME_SENSOR) {
            return;
        }
        switch (event.sensor.getType()) {
            case Sensor.TYPE_ORIENTATION: {
                float x = event.values[0];
                x += getScreenRotationOnPhone(mContext);
                x %= 360.0F;
                if (x > 180.0F)
                    x -= 360.0F;
                else if (x < -180.0F)
                    x += 360.0F;

                if (Math.abs(mAngle - x) < 3.0f) {
                    break;
                }
                mAngle = Float.isNaN(x) ? 0 : x;
                if (mMarker != null) {
                    mMarker.setRotateAngle(360 - mAngle);
                }
                lastTime = System.currentTimeMillis();
            }
        }

    }
```
- 获取当前手机屏幕旋转角度
```java
    /**
     * 获取当前屏幕旋转角度
     *
     * @param context
     * @return 0表示是竖屏; 90表示是左横屏; 180表示是反向竖屏; 270表示是右横屏
     */
    public static int getScreenRotationOnPhone(Context context) {
        final Display display = ((WindowManager) context
                .getSystemService(Context.WINDOW_SERVICE)).getDefaultDisplay();

        switch (display.getRotation()) {
            case Surface.ROTATION_0:
                return 0;

            case Surface.ROTATION_90:
                return 90;

            case Surface.ROTATION_180:
                return 180;

            case Surface.ROTATION_270:
                return -90;
        }
        return 0;
    }
```

 - 定位成功回调处理Marker与Circle绘制
```java
            if (!mFirstFix) {
                mFirstFix = true;
                addCircle(location, amapLocation.getAccuracy());//添加定位精度圆
                addMarker(location);//添加定位图标
                mSensorHelper.setCurrentMarker(mLocMarker);//定位图标旋转
                aMap.moveCamera(CameraUpdateFactory.newLatLngZoom(location,18));
            } else {
                mCircle.setCenter(location);
                mCircle.setRadius(amapLocation.getAccuracy());
                mLocMarker.setPosition(location);
                aMap.moveCamera(CameraUpdateFactory.changeLatLng(location));
            }
```



