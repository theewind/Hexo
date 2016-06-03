---
title: iOS下载内容保存在document中需要设置不同步
date: 2016-05-26 11:26:03
tags: 
- iOS
- 缓存
---

之前因为在document，保存了下载数据，导致app审核不通过。这个苹果是严格要求，网络下载的数据一般是不能保存在document目录中，建议是保存在cache中，如果真的需要保存在document中，就需要对保存的内容设置不同步标识。具体代码如下：

```
- (BOOL)addSkipBackupAttributeToItemAtURL:(NSURL *)URL
{
    if (![[NSFileManager defaultManager] fileExistsAtPath:[URL path]])
    {
        return NO;
    }
    
    BOOL success = NO;
    NSError *error = nil;
    success = [URL setResourceValue:[NSNumber numberWithBool: YES]
                             forKey:NSURLIsExcludedFromBackupKey error: &error];
    return success;
}
```
