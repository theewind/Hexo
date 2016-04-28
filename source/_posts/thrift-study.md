---
title: thrift学习
date: 2016-04-29 17:44:20
tabs: thrift
---

**其实thrift和点评的nvobject蛮像的，其中，TBinaryProtocol都是二进制传输，然后将属性的字段进行压缩，较少网络传输的数据。
比如点评是采用hash值，每个对象的属性都有个hash值，然后传输此hash值，客户端根据hash值对应属性，而thrift是采用编号的方式，在idl描述文件中，就已经设置好了编号，然后传输编号到服务端，客户端根据编号对应属性。
**

**thrift -gen cocoa ******.thrift 生成.h.m文件，然后添加工程，然后调用方法如下：**

<!--more-->

##例如：
METMemberCardServiceClient是最终发送请求的类，而他依赖TBinaryProtocol<TPortocol>


	- (void)getDisplayTypefinished:(void(^)(METMineMembercardExhibitionType result, SAKError *error))finished
	{
    NSDictionary *headerParams = @{@"token" : kUserToken, @"userId" : kUserID, @"clientKey" :  @"group"};
    TBinaryProtocol *binaryProtocol = [self createThriftProtocol:kMemberCardServiceURLString headerParameters:headerParams];
    METMemberCardServiceClient *memberCardService = [[METMemberCardServiceClient alloc] initWithProtocol:binaryProtocol];
    [memberCardService getDisplayTypefinished:^(int result, SAKError *error) {
        if (error) {
            if (finished) {
                @within_main_thread(finished, 0, error);
            }
            return;
        }
        if (finished) {
            @within_main_thread(finished, result, error);
        }
    }];
	}

TBinaryProtocol的构造需要SAKMascoTransport<TTransport>，而SAKMascoTransport<TTransport>的构造需要：THTTPClient<TTransport>，其中SAKMascoTransport是对THTTPClient的封装。
THTTPClient的概念比较简单，主要是read，write及flush（发送网络请求），不管数据的意义，而SAKMascoTransport是有数据概念的，比如header，body，元数据等，将有意义的数据进行组合后，然后掉用THTTPClient的read，write等。

	- (TBinaryProtocol *)createThriftProtocol:(NSString *)serviceURLString headerParameters:(NSDictionary *)headerParameters
	{
    NSString *commonParameterString = [[[SAKEnvironment environment] commonParameter] queryStringEncoded];
    NSString *serviceURLStringWithCommonParameter = [NSString stringWithFormat:@"%@?%@", serviceURLString, commonParameterString];
    THTTPClient *httpClient = [[THTTPClient alloc] initWithURL:[NSURL URLWithString:serviceURLStringWithCommonParameter]];
    SAKMascoTransport *mascoTransport = [[SAKMascoTransport alloc] initWithTransport:httpClient andSerialID:1];
    
    [headerParameters enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
        NSString *keyString = (NSString *)key;
        if (keyString.length) {
            [mascoTransport appendHeaderData:[keyString dataUsingEncoding:NSUTF8StringEncoding]];
        }
        
        // Replace user id and user token
        NSString *valueString = (NSString *)obj;
        if ([valueString isEqualToString:kUserID]) {
            valueString = [valueString stringByReplacingOccurrencesOfString:kUserID withString:(self.user ? [self.user.userID stringValue] : @"")];
        }
        else if ([valueString isEqualToString:kUserToken]) {
            valueString = [valueString stringByReplacingOccurrencesOfString:kUserToken withString:(self.user ? self.user.token : @"")];
        } else {
            // Do nothing
        }
        
        if (valueString.length) {
            [mascoTransport appendHeaderData:[valueString dataUsingEncoding:NSUTF8StringEncoding]];
        }
    }];
    
    	TBinaryProtocol *customProtocol = [[TBinaryProtocol alloc] initWithTransport:mascoTransport strictRead:NO strictWrite:YES];
	    return customProtocol;
	}