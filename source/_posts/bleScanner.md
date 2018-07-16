---
title: 安卓BLE开发总结一
date: 2018-07-12 10:29:14
tags:
---

### 1.蓝牙扫描
通过服务号搜索BLE设备

<font size=3>**注意:通过服务号扫描需要确认BleServer端是否把服务号广播出来** </font>

```java

 BluetoothManager bluetoothManager = ((BluetoothManager)context.getSystemService(Context.BLUETOOTH_SERVICE));
 BluetoothAdapter bluetoothAdapter = bluetoothManager.getAdapter();
 bluetoothAdapter.startLeScan(new UUID[]{UUID.fromString(BinderConstant.UUID.SERVICE_UUID)},new BluetoothAdapter.LeScanCallback() {
                            @Override
                            public void onLeScan(BluetoothDevice device, int rssi, byte[] scanRecord) {
                                
                            }
                        })
```

### 2.蓝牙状态开关广播

监听蓝牙开关状态的广播action:BluetoothAdapter.ACTION_STATE_CHANGED
<font size=3>**蓝牙关闭开启需要重新获取bluetoothAdapter** </font>

```java
 public class BluetoothReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            Logger.i("onReceive BluetoothAdapter Status Change Broadcast");
            if (intent != null && intent.getAction() != null && BluetoothAdapter.ACTION_STATE_CHANGED.equals(intent.getAction())) {
                int bluetoothState = intent.getIntExtra(BluetoothAdapter.EXTRA_STATE, BluetoothAdapter.ERROR);
                if (bluetoothState == BluetoothAdapter.STATE_ON) {
                    bluetoothManager = ((BluetoothManager) context.getSystemService(Context.BLUETOOTH_SERVICE));
                    if (null != bluetoothManager) {
                        bluetoothAdapter = bluetoothManager.getAdapter();
                    }
                    Logger.i("onReceive: is called  blue tooth on !");
                } else if (bluetoothState == BluetoothAdapter.STATE_OFF) {
                    bluetoothAdapter = null;
                    Logger.i("onReceive: is called  blue tooth off !");
                }
            }
        }
    }
```


### 3.蓝牙常见错误分析

<font size=3>**ble蓝牙连接的代码很简单，但是很容易出现133 0 错误** </font>

解决思路：
1.连接 断开 关闭操作尽量在主线程进行 
2.出现错误之后及时close gatt通道进行重连

```java
    class MyBluetoothGattCallback extends BluetoothGattCallback {
        @Override
        public void onConnectionStateChange(BluetoothGatt gatt, int status, int newState) {
            super.onConnectionStateChange(gatt, status, newState);
            Logger.i("-------- onConnectionStateChange: status: " + status +
                    ", newState: " + newState);
            bluetoothGatt = gatt;
            if (status == BluetoothGatt.GATT_SUCCESS) {
                switch (newState) {
                    case BluetoothProfile.STATE_CONNECTED:
                        Logger.i("onConnectionStateChange: STATE_CONNECTED");
                       
                        break;
                    //主动断开
                    case BluetoothProfile.STATE_DISCONNECTED:
                        Logger.i("onConnectionStateChange: STATE_DISCONNECTED");
                        
                        break;
                    default:
                }
            } else {
                Logger.e("onConnectionStateChange callBack connect failed Name="
                        + gatt.getDevice().getName());
                //1.close蓝牙通道 2.重新连接
            }
        }
    }

```


