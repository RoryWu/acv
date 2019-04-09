# 优化之网络

### 概念：

需要监控什么信息？

### 工具:

#### 1. Network Profiler



#### 2. 抓包工具

* Charles
* Fiddler
* Wireshark
* TcpDump



#### 3. Stetho



### 代码层面获取流量情况

```java
private long getNetStats(Context context , long beginTime , long endTime) {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
            return 0;
        }
        long netDataRx = 0 ; // 接受
        long netDataTx = 0 ; // 发送
        TelephonyManager telephonyManager = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);
        @SuppressLint("MissingPermission") String subID = telephonyManager.getSubscriberId();
        NetworkStatsManager manager = (NetworkStatsManager) context.getSystemService(Context.NETWORK_STATS_SERVICE);
        NetworkStats networkStats = null;
        NetworkStats.Bucket bucket = new NetworkStats.Bucket();
        try {
            networkStats = manager.querySummary(NetworkCapabilities.TRANSPORT_WIFI, subID, beginTime , endTime);
        } catch (RemoteException e) {
            e.printStackTrace();
        }

        while ((networkStats.hasNextBucket())) {
            networkStats.getNextBucket(bucket);
            int uid = bucket.getUid();
            if (getUidByPackageName() == uid) {
                netDataRx += bucket.getRxBytes();
                netDataTx += bucket.getTxBytes();
            }
        }

        long  total = netDataRx + netDataTx;
        return total;
    }
```

























































