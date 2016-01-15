---
layout: post
title: facebook ios sdk的使用
date: 2012-05-03 02:37
comments: true
categories: [facebook, ios, objective-c]
---
facebook ios sdk使用说明[facebook ios sdk](http://developers.facebook.com/docs/mobile/ios/build/#sdk)下载sdk以后吧src文件夹添加进自己的project即可

AppDelegate.m里面追加如下代码
{% highlight objective-c %}
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    return [self.viewController.facebook handleOpenURL:url];
}
{% endhighlight %}

facebook登录view的代码SAViewController.h
{% highlight objective-c %}
#import <UIKit/UIKit.h>
#import "FBConnect.h"

@interface SAViewController : UIViewController<FBRequestDelegate,FBDialogDelegate,FBSessionDelegate>
{
    Facebook* facebook;
    IBOutlet UIButton *facebookLoginButton;
    IBOutlet UIButton *emailLoginButton;
}
@property (nonatomic, retain) Facebook *facebook;
-(IBAction)loginWithFacebook:(id)sender;
-(IBAction)loginWithEmail:(id)sender;
@end
{% endhighlight %}

SAViewController.m里面增加以下代码
{% highlight objective-c %}
@implementation SAViewController

@synthesize facebook;

- (void)sagetUserDefaults
{
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    if ([defaults objectForKey:@"FBAccessTokenKey"]
        && [defaults objectForKey:@"FBExpirationDateKey"]
        && facebook) {
        facebook.accessToken = [defaults objectForKey:@"FBAccessTokenKey"];
        facebook.expirationDate = [defaults objectForKey:@"FBExpirationDateKey"];
    }
}

- (void)viewDidLoad
{
    [super viewDidLoad];
	// Do any additional setup after loading the view, typically from a nib.
    if (!facebook)
        facebook = [[Facebook alloc] initWithAppId:@"facebook app id" andDelegate:self];

    [self sagetUserDefaults];

    if ([facebook isSessionValid]) {
        facebookLoginButton.hidden = YES;
        emailLoginButton.hidden = YES;
    }
}

- (void)fbDidLogin
{
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    [defaults setObject:[facebook accessToken] forKey:@"FBAccessTokenKey"];
    [defaults setObject:[facebook expirationDate] forKey:@"FBExpirationDateKey"];
    [defaults synchronize];
    facebookLoginButton.hidden = YES;
    emailLoginButton.hidden = YES;
}

- (IBAction)loginWithFacebook:(id)sender
{
    if (!facebook)
        facebook = [[Facebook alloc] initWithAppId:@"facebook app id" andDelegate:self];

    [self sagetUserDefaults];

    if (![facebook isSessionValid]) {
        [facebook authorize:nil];
    }
}
{% endhighlight %}

最后编辑info.plist

点击login with facebook以后会通过facebook进行认证
