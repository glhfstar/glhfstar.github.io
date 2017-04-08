---
title: iOS 抽象工厂模式
date: 2016-04-08 21:04:25
tags:
---
抽象工厂模式提供了一种方式，可以将一组具有同一主题的单独的工厂封装起来。在正常使用中，客户端程序需要创建抽象工厂的具体实现，然后使用抽象工厂作为接口来创建这一主题的具体对象。客户端程序不需要知道（或关心）它从这些内部的工厂方法中获得对象的具体类型，因为客户端程序仅使用这些对象的通用接口。抽象工厂模式将一组对象的实现细节与他们的一般使用分离开来。

<font color=red>**iOS 类簇使用的便是抽象工厂的设计模式，像NSNumber, NSDictionary.例如在NSNumber的初始化伪代码大概像这样：**</font>

{% codeblock lang:objc %}
- (id)initWithBool
{
	return [[__NSCFBoolean alloc]init];
}

- (id)initWithLong
{
	return [[__NSCFNumber alloc]init];
}
{% endcodeblock %}

在iOS 开发中，我们可以利用这种设计模式将若干抽象产品的接口与不同主题产品的具体实现分离开。这样就能在增加新的具体工厂的时候，不用修改引用抽象工厂的客户端代码。
下面用一个例子来实现抽象工厂的模式：

{% codeblock lang:objc %}
//  KSStorage.h

#import <Foundation/Foundation.h>

typedef NS_ENUM(NSInteger, DataType) {
    FileStorageDataType,
    DatabaseStorageDataType,
};

@protocol KSStorageProtocol 
@required
    -(void)saveData;
    -(id)initWithData:(NSData *)data;
@end

@interface KSStorage : NSObject <KSStorageProtocol>
-(id)initWithData:(NSData *)data type:(DataType)dataType;
@end
{% endcodeblock %}

{% codeblock lang:objc %}
//  KSStorage.m

#import "KSStorage.h"

@interface FileStorage : KSStorage
@property (nonatomic,strong) NSURL  *storageURL;
@property (nonatomic,strong) NSData *data;

@end

@interface DatabaseStorage : KSStorage
@property (nonatomic,strong) NSDictionary  *dbConnectionInformationDictionary;
@property (nonatomic,strong) NSData *data;

@end

@implementation KSStorage

-(id)initWithData:(NSData *)data type:(DataType)dataType{
    self = nil;
    if (dataType == FileStorageDataType){
        self = [[FileStorage alloc] initWithData:data];
    }
    else if (dataType == DatabaseStorageDataType){
        self = [[DatabaseStorage alloc] initWithData:data];
    }
    return self;
}

-(void)saveData{
    [NSException raise:NSInternalInconsistencyException
                format:@"You have not implemented %@ in %@", NSStringFromSelector(_cmd), NSStringFromClass([self class])];
}

-(id)initWithData:(NSData *)data{
    [NSException raise:NSInternalInconsistencyException
                format:@"You have not implemented %@ in %@", NSStringFromSelector(_cmd), NSStringFromClass([self class])];
    return nil;
}

@end


@implementation FileStorage

-(id)initWithData:(NSData *)data{
    self.data = data;
    self.storageURL = [NSURL URLWithString:@"file:///Users/debasisdas/Knowstack/SampleData.txt"];
    return self;
}

-(void)saveData{
    NSLog(@"data is %@",self.data);
    NSLog(@"Now store it to stated URL %@",self.storageURL);
}
@end


@implementation DatabaseStorage

-(id)initWithData:(NSData *)data{
    self.data = data;
    self.dbConnectionInformationDictionary = @{@"connection":@"xyz.domain.com",@"port":@"8080",@"password":@"*******"};
    return self;
}

-(void)saveData
{
    NSLog(@"data is %@",self.data);
    NSLog(@"Now store it to stated URL %@",self.dbConnectionInformationDictionary);

}
@end
{% endcodeblock %}
测试代码：
{% codeblock lang:objc %}
    NSString *sampleString = @"This is a sample string";
    NSData *data = [sampleString dataUsingEncoding:NSUTF8StringEncoding];
    KSStorage *storage1 = [[KSStorage alloc] initWithData:data type:FileStorageDataType];
    NSLog(@"storage1 %@",storage1);
    if ([storage1 respondsToSelector:@selector(saveData)]){
        [storage1 saveData];
    }

    KSStorage *storage2 = [[KSStorage alloc] initWithData:data type:DatabaseStorageDataType];
    NSLog(@"storage2 %@",storage2);
    if ([storage2 respondsToSelector:@selector(saveData)]){
        [storage2 saveData];
    }
{% endcodeblock %}