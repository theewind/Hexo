layout: url
title: url encode的问题
date: 2016-08-18 19:39:46
tags: 
- iOS
- URL
---


URL参数encode的问题
===

外卖下单时，会先检测用户是否绑定了手机号，如果没有绑定，preview会提示进行绑定，跳转到绑定手机页面，如果绑定成功，就继续回到preview页面。

这就需要将preview的URLAction传递给绑定手机页面，它再进行一个透传。

<!--more-->

正确流程
---
这个过程就会有encode的问题，因为原则上，对URL里面的每一参数都需要记性encode，如果不含有特殊字符，encode之后结果不变，所以我们这里需要对参数
trolleyarr进行encode，因为他是一个类似1294711,1,0#4639,4688|1294710,2,0#4643,4647,4690|1294901,1,0这样的字符串，因此encode的结果是：

```
trolleyarr=1294711,1,0#4639,4688|1294710,2,0#4643,4647,4690|1294901,1,0

encode之后：

trolleyarr=1294711%2C1%2C0%234639%2C4688%7C1294710%2C2%2C0%234643%2C4647%2C4690%7C1294901%2C1%2C0
```
之后trolleyarr及lat，lng，shopid等组成一个重定向的URL，即redir，需要再对redir进行一次encode

```
redir=
dianping://takeawayorder?trolleyarr=1294711%2C1%2C0%234639%2C4688%7C1294710%2C2%2C0%234643%2C4647%2C4690%7C1294901%2C1%2C0%0A&initiallat=31.214027&initiallng=121.423747&shopid=2724613&mtwmpoiid=90939&mdcid=0&queryid=b15f2df1-a008-4e96-af45-5eaa3efe0110&phoneBinded=1

encode之后：

redir=
dianping%3A%2F%2Ftakeawayorder%3Ftrolleyarr%3D1294711%252C1%252C0%25234639%252C4688%257C1294710%252C2%252C0%25234643%252C4647%252C4690%257C1294901%252C1%252C0%250A%26initiallat%3D31.214027%26initiallng%3D121.423747%26shopid%3D2724613%26mtwmpoiid%3D90939%26mdcid%3D0%26queryid%3Db15f2df1-a008-4e96-af45-5eaa3efe0110%26phoneBinded%3D1%0A
```

而绑定手机号页面的整个URL是：

	NSString * const TABindPhoneURL = @"http://m.dianping.com/account/getBindPageAfterLogin?redir=%@&necessary=true";

redir作为其中的一个参数，需要继续进行一次encode，所以绑定页面的url为：

```
url = 
http://m.dianping.com/account/getBindPageAfterLogin?redir=dianping%3A%2F%2Ftakeawayorder%3Ftrolleyarr%3D1294711%252C1%252C0%25234639%252C4688%257C1294710%252C2%252C0%25234643%252C4647%252C4690%257C1294901%252C1%252C0%250A%26initiallat%3D31.214027%26initiallng%3D121.423747%26shopid%3D2724613%26mtwmpoiid%3D90939%26mdcid%3D0%26queryid%3Db15f2df1-a008-4e96-af45-5eaa3efe0110%26phoneBinded%3D1%0A&necessary=true

encode之后：

url = 
http%3A%2F%2Fm.dianping.com%2Faccount%2FgetBindPageAfterLogin%3Fredir%3Ddianping%253A%252F%252Ftakeawayorder%253Ftrolleyarr%253D1294711%25252C1%25252C0%2525234639%25252C4688%25257C1294710%25252C2%25252C0%2525234643%25252C4647%25252C4690%25257C1294901%25252C1%25252C0%25250A%2526initiallat%253D31.214027%2526initiallng%253D121.423747%2526shopid%253D2724613%2526mtwmpoiid%253D90939%2526mdcid%253D0%2526queryid%253Db15f2df1-a008-4e96-af45-5eaa3efe0110%2526phoneBinded%253D1%250A%26necessary%3Dtrue

```

dianping://takeawayorder?trolleyarr=1294711,1,0#4639,4688|1294710,2,0#4643,4647,4690|1294901,1,0&initiallat=31.214027&initiallng=121.423747&shopid=2724613&mtwmpoiid=90939&mdcid=0&queryid=b15f2df1-a008-4e96-af45-5eaa3efe0110&phoneBinded=1

到这里就告一段落，我们只需要通过：

	dianping://web?url=%@ 去打开绑定手机页面就可以了。


遇到问题
---
####iOS 坑一
iOS之前因为没有对trolleyarr的值进行decode，所以会出现绑定手机号之后，preview页面提示`“缺少必要”`的错误提示。这是因为服务端对传入的redir进行decode，这样redir就会变成类似：

	dianping://takeawayorder?trolleyarr=1294711,1,0#4639,4688|1294710,2,0#4643,4647,4690|1294901,1,0&initiallat=31.214027&initiallng=121.423747&shopid=2724613&mtwmpoiid=90939&mdcid=0&queryid=b15f2df1-a008-4e96-af45-5eaa3efe0110&phoneBinded=1
之后再把这样的url通过点评的openURL的方式打开，就会出现解析错误问题：
因为框架会对这样的url进行解析，其中NVURLAction的一部分代码是

```
- (id)initWithURL:(NSURL *)url {
	if (self = [super init]) {
        _url = url;
        
        NSDictionary *dic = [url parseQuery];
        _params = [NSMutableDictionary dictionary];
        for (NSString *key in [dic allKeys]) {
            id value = [dic objectForKey:key];
            [_params setObject:value forKey:[key lowercaseString]];
        }
	}
	return self;
}

NSURL的parseQuery方法如下：
- (NSDictionary *)parseQuery {
    NSString *query = [self query];
    NSMutableDictionary *dict = [[NSMutableDictionary alloc] initWithCapacity:6];
    NSArray *pairs = [query componentsSeparatedByString:@"&"];
    
    for (NSString *pair in pairs) {
        NSArray *elements = [pair componentsSeparatedByString:@"="];
		
		if ([elements count] <= 1) {
			continue;
		}
		
        NSString *key = [[elements objectAtIndex:0] stringByRemovingPercentEncoding];
        CFStringRef originValue = CFURLCreateStringByReplacingPercentEscapes(NULL, (CFStringRef)([elements objectAtIndex:1]),  CFSTR(""));
        NSString *oriValue = (__bridge NSString*)originValue;
        NSAssert(oriValue != nil, @"url is invalid");
        if (oriValue) {
            [dict setObject:oriValue forKey:key];
        }
        CFRelease(originValue);
    }
    return dict;
}

```
**NSURL的query方法，NSString *query = [self query]会直接截取`#`符号，因为#在url是一个合法字符**。所以就出现了URLAction的url实际变为了	

	dianping://takeawayorder?trolleyarr=1294711,1,0#
把这样的url丢给preview页面，结果就是上面的各种搞错，缺少各种必要参数。

#####如何解决这个问题？
原则上，对trolleyarr参数进行一次decode就应该可以的。这样返会到preview页面的url就是：

	dianping://takeawayorder?trolleyarr=1294711%2C1%2C0%234639%2C4688%7C1294710%2C2%2C0%234643%2C4647%2C4690%7C1294901%2C1%2C0%0A&initiallat=31.214027&initiallng=121.423747&shopid=2724613&mtwmpoiid=90939&mdcid=0&queryid=b15f2df1-a008-4e96-af45-5eaa3efe0110&phoneBinded=1

客户端自己对trolleyarr进行一次decode，获取正确的内容就可以了。满心欢喜，以为就此可以解决问题的，但是，但是。。。。

**其实外卖不需要再进行decode了，因为从NVURLAction取数据的时候，比如getString等，就是已经decode过的，因为`parseQuery`方法
**
####坑二
没有想到，服务端会对传入的redir进行`两次decode`,注意是两次，**(合理的操作应该是一次就可以了)**，导致上述的trolleyarr又被冲洗decode了，变成了类似*1294711,1,0#4639,4688|1294710,2,0#4643,4647,4690|1294901,1,0*的一坨东西，客户端继续出现问题。但神奇是，android竟然是正确的。他是怎么做到的呢。
可以看到android的URL是这样的

```
http://m.51ping.com/account/getBindPageAfterLogin?redir=dianping%3A%2F%2Ftakeawayorder%3Fshopid%3D2724613%26mtwmpoiid%3D90939%26mdcid%3D0%26queryid%3Dde133485-6bea-4a5e-a2fc-8532463af336%26initiallat%3D31.21756%26initiallng%3D121.41558%26carrier%3Dnull%26v%3D1%26cartcontents%3D%25255B%25257B%252522curPrice%252522%25253A1%25252C%252522selectedNum%252522%25253A1%25252C%252522name%252522%25253A%252522%252522%25252C%252522cartcontents%252522%25253A%2525221294901%25252C1%25252C0%252522%25257D%25252C%25257B%252522curPrice%252522%25253A10%25252C%252522selectedNum%252522%25253A1%25252C%252522name%252522%25253A%252522%252522%25252C%252522cartcontents%252522%25253A%2525221294711%25252C1%25252C0%2525234639%25252C4688%252522%25257D%25252C%25257B%252522curPrice%252522%25253A2.2%25252C%252522selectedNum%252522%25253A2%25252C%252522name%252522%25253A%252522%252522%25252C%252522cartcontents%252522%25253A%2525221294710%25252C2%25252C0%2525234643%25252C4647%25252C4690%252522%25257D%25255D&necessary=true&product=dpapp

不懂，decode一次：
http://m.51ping.com/account/getBindPageAfterLogin?redir=dianping://takeawayorder?shopid=2724613&mtwmpoiid=90939&mdcid=0&queryid=de133485-6bea-4a5e-a2fc-8532463af336&initiallat=31.21756&initiallng=121.41558&carrier=null&v=1&cartcontents=%255B%257B%2522curPrice%2522%253A1%252C%2522selectedNum%2522%253A1%252C%2522name%2522%253A%2522%2522%252C%2522cartcontents%2522%253A%25221294901%252C1%252C0%2522%257D%252C%257B%2522curPrice%2522%253A10%252C%2522selectedNum%2522%253A1%252C%2522name%2522%253A%2522%2522%252C%2522cartcontents%2522%253A%25221294711%252C1%252C0%25234639%252C4688%2522%257D%252C%257B%2522curPrice%2522%253A2.2%252C%2522selectedNum%2522%253A2%252C%2522name%2522%253A%2522%2522%252C%2522cartcontents%2522%253A%25221294710%252C2%252C0%25234643%252C4647%252C4690%2522%257D%255D&necessary=true&product=dpapp

还不懂，decode两次：
http://m.51ping.com/account/getBindPageAfterLogin?redir=dianping://takeawayorder?shopid=2724613&mtwmpoiid=90939&mdcid=0&queryid=de133485-6bea-4a5e-a2fc-8532463af336&initiallat=31.21756&initiallng=121.41558&carrier=null&v=1&cartcontents=%5B%7B%22curPrice%22%3A1%2C%22selectedNum%22%3A1%2C%22name%22%3A%22%22%2C%22cartcontents%22%3A%221294901%2C1%2C0%22%7D%2C%7B%22curPrice%22%3A10%2C%22selectedNum%22%3A1%2C%22name%22%3A%22%22%2C%22cartcontents%22%3A%221294711%2C1%2C0%234639%2C4688%22%7D%2C%7B%22curPrice%22%3A2.2%2C%22selectedNum%22%3A2%2C%22name%22%3A%22%22%2C%22cartcontents%22%3A%221294710%2C2%2C0%234643%2C4647%2C4690%22%7D%5D&necessary=true&product=dpapp

仍不懂，decode三次：
http://m.51ping.com/account/getBindPageAfterLogin?redir=dianping://takeawayorder?shopid=2724613&mtwmpoiid=90939&mdcid=0&queryid=de133485-6bea-4a5e-a2fc-8532463af336&initiallat=31.21756&initiallng=121.41558&carrier=null&v=1&cartcontents=[{"curPrice":1,"selectedNum":1,"name":"","cartcontents":"1294901,1,0"},{"curPrice":10,"selectedNum":1,"name":"","cartcontents":"1294711,1,0#4639,4688"},{"curPrice":2.2,"selectedNum":2,"name":"","cartcontents":"1294710,2,0#4643,4647,4690"}]&necessary=true&product=dpapp

原来他们对购物车的数据进行了两次decode，再加上redir，总共decode了三次，刚好服务端decode两次，自己再decode一次，妥妥的，完美/(ㄒoㄒ)/~~
```

再加上对dianping://web?url=http://m.51.......... 进行的一次decode，总共是四次，而iOS只有三次，所以iOS肯定有问题。闪开，让我哭一会~~~

####其它坑
* 为什么之前没有遇到这样的问题呢，一种说法是，之前的URL里面都没有`#`符号，因为没有选规格菜品。
* 为什么iOS会遇到userid==0，然后反复跳转的情况呢，因为H5从cookie中取到的newtoken为空，所以iOS有添加了一个newtoken=!的占位符。

* 在调试的过程中，会发现 po NVURLAction的时候，发现encode过的url会被decode，这是因为po object实际上是调用object的description方法，而NVURLAction重写了description方法，部分代码如下：

```
            } else if ([value isKindOfClass:[NSString class]]) {
                [paramsDesc addObject:[NSString stringWithFormat:@"%@=%@", [key lowercaseString], [value stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding]]];

```
打印的时候，输出decode的内容，所以如果URL有过encode，实际po的结果跟真实的内容是不一样的。

* NVWebViewController取出url的时候，会得到decode后的内容，如下：

[urlAction stringForKey:@"url"]; 而stringForKey是从_params里面取的。参上最上面的`parseQuery`方法，他会对url=之后的内容进行decode。这就是对应我们上面的最后一次encode了，即iOS的第三次，android的第四次，decode后，得到打开h5的URL，如：

```
http://m.dianping.com/account/getBindPageAfterLogin?redir=dianping%3A%2F%2Ftakeawayorder%3Ftrolleyarr%3D1294711%252C1%252C0%25234639%252C4688%257C1294710%252C2%252C0%25234643%252C4647%252C4690%257C1294901%252C1%252C0%26initiallat%3D31.214027%26initiallng%3D121.423747%26shopid%3D2724613%26mtwmpoiid%3D90939%26mdcid%3D0%26queryid%3Db15f2df1-a008-4e96-af45-5eaa3efe0110%26phoneBinded%3D1&necessary=true
```

* urlAction的url实际是下面，但是通过po出来查看，就是错误的：
_url	NSURL *	@"dianping://takeawayorder?trolleyarr=1294895%2C1%2C0%7C1294714%2C2%2C0%234489%7C1314261%2C1%2C0%7C1294711%2C1%2C0%234639%2C4688&initiallat=31.214027&initiallng=121.423747&shopid=2724613&mtwmpoiid=90939&mdcid=0&queryid=fc2ff9f9-ff00-4c78-ab95-04d278774ee4&phoneBinded=1"	0x00007fb02486a7d0
_clients	__NSCFString *	@"dianping://takeawayorder?trolleyarr=1294895%2C1%2C0%7C1294714%2C2%2C0%234489%7C1314261%2C1%2C0%7C1294711%2C1%2C0%234639%2C4688&initiallat=31.214027&initiallng=121.423747&shopid=2724613&mtwmpoiid=90939&mdcid=0&queryid=fc2ff9f9-ff00-4c78-ab95-04d278774ee4&phoneBinded=1"	0x00007fb024891ea0

(lldb) po urlAction
dianping://takeawayorder?shopid=2724613&trolleyarr=1294895,1,0%7C1294714,2,0%234489%7C1314261,1,0%7C1294711,1,0%234639,4688&mtwmpoiid=90939&queryid=fc2ff9f9-ff00-4c78-ab95-04d278774ee4&initiallat=31.214027&mdcid=0&initiallng=121.423747&phonebinded=1
