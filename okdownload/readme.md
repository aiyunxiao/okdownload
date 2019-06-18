### 简介
本项目基于 okdownload 库1.0.5版本源码修改而成

### 使用说明

集成方式

```groovy
dependencies {
    implementation 'com.github.aiyunxiao:okdownload:v1.0.5.1'
}
```

### 使用方式
参考官方readme




### 迭代记录
    v1.0.5.1 修改源码。增加key字段 


###  修改源码记录
     辅导下载中存在下载的url和token有关，无法作为唯一的标识，所以增加一个唯一标示key DownloadTask中的
     具体改动
```java 
    private String key;
 // iyunxiao 添加的属性，辅导下载任务的唯一标识，因为辅导下载的url同一个视频受token影响。不同时间获取的下载url不同

 public DownloadTask(String url, Uri uri, int priority, int readBufferSize, int flushBufferSize,
                        int syncBufferSize, int syncBufferIntervalMills,
                        boolean autoCallbackToUIThread, int minIntervalMillisCallbackProcess,
                        Map<String, List<String>> headerMapFields, @Nullable String filename,
                        boolean passIfAlreadyCompleted, boolean wifiRequired,
                        Boolean filenameFromResponse, @Nullable Integer connectionCount,
                        @Nullable Boolean isPreAllocateLength,
                        String key) {
        this.url = url;
        this.uri = uri;
        this.priority = priority;
        this.readBufferSize = readBufferSize;
        this.flushBufferSize = flushBufferSize;
        this.syncBufferSize = syncBufferSize;
        this.syncBufferIntervalMills = syncBufferIntervalMills;
        this.autoCallbackToUIThread = autoCallbackToUIThread;
        this.minIntervalMillisCallbackProcess = minIntervalMillisCallbackProcess;
        this.headerMapFields = headerMapFields;
        this.lastCallbackProcessTimestamp = new AtomicLong();
        this.passIfAlreadyCompleted = passIfAlreadyCompleted;
        this.wifiRequired = wifiRequired;
        this.connectionCount = connectionCount;
        this.isPreAllocateLength = isPreAllocateLength;
        this.key = key;
        if (Util.isUriFileScheme(uri)) {
            final File file = new File(uri.getPath());
            if (filenameFromResponse != null) {
                if (filenameFromResponse) {
                    // filename must from response.
                    if (file.exists() && file.isFile()) {
                        // it have already provided file for it.
                        throw new IllegalArgumentException("If you want filename from "
                                + "response please make sure you provide path is directory "
                                + file.getPath());
                    }

                    if (!Util.isEmpty(filename)) {
                        Util.w("DownloadTask", "Discard filename[" + filename
                                + "] because you set filenameFromResponse=true");
                        filename = null;
                    }

                    directoryFile = file;
                } else {
                    // filename must not from response.
                    if (file.exists() && file.isDirectory() && Util.isEmpty(filename)) {
                        // is directory but filename isn't provided.
                        // not valid filename found.
                        throw new IllegalArgumentException("If you don't want filename from"
                                + " response please make sure you have already provided valid "
                                + "filename or not directory path " + file.getPath());
                    }

                    if (Util.isEmpty(filename)) {
                        filename = file.getName();
                        directoryFile = Util.getParentFile(file);
                    } else {
                        directoryFile = file;
                    }
                }
            } else if (file.exists() && file.isDirectory()) {
                filenameFromResponse = true;
                directoryFile = file;
            } else {
                // not exist or is file.
                filenameFromResponse = false;

                if (file.exists()) {
                    // is file
                    if (!Util.isEmpty(filename) && !file.getName().equals(filename)) {
                        throw new IllegalArgumentException("Uri already provided filename!");
                    }
                    filename = file.getName();
                    directoryFile = Util.getParentFile(file);
                } else {
                    // not exist
                    if (Util.isEmpty(filename)) {
                        // filename is not provided, so we use the filename on path
                        filename = file.getName();
                        directoryFile = Util.getParentFile(file);
                    } else {
                        // filename is provided, so the path on file is directory
                        directoryFile = file;
                    }
                }
            }

            this.filenameFromResponse = filenameFromResponse;
        } else {
            this.filenameFromResponse = false;
            directoryFile = new File(uri.getPath());
        }

        if (Util.isEmpty(filename)) {
            filenameHolder = new DownloadStrategy.FilenameHolder();
            providedPathFile = directoryFile;
        } else {
            filenameHolder = new DownloadStrategy.FilenameHolder(filename);
            targetFile = new File(directoryFile, filename);
            providedPathFile = targetFile;
        }

        this.id = OkDownload.with().breakpointStore().findOrCreateId(this);
    }

        // 在builder中添加
        public Builder setKey(String key){
            this.key = key;
            return this;
        }

        // 修改builder的build方法
        public DownloadTask build() {
            return new DownloadTask(url, uri, priority, readBufferSize, flushBufferSize,
                    syncBufferSize, syncBufferIntervalMillis,
                    autoCallbackToUIThread, minIntervalMillisCallbackProcess,
                    headerMapFields, filename, passIfAlreadyCompleted, isWifiRequired,
                    isFilenameFromResponse, connectionCount, isPreAllocateLength,key);
        }



    public String getKey() {
        return key;
    }
```     
        
    KeyToIdMap类修改
```java
   String generateKey(@NonNull DownloadTask task) {
           String key = task.getKey();
           if (key != null && !key.isEmpty()) {
               return key;
           }
           return task.getUrl() + task.getUri() + task.getFilename();
       }
```     