---
layout: post
title: "DynomaDB的ios sdk使用说明"
date: 2012-05-29 02:11
comments: true
categories: [objective-c, dynomadb, ios]
---
## 建立连接
连接dynomadb需要session token，session token需要通过token vending machine (TVM)来获得，另外一个获得token的方法是AmazonSecurityTokenServiceClient获得

这里介绍第二种方法
{% highlight objective-c %}
AmazonSecurityTokenServiceClient *tst = [[[AmazonSecurityTokenServiceClient alloc] initWithAccessKey:ACCESS_KEY_ID withSecretKey:SECRET_KEY] autorelease];
SecurityTokenServiceGetSessionTokenRequest *tokenRequest = [[[SecurityTokenServiceGetSessionTokenRequest alloc] init] autorelease];
SecurityTokenServiceGetSessionTokenResponse *rep = [tst getSessionToken:tokenRequest];
SecurityTokenServiceCredentials *c = [rep credentials];
AmazonCredentials *credentials = [[[AmazonCredentials alloc] initWithAccessKey:c.accessKeyId withSecretKey:c.secretAccessKey withSecurityToken:c.sessionToken] autorelease];
AmazonDynamoDBClient *ddb = [[[AmazonDynamoDBClient alloc] initWithCredentials:credentials] autorelease];
{% endhighlight %}

## 上传item
{% highlight objective-c %}
NSString *id = @"1";
NSString *userName = @"name";
DynamoDBAttributeValue *v1 = [[[DynamoDBAttributeValue alloc] initWithS:id] autorelease];
DynamoDBAttributeValue *v2 = [[[DynamoDBAttributeValue alloc] initWithS:userName] autorelease];
NSMutableDictionary *dic = [NSDictionary dictionaryWithObjectsAndKeys:v1, @"id", v2, @"name", nil];
DynamoDBPutItemRequest *putItemRequest = [[[DynamoDBPutItemRequest alloc] initWithTableName:TABLE_NAME andItem:dic] autorelease];
DynamoDBPutItemResponse *putItemResponse = [ddb putItem:putItemRequest];
{% endhighlight %}
DynamoDBAttributeValue有4个属性，分别是，n s ns ss,他们都是NSString或者Nsstring的set，但是代表不同的type  
n:number  
s:string  
ns:array of number  
ss:array of string

## 更新item
{% highlight objective-c %}
DynamoDBAttributeValue *attr = [[DynamoDBAttributeValue alloc] initWithS:aValue];
DynamoDBAttributeValueUpdate *attrUpdate = [[DynamoDBAttributeValueUpdate alloc] initWithValue:attr andAction:@"PUT"];
[attr release];
DynamoDBUpdateItemRequest *request = [[DynamoDBUpdateItemRequest alloc] initWithTableName:TABLE_NAME andKey:[[[DynamoDBKey alloc] initWithHashKeyElement:aPrimaryKey] autorelease] andAttributeUpdates:[NSMutableDictionary dictionaryWithObject: attrUpdate forKey:aKey]];
[attrUpdate release];
[[AmazonClientManager ddb] updateItem:request];
[request release];
{% endhighlight %}

## 删除item
{% highlight objective-c %}
DynamoDBDeleteItemRequest *request = [[DynamoDBDeleteItemRequest alloc] initWithTableName:TABLE_NAME andKey:[[[DynamoDBKey alloc] initWithHashKeyElement:aPrimaryKey] autorelease]];
[ddb deleteItem:request];
[request release];
{% endhighlight %}

## qurey
{% highlight objective-c %}
DynamoDBAttributeValue *v1 = [[[DynamoDBAttributeValue alloc] initWithN:@"1"] autorelease];
DynamoDBQueryRequest *queryRequest = [[[DynamoDBQueryRequest alloc] initWithTableName:TABLE_NAME andHashKeyValue:v1] autorelease];
[queryRequest addAttributesToGet:@"name"];
DynamoDBQueryResponse *queryResponse = [ddb query:queryRequest];
{% endhighlight %}
这段代码出现error，有待解决

## scan
{% highlight objective-c %}
DynamoDBScanRequest *request = [[[DynamoDBScanRequest alloc] initWithTableName:TABLE_NAME] autorelease];
DynamoDBScanResponse *response = [[AmazonClientManager ddb] scan:request];
NSMutableArray *users = response.items;
{% endhighlight %}
