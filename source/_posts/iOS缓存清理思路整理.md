---
title: iOS缓存清理思路整理
date: 2016-12-30 11:25:43
tags: [iOS,oc]
categories: oc
---

### 获取系统剩余可用的空间

```objective-c
#import <sys/mount.h>

+ (long long) freeDiskSpaceInBytes {
    struct statfs buf;
    long long freespace = -1;
    if(statfs("/var", &buf) >= 0){
        freespace = (long long)(buf.f_bsize * buf.f_bfree);
    }
    return freespace;
}
```

<!-- more -->

### 填充剩余空间

```objective-c

  NSFileManager* fileManager = [[NSFileManager alloc ]init];

  NSString *apppath =[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0];

  NSString *filePath = [apppath stringByAppendingPathComponent:@"test.txt"];

  if (![fileManager fileExistsAtPath:filePath]) {
      [fileManager createFileAtPath:filePath contents:nil attributes:nil];
  }
  NSFileHandle *outFile = [NSFileHandle fileHandleForWritingAtPath:filePath];

  long long truncateFile = [freeSpace longLongValue]-10*1024;
  if (truncateFile > 0) {
      [outFile truncateFileAtOffset:truncateFile];
  }

```

### 删除刚刚写入的文件

```objective-c
    NSFileManager* fileManager = [[NSFileManager alloc ]init];
    NSString *apppath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0];
    NSArray *files = [[NSFileManager defaultManager] subpathsAtPath:apppath];
    for (NSString *p in files) {
        NSError *error;
        NSString *path = [apppath stringByAppendingPathComponent:p];
        if ([fileManager fileExistsAtPath:path]) {
            [fileManager removeItemAtPath:path error:&error];
        }
    }
```



### 原理

获取了系统剩余的空间后,然后通过 truncateFileAtOffset 写入一个和剩余空间大小差不多的文件,出发iOS系统的空间清理机制,最后再删除这个文件,这样就可以达到缓存清理的目的.
