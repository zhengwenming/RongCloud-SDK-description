# RongCloud-SDK-description

# 欢迎关注我的斗鱼直播间，用手机斗鱼TV，直接搜索`文明直播间`或者`极端恐惧`就可以找到我的直播。`iOS技术分享`直播。进来点一下关注，谢谢，开播会有推送到大家手机。（个人直播，非机构，适合`初级iOS`和`中级iOS`）。

# 欢迎加入`iOS开发技术支持群`，`479259423`，进群必须改名（2元付费群，慎入！），群名片格式：`城市-iOS-名字`，例如`广州-iOS-文明`。

下面几篇文章分别详细的介绍了融云`SDK`即时通讯机制和集成步骤，由于国内`CSDN`博客封杀带有广告性质的文章（其实不是打广告，纯粹的技术分享），所以只能在`Github`发表了。希望大家支持我，谢谢。

# 融入`Voip`功能的提示：

现在的新`SDK`用不了`Voip`了，已经分离出一个`CallKit`进行收费。如果真的要用`voip`，智能用`2.5.2版本的sdk`。下面是引用融云官方的文档原话

由于底层引擎技术不同，`2.6.0` 之后的音视频 `SDK` 与 `2.6.0` 之前的 `SDK` 中的 `VoIP` 不能互通。
音视频 `SDK` 为商用收费功能，之前的 `SDK` 中的 `VoIP` 为免费测试功能，如果您还想使用之前的 `VoIP`，可以使用 `2.5.2` 版本。
 
 
# 说说融云即时通讯`SDK`理论篇(一) 

>标签： `即时通讯` `聊天` `融云` `sdk` `iOS`

>分类：`即时通讯`

>版权声明：本文为博主原创文章，未经博主允许不得转载。

  本人用过融云的`1.4.4`和`2.2.4`版本的`SDK`，分别用于对两款`APP`的即时通讯功能开发集成，主要用了`IMKit`类，功能为`单聊`（聊天室和群聊不涉及，理解了单聊，聊天室和群聊也就不攻而破）,吐槽一下关于`iOS`端融云`SDK`集成资料很少，只能看融云官方的开发文档，由于开发文档的语言和关键词用的也比较官方比较专业化，我想要通俗易通，雅俗共赏的资料，没有！没有！真的没有！博客之类的资料只能找到安卓的文章，无奈只能`iOS`小屌丝看安卓大神写的关于融云集成的android代码，小伙伴们看的是很蛋疼。现在福利来了，`iOS`端的融云集成博客这边看，会不断更新，持续关注。想集成融云聊天功能的小伙伴不防先普及下理论知识，防止以后走弯路又或者总是问些不靠谱的问题，`我为什么不能聊天`，`为什么头像和名字出不来` 等等一系列问题，文章中如果有错误的地方希望大家提出，本人会及时改正，避免误人子弟，高手勿喷，哈哈。。。

  融云的`SDK`可以解决很多`app`中即时通讯的功能，无论是安卓还是`iOS`（`web`也可以，强大无比），都可以考虑用第三方去实现聊天的功能，而且现在`app`中集成聊天的功能很火很时髦，不是吗？收废品、旧家电的老板都恨不得为自己的`APP`加上即时通讯翅膀，让自己的`app`飞起来。废话少说，先来理解下融云的即时通讯机制，也就是理论部分。

   融云不维护好友关系，只负责转发消息。不维护好友关系是什么意思？我用融云的`SDK`集成聊天，好友之间都可以进行聊天啊，怎么会不维护好友关系呢！？那么我们来理解下： `不维护好友关系，只负责转发消息` 的意思是融云的服务器给我们转发消息，融云的`SDK`完成发送消息的功能（`SDK`中的方法实现，`web后台`也可以调用融云提供的接口发送消息），消息从一个人`fromUserId`（每个人都有一个id，后面会详细讲到）转发到另一个人`toUserId`那里，那么每个用户的好友关系和群组关系是要保存在自己公司的服务器的，这点就解释了上面的话，融云提供一个转发消息的平台（服务器和SDK以及接口等），因为要满足广大开发者的需求，所以不可能为没个开发者都维护这个好友关系和群组关系。举个栗子，`用户A`有100个好友，那么这个融云是不知道的，也不会关心`用户A`的好友关系，谁知道呢？我们开发者自己的后台要知道，我们自己的服务器要维护好友关系和群组关系，具体做法就是要用接口去建立好友关系，然后再用另外的一个接口去保存`用户A`的所有好友。这样的话，`用户A`登录之后就可以调用好友列表的接口拿到自己的所有好友，然后就可以为所欲为了，其实也就是和自己的好友发送消息了，发送消息的时候，不是走自己公司的服务器发送消息，注意，这里发送消息是走融云的服务器，怎么会走融云的服务器呢？奇怪，融云怎么知道我登录了呢？别急，下面会讲到`token`，以及`connect`连接融云服务器。消息的流向是这样的，从自己这里`fromUserId`（自己的`id`）走到融云的服务器，然后融云服务器转发到好友`toUserId`那里，因为消息体中含有很多的属性，消息体本身可以附带信息，比如`targetId`（就是`toUserId`，这里两者一样），`extra`附加信息，用于提供给开发者拓展功能，你想让消息带什么信息都可以带，但是大小有限制，一般带`字符串`或`json`数据，我用这个字段解决了好友头像的及时更新功能，这个字段的用法放开发篇章中细细讲解。

   现在说说`userId`，用户的 `身份证号码` 。每个`APP`的注册用户都会被分配一个`userId`，就是用来区分用户的，就好像我们的身份证号码一样，不会重复，具有唯一性。那么再说说谁去分配`用户A`的`userId`呢？融云会分配吗？答案是`NO`。还是那句话融云只负责转发消息，分配`userId`这个任务还是要我们后台自己来搞，当`用户A`注册的时候，后台分配给`A`一个`userId`并且返回给前端开发人员使用，前端开发人员拿到`A`的`userId`之后，就要用这个A用户的唯一的`userId`去登录融云的服务器，当然不是任何人有个userId都可以随便连接融云的服务器的哦，大家都知道任何第三方公司都没有那么随便，不安全，会被攻击，是要`Token`验证的。有这个`Token`就让你连接，没有`token`的话，滚蛋。那么我们写`demo`，测试的话怎么办，`token`怎么获取？去融云强大的后台调试系统里面去拿token测试，登录融云的官网之后点击你创建的应用，然后找到IM服务中的API调试，第一个就是获取`token`。整个登录的流程是这样滴，开发者去融云官网注册个账号---->`登录融云官网`---->`创建一个应用`---->`拿到融云分配给此应用的appkey`---->`app中代码先注册融云的appKey`---->`获取token获取token`---->`拿token去connect融云服务器`。大概就是这样，但是开发过程中要注意以下几点：

 （一）`appkey`包括正式上线的 `appkey` 和开发环境的 `appkey`。现在的新 SDK 取消用 `appSecret`，我们代码中用的是`appkey`，不是`appSecret`，所以暂时不要管`appSecret`，但是不要随便刷新这个秘钥，后果自负。

 （二）`token`怎么获取，哪里去获取？`token`的获取是要自己公司的后台提供一个获取融云`token`的专门接口，自己的后台开发人员去看文档做接口，后台相关事情这里不做解答（一句话：去看融云开发文档）。获取到`token`后，就可以拿一个`用户A`的`userId`去连接融云的服务器了，融云的方法叫`connectWithToken` `connect` 成功了之后会返回登录人的`userId`，失败了会返回`error`的描述（一个枚举，详细的解释`connect`失败的原因）。具体看下面代码，注意，我的做法是每次都获取`token`，还有其他的处理`token`方式，具体的看融云开发指南，我这里提供一种解决方案。
 （三）集成的步骤不要错了，一定要按照顺序。我举个栗子:有些开发者把项目用`StoryBoard`搭建起来之后，就马上想用聊天列表，想看到聊天列表，想看看聊天界面，这样有时候会崩溃，就跑来问我，怎么回事？我也很难回答你是怎么一回事。所以大家先把项目建立起来，然后按照步骤，一步一步的来，你都没注册`appkey`，也没有拿`token`，也没有`connect`。那么这样的崩溃让融云的技术或者是我去解决这样的`bug`，其实我们的内心也是崩溃的。

这句代码是注册`appkey`的，先看看，到开发篇里面还会有 `[[RCIM sharedRCIM] initWithAppKey:@"YourTestAppKey"];`
下面的`blok`是用`token`去链接融云服务器。开发篇还会再提供，先初步了解下。
```Object-C
 [[RCIMsharedRCIM]connectWithToken:YourToken success:^(NSString *userId) {

                    [RCIMsharedRCIM].globalNavigationBarTintColor = [UIColorwhiteColor];

                    NSLog(@"login success with userId %@",userId);

                    [UserInfoModel loginUserinfo:model];

                    //同步好友列表

                    [self syncFriendList:^(NSMutableArray *friends,BOOL isSuccess) {

                        NSLog(@"%@",friends);

                        if (isSuccess) {

                            NSLog(@" success发送通知");

                            [[NSNotificationCenterdefaultCenter]postNotificationName:@"alreadyLogin" object:nil];//发送自定义通知（不是融云的），处理一些逻辑，比如什么各单位注意，我已登录，我已登录。

                        }

                    }];

                    [RCIMClient sharedRCIMClient].currentUserInfo = [[RCUserInfoalloc]initWithUserId:userinfo.userIdname:userinfo.userNameportrait:userinfo.photophone:userinfo.phoneaddressInfo:userinfo.companyrealName:userinfo.realName];

                   

                } error:^(RCConnectErrorCode status) {

                    NSLog(@"status = %ld",(long)status);

                } tokenIncorrect:^{

                    self.success =NO;

                    [MSUtil showTipsWithHUD:@"tokenIncorrect"];

                }];

```

>连接成功了，继续看。要想和好友聊天，我们还要有好友，有了好友才能正常聊天，我的好友是保存在一个全局的数组里面的，app启动的时候，我网络请求自己的后台，拿到我们后台维护的好友数组，然后放数组里面，这样我全局可以拿到。另外，还要实现一个代理方法，这个代理叫做`RCIMUserInfoDataSource`（`2.2.4版本`的叫`RCIMUserInfoDataSource`，以前的老版本比较麻烦些，要实现两个代理`RCIMFriendsFetcherDelegate,RCIMUserInfoFetcherDelegagte`）这个代理方法是提供好友信息的，给谁提供好友信息？你和你的好友聊天，系统会自动走代理方法，去拿好友信息，所以那些说“我的好友头像和名字怎么出不来”的同学要注意了，遵守代理了吗？是不是你的代理没设置，是不是没实现代理方法，方法里面有没有问题，有没有配置好你的好友信息？不然好友头像和名字怎么会出不来呢！附上代理方法名字和实现

```Object-C
- (void)getUserInfoWithUserId:(NSString*)userId completion:(void (^)(RCUserInfo*))completion

{

    NSLog(@"getUserInfoWithUserId ----- %@", userId);

    

    if (userId == nil || [userId length] == 0 )

    {

        completion(nil);

        return ;

    }

   

    if ([userIdisEqualToString:[RCIMsharedRCIM].currentUserInfo.userId]) {//自己

        UserInfoModel *myselfInfo = [UserInfoModelcurrentUserinfo];

        RCUserInfo *aUser = [[RCUserInfoalloc]initWithUserId:myselfInfo.userIdname:myselfInfo.realNameportrait:myselfInfo.photophone:myselfInfo.phoneaddressInfo:myselfInfo.companyrealName:myselfInfo.realName];

        completion(aUser);

    }

    

    for (NSInteger i =0; i<[AppDelegateshareAppDelegate].friendsArray.count; i++) {//循环全局的好友数组，拿到userId相同的，比如，userId＝ 12的是老王，那么循环，找到数组中userId＝12的，那么肯定就是老王啦。那么肯定就配置好老王的信息了，和老王聊天的时候，融云内部封装的方法就可以拿到老王的信息，这样老王的头像和名字就可以出来了

        RCUserInfo *aUser = [AppDelegateshareAppDelegate].friendsArray[i];

        if ([userId isEqualToString:aUser.userId]) {

            completion(aUser);

        }

    }

}
```



最后开始发消息给好友，比如点击一个`cell`（或者一个页面的`Button`），拿到这个`cell`对应人的`userId`，拿到对方的`userId`后，就可以建立一个会话了，那么开始上代码
```Object-C
ConversationViewController *_conversationVC = [[ConversationViewControlleralloc]init];

_conversationVC.conversationType = ConversationType_PRIVATE;//单聊（也叫私聊），会话类型

_conversationVC.targetId = [NSStringstringWithFormat:@"%@",model.data[@"id"]];//消息的目标id，对方的userId，就是上文说的toUserId

_conversationVC.userName = [NSStringstringWithFormat:@"%@",model.data[@"agentTeamName"]];//会话对方的人名字

_conversationVC.title = [NSStringstringWithFormat:@"%@",model.data[@"realName"]];//会话导航上的title

[self.navigationControllerpushViewController:_conversationVCanimated:YES];
```


`ConversationViewController`是开发者自己建立的`VC`，继承融云的`RCConversationViewController`类，继承之后，子类就可以用重写父类的方法了，想用什么就点进去找什么方法，如果你也不知道你需要什么方法，那么就跟着我的文章走，我将会介绍大部分融云API的使用场景。

好了，现在可以发送消息了，因为`RCConversationViewController`类已经实现了`UI`和`功能`。



理论篇到此结束。下面会有开发篇，谢谢。有问题在我的开发群里提问，群号`487599875`.


>－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－
－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－


 说说融云即时通讯`SDK`开发篇(二) 



理论知识理解之后，就可以快速集成了。有多快？常常有开发者问，一天可以集成融云的聊天功能吗？我想说我现在一天都搞不定，你不用处理逻辑的吗？你仅仅是写个`demo`还是做项目啊？如果是写个`demo`，我`1个小时`可以集成`聊天功能`了。所以大家还是要按部就班的来，欲速测不达。

### 好了，不扯淡了，开始集成。

>一、去[融云官网](http://www.rongcloud.cn)下载`SDK`。补充一句最好用官网最新的`SDK`，千万不要倒退啊，最新的`SDK`比较完善，把以前旧版本的`SDK`的`bug`都修复了，当然肯定还有`bug`存在，但是已经比较少了，而且你也不一定就踩到雷了，如果踩到了，去给融云发工单，每个工单都会得到回答，大部分会令你满意的，不满意也没办法。好了，下载完之后找到`SDK`的文件夹，打开，然后自己在自己工程中建立一个新的文件夹比如我的就是`RongCloud_SDK_2_2_4`，这样清晰明了的看到了作用和版本号，再然后就是拖SDK进来工程中的文件夹`RongCloud_SDK_2_2_4`，除去`release_notes_ios.txt`不用拖进来，其他的都拖进来，不然你就拖了两个`frameWork`，后面开发中肯定你会叫，为什么我的表情😊出不来，为什么我的是英文的，各种奇葩`bug`出现了，原因就在此，你要把`emoji`的`plist`文件和语言国际化的东东也一起拉进工程。记得点击`copy`选项。

>二、添加frameWork，而且要全面。在第一步的基础上添加融云SDK的依赖库，都是系统的，官网有库列表，慢慢加。

`AssetsLibrary.framework`

`AudioToolbox.framework`

`AVFoundation.framework`

`CFNetwork.framework`

`CoreAudio.framework`

`CoreGraphics.framework`

`CoreLocation.framework`

`CoreMedia.framework`

`CoreTelephony.framework`

`CoreVideo.framework`

`ImageIO.framework`

`libc++.tbd`

`libc++abi.tbd`

`libsqlite3.tbd`

`libstdc++.tbd`

`libxml2.tbd`

`libz.tbd`

`MapKit.framework`

`OpenGLES.framework`

`QuartzCore.framework`

`SystemConfiguration.framework`

`UIKit.framework`

>三、到工程的设置选择中，系统setting，搜索到Other linker Flags 双击这一行，填写-ObjC,主意O和C是大写的，大小写敏感。（不加这个flag好像发消息会崩溃）

>四、导入头文件

`#import <RongIMKit/RongIMKit.h>`
`import <RongIMKit/RongIMKit.h>`

  这个一般应该是在`appdelegate`里面倒入，因为程序入口，我们要初始化融云的appkey（理论篇有详细介绍，开发篇不再赘述）。
在这个方法中初始化
`- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`  最好写一个方法专门放置融云的逻辑，`initRongCloud`方法放置融云的初始化
```Object-C
- (void)initRongCloud{

   // 这个好友数组friendArray，我放到appdelegate的.h中声名，程序任何地方可以通过appdelegate这个单例拿到，想哪里用就可以在哪里用。这个数组是存放我的全部好友信息的，里面放的是RCUserInfo，每个好友都是RCUserInfo的一个实例对象。
   self.friendsArray = [[NSMutableArray alloc]init];

    //测试环境          AppKey XXXXXAppKey      AppSecret XXXXXXAppSecret

    //正式上线环境       AppKey XXXXXAppKey      AppSecret  XXXXXAppKey

    NSString *rongYunKey = @"yourAPPKey";
//判断用正式appkey还是测试appkey，测试开发环境最好不要用正式的key哦，不要怪我没提醒。
    if ([kNetwork_Host isEqualToString:@"http://weixintest.ihk.cn"]) {//正式环境用正式key，开发测试环境用测试的key

        rongYunKey = @"your 正式appkey";

    }else{
            rongYunKey = @"your 测试appkey";

    }
//初始化appkey
    [[RCIM sharedRCIM] initWithAppKey:rongYunKey];
//初始化全局的单例RCDataManager（融云数据管理者），这个RCDataManager把所以融云有关数据的逻辑和代码分离了，方便该，也方便维护。这里把userInfoDatasource设置为RCDataManager。
    [RCIM sharedRCIM].userInfoDataSource = [RCDataManager shareManager];

/**enableMessageAttachUserInfo
 *  默认NO，如果YES，发送消息会包含自己用户信息。
 */


    [RCIM sharedRCIM].enableMessageAttachUserInfo = YES;


//登录融云（注意这里是第n次登录，n>1，第一次登录的逻辑在登录页面点击登录按钮的时候，一样调用这句话，因为逻辑全交给RCDataManager处理了，外部调用方法即可）
    [[RCDataManager shareManager] loginRongCloud];

}
```
  初始化完成后就是拿`token`，然后用一个人（当前用户啊）的`userId`去`connect`了，那么这个逻辑我没有把代码写在`appdelegate`，因为业务相同的逻辑应该分离出来，要不全部代码写`appdelegate`里面太乱，太难维护，我们把有关融云的逻辑抽离出来，放一个类`RCDataManager`中去管理，那么我的这个`RCDataManager`完成了什么功能，什么逻辑，具体怎么设计这个类，为什么要这样设计。下面详细介绍下怎么把融云的逻辑模块分离出来，写出高一点质量的代码。
`RCDataManager`是个单例类，主要功能就是登录融云，刷新好友列表，设置好友信息提供者代理等等，设置`tabbar`的角标等等。他的好处是把融云的逻辑分离出来，业务逻辑分离，方便维护工程，我们用的时候就一句代码就可以了，比如登录就这样 `[[RCDataManager shareManager] loginRongCloud];`具体`RCDataManager`类的代码如下：
```Objcect-C
@interface RCDataManager : NSObject<RCIMUserInfoDataSource>

RCIMUserInfoDataSource是融云的好友提供者代理，很重要，非常重要，及其重要，遵守之，实现之。





@property(nonatomic,assign)BOOL success;

+(RCDataManager *) shareManager;

- (void)getUserInfoWithUserId:(NSString*)userId completion:(void (^)(RCUserInfo*))completion;

/**

 *  从服务器同步好友列表

 */

-(BOOL)hasTheFriendWithUserId:(NSString *)userId;

-(void)loginRongCloud;
//同步好友列表的方法，用此方法可以刷新到最新的好友列表，比如有新朋友了，那么就要同步一下
-(void) syncFriendList:(void (^)(NSMutableArray * friends,BOOL isSuccess))completion;

-(void)refreshBadgeValue;

-(NSString *)currentNameWithUserId:(NSString *)userId;

-(RCUserInfo *)currentUserInfoWithUserId:(NSString *)userId;

@end

```


>再看实现文件里面的代码

```Object-C
#import "RCDataManager.h"



@implementation RCDataManager{

    NSMutableArray *dataSoure;



}

- (instancetype)init{

    if (self = [super init]) {

        [RCIM sharedRCIM].userInfoDataSource = self;

        dataSoure = [[NSMutableArray alloc]init];

    }

    return self;

}



+ (RCDataManager *)shareManager{

    static RCDataManager* manager = nil;

    static dispatch_once_t predicate;

    dispatch_once(&predicate, ^{

        manager = [[[self class] alloc] init];

    });

    return manager;

}



-(void)syncFriendList:(void (^)(NSMutableArray* friends,BOOL isSuccess))completion

{
//这里是用户的身份，可能其他开发者不需要这个逻辑，不需要的话，就直接调用好友列表的接口，拿到好友列表的数组，做成RCUserInfo，然后add进去数组。就这简单！
    UsersType type = [MSUtil checkUserType];

    if (type==UsersTypeYouke) {//游客

    //不能聊天


    }

    else if(type==UsersTypeSales){//销售

        /* 获取我的客户(包括销售) */

        [Networking getMyClientsWithPage:@"1" pagesize:@"10000" success:^(NetworkModel *model) {

            NSLog(@"%@",model.msg);



            if ([model.data isKindOfClass:[NSString class]]) {

                //LOGIN_CHECK_NO

                NSLog(@"%@",model.data);

            }else if ([model.data isKindOfClass:[NSDictionary class]]){

                [dataSoure removeAllObjects];

            

                for (NSDictionary *infoDic in model.data[@"rows"]) {

                    RCUserInfo *userInfo = [[RCUserInfo alloc]init];

                    userInfo.userId = [NSString stringWithFormat:@"%@",infoDic[@"id"]];

//                    if ([[NSString stringWithFormat:@"%@",infoDic[@"salesNickName"]] isEqualToString:@""]) {

//                        userInfo.name = [NSString stringWithFormat:@"%@",infoDic[@"realName"]];

//                    }else{

//                        userInfo.name = [NSString stringWithFormat:@"%@  %@",infoDic[@"realName"],infoDic[@"salesNickName"]];

//                    }



                    userInfo.name = [[NSString stringWithFormat:@"%@",infoDic[@"realName"]] isEqualToString:@""]?[NSString stringWithFormat:@"%@",infoDic[@"loginName"]]:[NSStringstringWithFormat:@"%@",infoDic[@"realName"]];

                    userInfo.portraitUri = [NSString stringWithFormat:@"%@",infoDic[@"photo"]];

                    userInfo.phone = [NSString stringWithFormat:@"%@",infoDic[@"phone"]];

                    userInfo.addressInfo = [NSString stringWithFormat:@"%@",infoDic[@"company"]];

                    userInfo.realName = [NSString stringWithFormat:@"%@",infoDic[@"realName"]];



                    [dataSoure addObject:userInfo];

                    

                }

                [AppDelegate shareAppDelegate].friendsArray = dataSoure;

                completion(model.data[@"rows"],[model.result isEqualToString:@"10000"]?YES:NO);



                NSLog(@"好友列表 = %@",[AppDelegate shareAppDelegate].friendsArray);

                if ([AppDelegate shareAppDelegate].friendsArray.count) {

                    NSLog(@"从服务器同步好友列表成功FFF");

                    [self loginRongCloud];

                    for (RCUserInfo *aUser in [AppDelegate shareAppDelegate].friendsArray) {

                        NSLog(@"name =%@ userId =%@",aUser.name ,aUser.userId);

                    }

                }

            }

            

        } fail:^(NSError *error) {

            NSLog(@"error＝ %@",error);

            NSLog(@"从服务器同步好友列表失败 ～～～");



        }];

    }else if(type==UsersTypeCustomer){//用户

        [Networking getMyBrokerAppUsersWithPage:@"1" pagesize:@"10000" success:^(NetworkModel *model) {

            if ([model.data isKindOfClass:[NSString class]]) {

                

            }else if ([model.data isKindOfClass:[NSDictionary class]]){

                [dataSoure removeAllObjects];

    

                for (NSDictionary *infoDic in model.data[@"rows"]) {

                    

                    RCUserInfo *userInfo = [[RCUserInfo alloc]init];

                    userInfo.userId = [NSString stringWithFormat:@"%@",infoDic[@"id"]];

                    

                    userInfo.name = [[NSString stringWithFormat:@"%@",infoDic[@"salesNickName"]] isEqualToString:@""]?[NSString stringWithFormat:@"%@",infoDic[@"realName"]]:[NSStringstringWithFormat:@"%@",infoDic[@"salesNickName"]];

                    userInfo.portraitUri = [NSString stringWithFormat:@"%@",infoDic[@"photo"]];

                    userInfo.phone = [NSString stringWithFormat:@"%@",infoDic[@"phone"]];

                    userInfo.addressInfo = [NSString stringWithFormat:@"%@",infoDic[@"company"]];

                    userInfo.realName = [NSString stringWithFormat:@"%@",infoDic[@"realName"]];



                    [dataSoure addObject:userInfo];



                }

                [AppDelegate shareAppDelegate].friendsArray = dataSoure;

                NSLog(@"好友列表 = %@",[AppDelegate shareAppDelegate].friendsArray);

                completion(model.data[@"rows"],[model.result isEqualToString:@"10000"]?YES:NO);



                if ([AppDelegate shareAppDelegate].friendsArray.count) {

                    NSLog(@"从服务器同步好友列表成功FFF");

                    if ([RCIMClient sharedRCIMClient].currentUserInfo.userId) {

                        

                    }else{

                        [self loginRongCloud];



                    }

                    for (RCUserInfo *aUser in [AppDelegate shareAppDelegate].friendsArray) {

                        NSLog(@"name = %@",aUser.name);

                        NSLog(@"name = %@",aUser.userId);

                    }

                }

            }

        } fail:^(NSError *error) {

            NSLog(@"error ＝ %@",error);

            NSLog(@"从服务器同步好友列表失败 ～～～");





        }];

    }

}

//获取一个userId对应的好友对象，这个对象我们叫做RCUserInfo，这个是融云的类，我们看看源代码的.h里面怎么写

-(RCUserInfo *)currentUserInfoWithUserId:(NSString *)userId{

    for (NSInteger i = 0; i<[AppDelegate shareAppDelegate].friendsArray.count; i++) {

        RCUserInfo *aUser = [AppDelegate shareAppDelegate].friendsArray[i];

        if ([userId isEqualToString:aUser.userId]) {

            NSLog(@"current ＝ %@",aUser.name);

            return aUser;

        }

    }

    return nil;

}

这是RCUserInfo的.h
/**

 *  用户信息类

 */

@interface RCUserInfo : NSObject <NSCoding>

/** 用户ID */

@property(nonatomic, strong) NSString *userId;

/** 用户名*/

@property(nonatomic, strong) NSString *name;

/** 头像URL*/

@property(nonatomic, strong) NSString *portraitUri;



/**

 *  指派的初始化方法，根据给定字段初始化实例

 *

 *  @param userId       用户ID

 *  @param username     用户名

 *  @param portrait     头像URL

 */
- (instancetype)initWithUserId:(NSString *)userId name:(NSString *)username portrait:(NSString *)portrait;
```
这个类就一个`model`类，保存了一个人的用户信息，有三个，`userId`（唯一id），`name`名字，`portraitUri`头像地址`url string`，类中提供了一个初始化方法，传三个参数就可以了，头像可以用`nil`，传`nil`就用默认头像。大家看看我的`RCDataManager`类中用到的`RCUserInfo`的初始化方法，是不是不一样，
```Object-C
        RCUserInfo *aUser = [[RCUserInfo alloc]initWithUserId:myselfInfo.userId name:myselfInfo.realNameportrait:myselfInfo.photo phone:myselfInfo.phone addressInfo:myselfInfo.companyrealName:myselfInfo.realName];
```
我`category`了`RCUserInfo`添加了我要的属性进去，比如公司名字，地址，真是名字等等。在开发中如果觉得`RCUserInfo`类中的属性不够用，可能开发者还需要每个用户带有更多的属性，那么就可以用我的方法，这样一来，每个用户身上就绑定了很多的属性，想怎么用就怎么用。
那么紧接着我把`RCUserInfo`的给大家展示下，这个可能大部分开发者都有这个需求。我是之一，废话不说，直接上代码看`.h`
```Object-C
#import <RongIMLib/RongIMLib.h>

@interface RCUserInfo (Addition)

/**

 用户信息类

 */

/** 用户ID */

@property(nonatomic, strong) NSString *userId;

/** 用户名*/

@property(nonatomic, strong) NSString *name;

/** 头像URL*/

@property(nonatomic, strong) NSString *portraitUri;

/** phone*/

@property(nonatomic, strong) NSString *phone;



/** addressInfo*/

@property(nonatomic, strong) NSString *addressInfo;





/** realName*/

@property(nonatomic, strong) NSString *realName;



/**

 

 指派的初始化方法，根据给定字段初始化实例

 

 @param userId          用户ID

 @param username        用户名

 @param portrait        头像URL

 @param phone           电话

 @param addressInfo     addressInfo

 */

- (instancetype)initWithUserId:(NSString *)userId name:(NSString *)username portrait:(NSString *)portrait phone:(NSString *)phone addressInfo:(NSString *)addressInfo realName:(NSString *)realName;

@end



再看RCUerInfo的category的.m文件。这里用了runtime的知识，不懂的请自行百度或者参考此链接http://www.cnblogs.com/wupher/archive/2013/01/05/2845338.html
#import "RCUserInfo+Addition.h"

#import <objc/runtime.h>


@implementation RCUserInfo (Addition)

@dynamic addressInfo;

@dynamic phone;

- (instancetype)initWithUserId:(NSString *)userId name:(NSString *)username portrait:(NSString *)portrait phone:(NSString *)phone addressInfo:(NSString *)addressInfo realName:(NSString *)realName{

    if (self = [super init]) {

        self.userId        =   userId;

        self.name          =   username;

        self.portraitUri   =   portrait;

        self.phone         =   phone;

        self.addressInfo   =   addressInfo;

        self.realName     =   realName;



    }

    return self;

}



//添加属性扩展set方法

char* const PHONE = "PHONE";

char* const ADDRESSINFO = "ADDRESSINFO";

char* const REALNAME   = "REALNAME";



-(void)setPhone:(NSString *)newPhone{

    

    objc_setAssociatedObject(self,PHONE,newPhone,OBJC_ASSOCIATION_RETAIN_NONATOMIC);

    

}

-(void)setAddressInfo:(NSString *)newAddressInfo{

    

    objc_setAssociatedObject(self,ADDRESSINFO,newAddressInfo,OBJC_ASSOCIATION_RETAIN_NONATOMIC);



}

-(void)setRealName:(NSString *)nweRealName{   

    objc_setAssociatedObject(self,REALNAME,nweRealName,OBJC_ASSOCIATION_RETAIN_NONATOMIC);

}

//添加属性扩展get方法

-(NSString *)phone{

    return objc_getAssociatedObject(self,PHONE);

}

-(NSString *)addressInfo{

    return objc_getAssociatedObject(self,ADDRESSINFO);

}

-(NSString *)realName{

    return objc_getAssociatedObject(self,REALNAME);

}
@end



//获取一个userId的好友的名字

-(NSString *)currentNameWithUserId:(NSString *)userId{

    for (NSInteger i = 0; i<[AppDelegate shareAppDelegate].friendsArray.count; i++) {

        RCUserInfo *aUser = [AppDelegate shareAppDelegate].friendsArray[i];

        if ([userId isEqualToString:aUser.userId]) {

            NSLog(@"current ＝ %@",aUser.name);

            return aUser.name;

        }

    }

    return nil;

}

//这个代理方法很重要，好友信息的提供者，单例的时候，是一定要实现这个代理方法的，并把你的好友信息用一个for循环给completion。

#pragma mark - RCIMUserInfoDataSource

- (void)getUserInfoWithUserId:(NSString*)userId completion:(void (^)(RCUserInfo*))completion

{

    NSLog(@"getUserInfoWithUserId ----- %@", userId);

    

    if (userId == nil || [userId length] == 0 )

    {

        completion(nil);

        return ;

    }

   

    if ([userId isEqualToString:[RCIM sharedRCIM].currentUserInfo.userId]) {

        UserInfoModel *myselfInfo = [UserInfoModel currentUserinfo];

        RCUserInfo *aUser = [[RCUserInfo alloc]initWithUserId:myselfInfo.userId name:myselfInfo.realNameportrait:myselfInfo.photo phone:myselfInfo.phone addressInfo:myselfInfo.companyrealName:myselfInfo.realName];

        completion(aUser);

    }

    

    for (NSInteger i = 0; i<[AppDelegate shareAppDelegate].friendsArray.count; i++) {

        RCUserInfo *aUser = [AppDelegate shareAppDelegate].friendsArray[i];

        if ([userId isEqualToString:aUser.userId]) {

            completion(aUser);

        }

    }

}

//登录融云

-(void)loginRongCloud{

    UserInfoModel *userinfo = [UserInfoModel currentUserinfo];

    if (userinfo.login==YES&&![userinfo.userId isEqualToString:@""]&&![RCIMClientsharedRCIMClient].currentUserInfo.userId) {

        [Networking getAppUserTokenWithUserId:userinfo.userId name:userinfo.realNameportraitUri:userinfo.photo success:^(NetworkModel *model) {

            if ([model.result isEqualToString:@"10000"]) {

                NSLog(@"RCToken = %@",model.data);//model.data就是我们的token啦，有了他，我们才能进行下一步connect

                self.success = YES;

                [[RCIM sharedRCIM] connectWithToken:model.data success:^(NSString *userId) {

                    [RCIM sharedRCIM].globalNavigationBarTintColor = [UIColor whiteColor];

                    NSLog(@"login success with userId %@",userId);

                    [UserInfoModel loginUserinfo:model];

                    //同步好友列表

                    [self syncFriendList:^(NSMutableArray *friends, BOOL isSuccess) {

                        NSLog(@"%@",friends);

                        if (isSuccess) {

                            NSLog(@" success 发送通知");

                            [[NSNotificationCenter defaultCenter] postNotificationName:@"alreadyLogin"object:nil];

                        }

                    }];

//登录成功，设置当前用户是XXX

                    [RCIMClient sharedRCIMClient].currentUserInfo = [[RCUserInfo alloc] initWithUserId:userinfo.userId name:userinfo.userName portrait:userinfo.photo phone:userinfo.phoneaddressInfo:userinfo.company realName:userinfo.realName];

                   

                    

                    [[RCDataManager shareManager] refreshBadgeValue];

                } error:^(RCConnectErrorCode status) {

                    NSLog(@"status = %ld",(long)status);

                } tokenIncorrect:^{

                    self.success = NO;

                    [MSUtil showTipsWithHUD:@"tokenIncorrect"];

                }];

            }

            

        } fail:^(NSError *error) {

            NSLog(@"error %@",error);

            self.success = NO;

        }];

    }else{



    }

}

//刷新角标

-(void)refreshBadgeValue{

    

    dispatch_async(dispatch_get_main_queue(), ^{

        

        int notReadMessage = [[UserInfoModel currentUserinfo].notReadMessage intValue];

        

        NSInteger unreadMsgCount = (NSInteger)[[RCIMClient sharedRCIMClient] getUnreadCount:@[@(ConversationType_PRIVATE)]];

        BaseNavigationController  *chatNav = [AppDelegate shareAppDelegate].rootTabbar.viewControllers[2];

        if (unreadMsgCount == 0) {

            chatNav.tabBarItem.badgeValue = nil;

            if (notReadMessage > 0) {

                [UIApplication sharedApplication].applicationIconBadgeNumber = notReadMessage;

            }

            else {

                [UIApplication sharedApplication].applicationIconBadgeNumber = 0;

            }



        }else{

            chatNav.tabBarItem.badgeValue = [NSString stringWithFormat:@"%li",(long)unreadMsgCount];

            if (notReadMessage > 0) {

                [UIApplication sharedApplication].applicationIconBadgeNumber = unreadMsgCount + notReadMessage;

            }

            else {

                [UIApplication sharedApplication].applicationIconBadgeNumber = unreadMsgCount;

            }



        }

    });

}

//判断有没有这个好友

-(BOOL)hasTheFriendWithUserId:(NSString *)userId{

    if ([AppDelegate shareAppDelegate].friendsArray.count) {

        NSMutableArray *tempArray = [[NSMutableArray alloc]init];



        for (RCUserInfo *aUserInfo in [AppDelegate shareAppDelegate].friendsArray) {

            [tempArray addObject:aUserInfo.userId];

        }

        

        if ([tempArray containsObject:userId]) {

            return YES;

        }

    }
    return NO;

}

@end
```

###好，到此处，已经把融云的逻辑分离出来，并且登录了融云服务器。下一篇章该讲如何聊天，生成聊天列表，显示头像名字等。
持续关注，持续更新，谢谢。博主原创，不得转载，谢谢。



>－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－
－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－
－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－





 ##说说融云即时通讯`SDK`开发篇(三)
 
 
接着开发篇二的内容，我们已经把融云的逻辑分离出来，并且登录了融云服务器。此篇章讲如何聊天，生成聊天列表，显示头像名字等。
 各个`APP`的界面`UI`设计的不同，每个开发者的需求也不同，但是万变不离其宗，我们抓住不变的地方，也就是共同之处。那么共同之处在哪里？就是我们要触发一个事件，比如点击了一个`cell`，或者点击了一个`Button`，又或者我们调用一个接口后台返回给我们一个人的信息，那么各个开发者触发这一系列的事件都是为了和某个人聊天，触发事件的时候，我们一定要拿到对应事件的这个人的信息（`userInfo`，包含很多字段，各个开发者需求不同，字段肯定就不同，但是`userId`一定有的，这个是相同之处）。举个栗子，比如进入一个`tableView`展示的列表，那么我们点击`cell`，动态的去取到每个`cell`对应到人的信息，然后就是在点击`cell`的事件里面配置我们`ConversationViewController`的属性了，那么代码在下面。
      
  ```Object-C    
 ConversationViewController *_conversationVC = [[ConversationViewController alloc]init];
                                _conversationVC.conversationType = ConversationType_PRIVATE;
                                _conversationVC.targetId = [NSString stringWithFormat:@"%@",model.data[@"id"]];
                                
                                _conversationVC.userName = [NSString stringWithFormat:@"%@",model.data[@"agentTeamName"]];
                                _conversationVC.title = [NSString stringWithFormat:@"%@",model.data[@"realName"]];
[self.navigationController pushViewController:_conversationVC animated:YES];
```
 这里的`ConversationViewController`是我自己的VC继承`RCConversationViewController`，`RCConversationViewController`是用的融云写的`UI`，就是说这个`RCConversationViewController`里面的`UI`全部写好了，就很类似`QQ聊天`的界面，`键盘`，`表情`，`发送图片`，`发送语音`，一切的一切都搞定了。我们只需要配置一些属性，然后`push`就可以了。如果我们有自己的需求`UI`，我们也可以适当的在`ConversationViewController`基础上修改。（关于`RCConversationViewController`里面很多方法和属性，后续会慢慢涉及到，现在功能上没有涉及，所以先介绍到这里，后续讲解更高级的功能，就可以把更多的API给带出来，这样才有使用的场景，才更容易理解。）
 
那么每个`app`几乎都会有类似聊天列表的界面。描述一下，就是聊天之后会生成`cell`的，比如你点击了10个人的主页里面的聊天按钮，然后和这10个让聊过天，那么聊天列表就有10个`cell`自动生成（融云内部封装好的，理解为融云的机制，不要过分强求的分析是怎么回事。）。对应在融云这边的类就是`RCConversationListViewController`，注意，这个`RCConversationListViewController`我们也不能直接用啊，我们也要写一个自己的VC，那就是`ChatViewController`继承融云的`RCConversationListViewController`，这个`ChatViewController`就是我们自己写的聊天列表了，我们一旦有和某人聊天，那么自动会生成一个`cell`到这个`vc`里面，里面的机制大家不要去试图理解了，融云已经封装好了，一旦你和老王，前提你登录了融云服务器，并且老王是你的好友，那么你聊天后就可以回来这个`vc`看了，肯定出了一个`cell`，显示的是老王的名字和头像。那么下面我就把这个聊天列表`VC`的功能和`API`详细的介绍下，继续大尺度（没有人其他人愿意这么大尺度了，绝对的福利）的贴代码：
 ` .h里面代码`
  ```Object-C   
  #import "BaseViewController.h"

@interface ChatViewController : RCConversationListViewController


@end
再看.m实现文件里面代码
#import "ChatViewController.h"
#import "RCCustomCell.h"
#import "LoginViewController.h"
#import "SpecialAgentViewController.h"

@interface ChatViewController ()<RCIMReceiveMessageDelegate,RCIMConnectionStatusDelegate
,UIAlertViewDelegate,UITableViewDataSource,UITableViewDelegate>{
    UIView *bgView;
    UILabel *desLabel;
    UIImageView *defaultIV;
    UIView *firstShowView;
    UIImageView  *headerIV;
    MBProgressHUD *hud;
    UIImageView *aIV;
    UILabel *aLabel;
}

@end

@implementation ChatViewController

- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:RCKitDispatchMessageNotification object:nil];
}


- (instancetype)init{
    self = [super init];
    if (self) {
        [[UIApplication sharedApplication]setStatusBarStyle:UIStatusBarStyleLightContent];
        [self setConversationAvatarStyle:RC_USER_AVATAR_CYCLE];
        [self setDisplayConversationTypes:@[@(ConversationType_PRIVATE)]];//这里是会话类型，我只用单聊，所以是ConversationType_PRIVATE，如果你的还有群聊，等，那么点进去，看到一个枚举，找到他们，需要什么类型就加什么类型
        [RCIM sharedRCIM].receiveMessageDelegate = self;//这个是接收消息的监听代理
        [RCIM sharedRCIM].connectionStatusDelegate = self;
        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(didReceiveMessageNotification:)name:RCKitDispatchMessageNotification object:nil];
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(removeLoginHud:) name:@"alreadyLogin" object:nil];
        hud = [[MBProgressHUD alloc]initWithView:self.view];
        hud.square = YES;
        [self.view addSubview:hud];
        
    }
    return self;
}

/**
 * @brief 生成当天的某个点（返回的是伦敦时间，可直接与当前时间[NSDate date]比较）
 * @param hour 如hour为“8”，就是上午8:00（本地时间）
 */
- (NSDate *)getCustomDateWithHour:(NSInteger)hour
{
    //获取当前时间
    NSDate *currentDate = [NSDate date];
    NSCalendar *currentCalendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
    NSDateComponents *currentComps = [[NSDateComponents alloc] init];
    
    NSInteger unitFlags = NSYearCalendarUnit | NSMonthCalendarUnit | NSDayCalendarUnit | NSWeekdayCalendarUnit | NSHourCalendarUnit | NSMinuteCalendarUnit | NSSecondCalendarUnit;
    
    currentComps = [currentCalendar components:unitFlags fromDate:currentDate];
    
    //设置当天的某个点
    NSDateComponents *resultComps = [[NSDateComponents alloc] init];
    [resultComps setYear:[currentComps year]];
    [resultComps setMonth:[currentComps month]];
    [resultComps setDay:[currentComps day]];
    [resultComps setHour:hour];
    
    NSCalendar *resultCalendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
    return [resultCalendar dateFromComponents:resultComps];
}
// @param message 收到的消息实体。
// @param nLeft   剩余消息数。
// 这里是融云的监听消息事件，这个代理会在每次接收到消息的时候走，所以我们可以在这个方法中处理逻辑了，我处理的逻辑有勿扰时段和头像的及时更新等等
-(void)onRCIMReceiveMessage:(RCMessage *)message left:(int)left
{

    
  /*  这个RCMessage是融云的消息体，我姑且称他为消息体，每个消息都是消息体，消息体本身带有很多的信息，我贴上融云源代码，大家看看
    @interface RCMessage : NSObject <NSCopying, NSCoding>
/** 会话类型 */
@property(nonatomic, assign) RCConversationType conversationType;
/** 目标ID，如讨论组ID, 群ID, 聊天室ID */
@property(nonatomic, strong) NSString *targetId;
/** 消息ID */
@property(nonatomic, assign) long messageId;
/** 消息方向 */
@property(nonatomic, assign) RCMessageDirection messageDirection;
/** 发送者ID */
@property(nonatomic, strong) NSString *senderUserId;
/** 接受状态 */
@property(nonatomic, assign) RCReceivedStatus receivedStatus;
/**发送状态 */
@property(nonatomic, assign) RCSentStatus sentStatus;
/** 接收时间 */
@property(nonatomic, assign) long long receivedTime;
/**发送时间 */
@property(nonatomic, assign) long long sentTime;
/** 消息体名称 */
@property(nonatomic, strong) NSString *objectName;
/** 消息内容 */
@property(nonatomic, strong) RCMessageContent *content;
/** 附加字段 */
@property(nonatomic, strong) NSString *extra;

/**
 *  指派初始化方法，根据给定信息初始化实例
 *
 *  @param  conversationType    会话类型
 *  @param  targetId            目标ID，如讨论组ID, 群ID, 聊天室ID
 *  @param  messageDirection    消息方向
 *  @param  messageId           消息ID
 *  @param  content             消息体内容字段
 */
- (instancetype)initWithType:(RCConversationType)conversationType
                    targetId:(NSString *)targetId
                   direction:(RCMessageDirection)messageDirection
                   messageId:(long)messageId
                     content:(RCMessageContent *)content;

/**
 *  根据服务器返回JSON创建新实例
 *
 *  @param  jsonData    JSON数据字典
 */
+ (instancetype)messageWithJSON:(NSDictionary *)jsonData;
+ 

*/
```
接着，别看晕喽，这个消息体是不是很多信息在里面，那么我需要的就是这个`content`字段，以及这个`extra`。我现在给出一个思路去解决好友头像更新之后，我不能马上看到好友的新头像问题。那么假设我和老王正在聊天，聊的很热乎，突然老王去设置里面更新了个头像，我还继续和老王聊天，我能看到老王的新头像吗？你猜猜，如果你深入的看了理论篇，并切理解了理论篇，你就很清楚的知道，我肯定是看不到老王的新头像的，为什么，因为我`app`启动的时候就把老王的信息保存到全局数组里面了，而且融云的聊天列表实际上是有缓存的，我猜测是用数据库存储的。所以我不从新启动`app`的情况下，我是不可能看到老王的新头像的。那么如何解决这个问题呢？
 方案有两个：
  第一个，谁更新了头像就要给他自己的所有好友发送通知消息，这个是融云官方给的方案，我感觉不是太好，这是简单粗暴的。比如老王有1000个好友，那么老王更新了头像，那么就要给这1000个好友发送一条通知信息，通知他的所有的好友，哎！注意了，赶紧更新我的最新信息，我已经更新信息了，那么1000个好友很多都不在线，你发了也是浪费流量，而且很多人，可能有999个人是不关心你更不更新头像的呀！老王你烦不烦，老是更新头像给我发消息。所以这个方法不是很好。
  第二个，我觉得从消息入手，你老王更新了头像，你只要给我发送任何聊天消息我就能检测到你换了头像。怎么检测，那么我们来捋一捋这个逻辑，首先是老王和我在聊天，显示的是现在的头像，突然老王去换了头像，哎，我不知道啊，如果我给老王发消息，老王不理我我，我是拿不到老王的新头像的，那么一旦老王有回复我，就是老王有发一条任意类型的消息（消息有很多类型，语音消息，图片消息，地理位置消息等），我就从消息中获得老王的信息，信息中包含头像字段，那么就可以比较了，如果头像的url改变了，那么说明老王你更新了头像，我检测到了就马上刷新好友的最新信息并放全局数组中，并更新融云的缓存，这个缓存很重要，不更新缓存不行，不更新缓存就等于白做了，更新缓存的方法是 ` [[RCIM sharedRCIM] refreshUserInfoCache:theLastedInfo withUserId:userInfoDic[@"sendUsersId"]];`哇靠！这个方案好，让每条消息带有对方的信息，信息是个`json`的字符串，几乎可以忽略占用流量的大小，而且是最新的信息，不用给不聊天的好友发送通知信息了。好了，看下面代码。
我们用KVC拿到这个附加信息的字符串，是个`json`格式的字符串，一定要用`KVC`，不然你用.是拿不到的。`Xcode`后台调试，可以直接看到这些字段。那么我们需要在老王将要发送还没有发送的方法中去对消息做处理，做什么处理？就是加上老王的最新的信息啊！来看代码，者断代码是聊天界面的，不要搞混淆了（重写`RCConversationViewController`这个`VC`里面的方法）
```Object-C
- (RCMessageContent *)willSendMessage:(RCMessageContent *)messageCotent{
    
    if ([RCIMClient sharedRCIMClient].currentUserInfo.userId) {//如果登录了
        NSDictionary *jsonDic = @{@"sendUsersId":[RCIMClient sharedRCIMClient].currentUserInfo.userId,@"sendUsersName":self.userName,@"sendUsersPhoto":[UserInfoModel currentUserinfo].photo};
        NSError *parseError = nil;
        NSData *jsonData = [NSJSONSerialization dataWithJSONObject:jsonDic options:NSJSONWritingPrettyPrinted error:&parseError];

        NSString *jsonString = [[NSString alloc]initWithData:jsonData encoding:NSUTF8StringEncoding];
        
        if ([messageCotent isKindOfClass:[RCTextMessage class]]) {
            
            RCTextMessage *textMessage = (RCTextMessage*)messageCotent;
            
            textMessage.extra = jsonString;
            
        }else if ([messageCotent isKindOfClass:[RCVoiceMessage class]]) {
            
            RCVoiceMessage *voiceMessage = (RCVoiceMessage*)messageCotent;
            
            voiceMessage.extra = jsonString;
            
        }else if ([messageCotent isKindOfClass:[RCImageMessage class]]) {
            
            RCImageMessage *imageMessage = (RCImageMessage*)messageCotent;
            
            imageMessage.extra = jsonString;
            
        }

    }else{
        
    }
    
    return messageCotent;
}
```
以上的代码很简单，就是发送消息的前面，还没发送的时候，把自己（这时候就是老王自己）的信息，包装成`JSONDic`，然后在转成`NSString`，赋值给`extra`，这样对方就可以拿到我的信息了，就可以知道我的头像有没有更新了。
    以下的代码就是解析`JSON`的过程，和上面交相呼应啊。对不对！
    ```Object-C
    NSString *extraString = [[message valueForKey:@"content"] valueForKey:@"extra"];

    if (extraString) {
        NSData *jsonData = [extraString dataUsingEncoding:NSUTF8StringEncoding];
        NSError *err;
        if (jsonData) {
        拿到字符串，转换称Data，然后转成我们经常用的字典。
            NSDictionary *userInfoDic = [NSJSONSerialization JSONObjectWithData:jsonData
                                                                        options:NSJSONReadingMutableContainers
                                                                          error:&err];
            NSLog(@"FFFFF%@",userInfoDic);
            if (userInfoDic) {
            用RCDataManager的API直接拿到发送者老王的userInfo，然后下面就是比较头像的url是否一样了。
                RCUserInfo *senderInfo = [[RCDataManager shareManager] currentUserInfoWithUserId:userInfoDic[@"sendUsersId"]];
                
                RCUserInfo *theLastedInfo = [[RCUserInfo alloc]initWithUserId:userInfoDic[@"sendUsersId"] name:userInfoDic[@"sendUsersName"] portrait:userInfoDic[@"sendUsersPhoto"] phone:senderInfo.phone addressInfo:senderInfo.addressInfo realName:senderInfo.realName];
                
                
                if ([userInfoDic[@"sendUsersPhoto"] isEqualToString:senderInfo.portraitUri]) {
                    
                }else{
                这里更新好友的最新列表
                    [[RCDataManager shareManager] syncFriendList:^(NSMutableArray *friends, BOOL isSuccess) {
                        if (isSuccess) {
                        这里更新融云缓存，不更新缓存是不行的，我已经实验过了
                            [[RCIM sharedRCIM] refreshUserInfoCache:theLastedInfo withUserId:userInfoDic[@"sendUsersId"]];
                            
                        }else{
                            [[RCDataManager shareManager] syncFriendList:^(NSMutableArray *friends, BOOL isSuccess) {
                                
                            }];
                        }
                    }];
                }
            }
            
            
        }
    }
    
   // JSON字典就是下面的这个样子，iOSer看了就懂了
    //    {"sendUsersId":"85","sendUsersName":"快乐","sendUsersPhoto":"http://weixintest.ihk.cn/ihkwx_upload/userPhoto/13632415461-1449631301776.jpg"}

    NSString *notTroubleStr = [TheUserDefaults objectForKey:@"setSpareTimeYES"];
下面处理的是勿扰时段，就是何时有声音，何时没声音，何时有震动，何时无震动，如果app设置里面没用这些设置项，那么请忽略
    if ([notTroubleStr isEqualToString:@"setSpareTimeNO"]) {
        
        [self RemindSwitch];

    }else if ([notTroubleStr isEqualToString:@"setSpareTimeYES"]){
        
        NSString *timeDuring = [TheUserDefaults objectForKey:@"spareTimeStr"];//11位时间  21:00-09:00
        NSString *fromHourStr = [timeDuring substringToIndex:2];
        NSString *toHourStr   = [timeDuring substringWithRange:NSMakeRange(6,2)];
        NSInteger fromHour=0;
        NSInteger toHour =0;
        if ([fromHourStr hasPrefix:@"0"]) {
            fromHour = [[fromHourStr substringWithRange:NSMakeRange(1, 1)] integerValue];
        }else{
            fromHour = [fromHourStr integerValue];
        }
        
        if ([toHourStr hasPrefix:@"0"]) {
            toHour = [[toHourStr substringWithRange:NSMakeRange(1, 1)] integerValue];
        }else{
            toHour = [toHourStr integerValue];
        }
        NSLog(@"fromHour =%ld ,toHour = %ld",fromHour,toHour);
        NSString *nowString  = [MSUtil gethhmmss];//01:28:02
        nowString = [nowString substringToIndex:2];
        if ([nowString hasPrefix:@"0"]) {
            nowString = [nowString substringFromIndex:0];
        }else{
            
        }
        NSLog(@"%@",nowString);
//        NSDate *nowDate = [self getCustomDateWithHour:fromHour];

        if (fromHour>toHour) {//21－－9跨天
            if ([nowString integerValue]>=fromHour||[nowString integerValue]<=toHour) {
                [RCIM sharedRCIM].disableMessageAlertSound = YES;//关闭声音
                
            }else{
                [self RemindSwitch];
            }
        }else if(fromHour==toHour){//8--8相同
            if ([nowString integerValue]==fromHour) {
                [RCIM sharedRCIM].disableMessageAlertSound = YES;//关闭声音

            }else{
                [self RemindSwitch];
            }
        }else if(fromHour<toHour){//6----10当天
            
            if ([nowString integerValue]>=fromHour&&[nowString integerValue]<=toHour) {
                [RCIM sharedRCIM].disableMessageAlertSound = YES;//关闭声音

            }else{
                [self RemindSwitch];
            }
        }
    }
    
    if (![[RCDataManager shareManager] hasTheFriendWithUserId:message.senderUserId]) {
        //检查message.senderUserId这个人在不在我好友里面，如果没有在，测网络刷新最新的好友列表
        [[RCDataManager shareManager] syncFriendList:^(NSMutableArray *friends,BOOL isSuccess) {
            [[RCDataManager shareManager] getUserInfoWithUserId:message.senderUserId completion:^(RCUserInfo *userInfo) {
                NSLog(@"名字 ＝ %@  ID ＝ %@",userInfo.name,userInfo.userId);
            }];
        }];
    }
    [[RCDataManager shareManager] getUserInfoWithUserId:message.senderUserId completion:^(RCUserInfo *userInfo) {
        NSLog(@"名字 ＝ %@  ID ＝ %@",userInfo.name,userInfo.userId);
    }];
    [self setDisplayConversationTypeArray:@[@(ConversationType_PRIVATE)]];
    [self refreshConversationTableViewIfNeeded];
    [[RCDataManager shareManager] refreshBadgeValue];
}

- (void)RemindSwitch {
    
    NSString *setShake = [TheUserDefaults objectForKey:@"setShakeYES"];
    NSString *setSound=[[NSUserDefaults standardUserDefaults]objectForKey:@"setSoundYES"];

    if (setShake && [setShake isEqualToString:@"setShakeYES"]) {
        AudioServicesPlayAlertSound(kSystemSoundID_Vibrate);//震动
    }
    
    if(setSound && [setSound isEqualToString:@"setSoundYES"])
    {//有声音，这个方法可以设置融云接收消息时候的声音的有和无
        [RCIM sharedRCIM].disableMessageAlertSound = NO;
    }
    else{//无声音
        [RCIM sharedRCIM].disableMessageAlertSound = YES;
    }
}
#pragma mark - RCIMConnectionStatusDelegate
//这个是检测连接connect状态的代理，踢人的功能可以做在这里，逻辑是，staue＝ConnectionStatus_KICKED_OFFLINE_BY_OTHER_CLIENT，就是被T了，然后弹出提示框，提示用户“您的帐号已在别的设备上登录，\n您被迫下线！”，然后点击确定就要调用登出的接口，处理登出的逻辑了，每位开发者自己处理登出的逻辑，登出的逻辑可以单独抽离出来。
/**
 *  网络状态变化。
 *  @param status 网络状态。
 */
- (void)onRCIMConnectionStatusChanged:(RCConnectionStatus)status {
    NSLog(@"RCConnectionStatus = %ld",(long)status);
    if (status == ConnectionStatus_KICKED_OFFLINE_BY_OTHER_CLIENT) {
        UIAlertView *alert = [[UIAlertView alloc]
                              initWithTitle:nil
                              message:@"您的帐号已在别的设备上登录，\n您被迫下线！"
                              delegate:self
                              cancelButtonTitle:@"知道了"
                              otherButtonTitles:nil, nil];
        [alert show];
    }
}

-(id)initWithCoder:(NSCoder *)aDecoder{
    self =[super initWithCoder:aDecoder];
    if (self) {
        //设置要显示的会话类型
        [self setDisplayConversationTypes:@[@(ConversationType_PRIVATE)]];
         [self setConversationAvatarStyle:1];
    }
    return self;
}
-(void)callSpecialAgent:(UITapGestureRecognizer *)sender{
    if (firstShowView) {
        [firstShowView removeFromSuperview];
        firstShowView = nil;
        
    }
    SpecialAgentViewController *saVC = [[SpecialAgentViewController alloc]init];
    [self.navigationController pushViewController:saVC animated:YES];
}
#pragma mark 
#pragma mark  viewDidLoad
- (void)viewDidLoad {
    [super viewDidLoad];
    self.edgesForExtendedLayout = UIRectEdgeNone;
    self.view.backgroundColor = [UIColor whiteColor];
    //设置title
    UIView *netWorkView = (UIView *)self.networkIndicatorView;
    netWorkView.frame = CGRectZero;
    netWorkView.hidden = YES;
    [netWorkView removeFromSuperview];
    self.navigationItem.title=KTitle_isShowAllSame?KTitle_SameStr: @"合记买楼";

    //布局
    UIImage *headerImage = [UIImage imageNamed:@"hjtgdk.jpg"];
    CGFloat radi = headerImage.size.width/headerImage.size.height;
    headerIV =[[UIImageView  alloc]initWithFrame:CGRectMake(0, 0, self.conversationListTableView.frame.size.width, self.conversationListTableView.frame.size.width/(radi))];
    headerIV.userInteractionEnabled = YES;
    headerIV.image = headerImage;
    NSLog(@"%@",self.conversationListTableView);
    NSLog(@"%@",headerIV);
    [self.view addSubview:headerIV];
    self.conversationListTableView.frame = CGRectMake(self.conversationListTableView.frame.origin.x, headerIV.frame.origin.y+headerIV.frame.size.height, self.conversationListTableView.frame.size.width, self.conversationListTableView.frame.size.height-headerIV.frame.size.height);
    self.conversationListTableView.rowHeight = kCellHeight;
    self.conversationListTableView.separatorStyle = UITableViewCellSeparatorStyleSingleLine;
    UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(callSpecialAgent:)];
    [headerIV addGestureRecognizer:tap];
    //设置tableView样式
    self.conversationListTableView.backgroundColor = [MSUtil colorWithHexString:@"#e5e5e5"];
//    self.conversationListTableView.backgroundColor = [UIColor whiteColor];
    self.conversationListTableView.separatorStyle = UITableViewCellSeparatorStyleNone;
    
    
    if ([RCIM sharedRCIM].currentUserInfo.userId) {
        if (self.conversationListDataSource.count==0) {
            self.conversationListTableView.tableFooterView = [UIView new];
        }else{
            UIView *afooter = [[UIView alloc]initWithFrame:CGRectMake(0, 0, self.conversationListTableView.frame.size.width, 0.5)];
            afooter.backgroundColor = [UIColor lightGrayColor];
            self.conversationListTableView.tableFooterView = afooter;
        }
    }else{
        self.conversationListTableView.tableFooterView = [UIView new];
    }
    if ([TheUserDefaults boolForKey:@"everShow"]==NO) {
        firstShowView = [[UIView alloc]initWithFrame:CGRectMake(0, 0, kScreenWidth, kScreenHeight-kNavbarHeight-kTabBarHeight)];
        firstShowView.backgroundColor = [UIColor colorWithWhite:0.2 alpha:0.95];
        [self.view addSubview:firstShowView];
        [self.view bringSubviewToFront:firstShowView];
        [TheUserDefaults setBool:YES forKey:@"everShow"];
        [TheUserDefaults synchronize];
       
        
        UIImage *image = [UIImage imageNamed:@"hjtgdc"];
        float radio = image.size.width/image.size.height;
        UIImageView *centerIV = [[UIImageView alloc]initWithFrame:CGRectMake(0, firstShowView.frame.size.height-((kScreenWidth-2*30)/radio)/2, kScreenWidth-2*30, (kScreenWidth-2*30)/radio)];
        centerIV.image = image;
        centerIV.userInteractionEnabled = YES;
        centerIV.center = firstShowView.center;
        [firstShowView addSubview:centerIV];
        UIButton *Xbtn = [UIButton buttonWithType:UIButtonTypeCustom];
        Xbtn.frame = CGRectMake(centerIV.frame.size.width-50+5, -5, 50, 50);
        [Xbtn setImage:[UIImage imageNamed:@"chatDelete"] forState:UIControlStateNormal];
        [Xbtn setImage:[UIImage imageNamed:@"chatDelete"] forState:UIControlStateSelected];
        [Xbtn addTarget:self action:@selector(removeShowView:) forControlEvents:UIControlEventTouchUpInside];
        [centerIV addSubview:Xbtn];
        [centerIV bringSubviewToFront:Xbtn];
        UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(callSpecialAgent:)];
        [centerIV addGestureRecognizer:tap];
    }
}
-(void)removeShowView:(UIButton *)sender{
    if (firstShowView) {
        [firstShowView removeFromSuperview];
        firstShowView = nil;
    }
    [self showEmptyConversationView];
}
- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    [[UIApplication sharedApplication]setStatusBarStyle:UIStatusBarStyleLightContent];
    [self.navigationController.navigationBar wm_setBackgroundColor:kColor_Theme];
    [self refreshConversationTableViewIfNeeded];
    [self resetConversationListBackgroundViewIfNeeded];
    if ([RCIMClient sharedRCIMClient].currentUserInfo.userId) {//登录
        [[RCDataManager shareManager] refreshBadgeValue];
        [self setDisplayConversationTypeArray:@[@(ConversationType_PRIVATE)]];
        
        aIV.hidden = NO;
        aLabel.hidden = NO;
        [bgView sendSubviewToBack:aIV];
        [bgView sendSubviewToBack:aLabel];
        [hud removeFromSuperview];
    }else{//没登录
        BaseNavigationController  *chatNav =[AppDelegate shareAppDelegate].rootTabbar.viewControllers[2];
        chatNav.tabBarItem.badgeValue = nil;
        [self setDisplayConversationTypeArray:nil];
        
        
        if (self.conversationListDataSource.count) {//已登出，有数据
//            [hud show:YES];
            aIV.hidden = YES;
            aLabel.hidden = YES;
            [bgView sendSubviewToBack:aIV];
            [bgView sendSubviewToBack:aLabel];
            [hud removeFromSuperview];

        }else{//已登出，没数据
            [hud show:NO];
            aIV.hidden = NO;
            aLabel.hidden = NO;
            [hud removeFromSuperview];
        }
        
        dispatch_async(dispatch_get_main_queue(), ^{
            [self refreshConversationTableViewIfNeeded];
            [self showEmptyConversationView];
        });
    }
    [self showEmptyConversationView];

}
-(void)removeLoginHud:(NSNotification *)obj{
    NSLog(@"收到通知");
    if (hud) {
        dispatch_async(dispatch_get_main_queue(), ^{
           
            [self setDisplayConversationTypeArray:@[@(ConversationType_PRIVATE)]];

            [self performSelector:@selector(refershTable) withObject:nil afterDelay:0.5];
        });
        
    }
}
-(void)refershTable{
    dispatch_async(dispatch_get_main_queue(), ^{
        [self.conversationListTableView reloadData];
        [self performSelector:@selector(refershTableIfNeeded) withObject:nil afterDelay:0.5];
    });
}
-(void)refershTableIfNeeded{
    [hud hide:YES];
    [hud removeFromSuperview];
    aIV.hidden  = NO;
    aLabel.hidden = NO;
    dispatch_async(dispatch_get_main_queue(), ^{
        [self refreshConversationTableViewIfNeeded];
        [self showEmptyConversationView];
    });
    
}
-(void)viewWillDisappear:(BOOL)animated{
    [super viewWillDisappear:animated];
    //show
    self.navigationItem.title = @"";
}

#pragma mark override
//通知更新未读消息数目，用于显示未读消息，当收到会话消息的时候，会触发一次。
- (void)notifyUpdateUnreadMessageCount{
    [[RCDataManager shareManager] refreshBadgeValue];
}
-(void)loginorRegister:(UIButton *)sender{
    [self.navigationController pushViewController:[LoginViewController new] animated:YES];
}
//重写方法，设置会话列表emptyConversationView的视图。//无聊天cell的时候一般会显示一个默认的图片或者加一些button或者一些imageView啊等等，那么逻辑就是在下面的这个方法中处理，我的方法可能不是最好的，如果有同学有好方法欢迎找我交流，加我的iOS技术交流群487599875，谢谢。
- (void)showEmptyConversationView{
    UsersType type = [MSUtil checkUserType];
    
    if (firstShowView) {
        return;
    }
    if (bgView==nil) {
      UIImage *  nochatDataImage = [UIImage imageNamed:@"nochatData"];
        bgView = [[UIView alloc]initWithFrame:CGRectMake(0, (kScreenHeight-kNavbarHeight-headerIV.frame.size.height-kTabBarHeight)/2-nochatDataImage.size.height/2, kScreenWidth, nochatDataImage.size.height+kNavbarHeight)];
        bgView.hidden  = NO;
        bgView.center = self.conversationListTableView.center;
        aIV = [[UIImageView alloc]initWithFrame:CGRectMake(kScreenWidth/2-nochatDataImage.size.width/2, 0, nochatDataImage.size.width, nochatDataImage.size.height)];
        aIV.image = nochatDataImage;
        [bgView addSubview:aIV];
        
        aLabel = [[UILabel alloc]initWithFrame:CGRectMake(0, aIV.frame.origin.y+aIV.frame.size.height, bgView.frame.size.width, kNavbarHeight)];
        aLabel.text = @"暂无聊天数据";
        aLabel.textColor = kFontColor_999999;
        aLabel.textAlignment = NSTextAlignmentCenter;
        [bgView addSubview:aLabel];
      
//        UserInfoModel *userinfo = [UserInfoModel currentUserinfo];
//
//        if (userinfo.login==YES&&![userinfo.userId isEqualToString:@""]&&![RCIMClient sharedRCIMClient].currentUserInfo.userId) {
//            
//            aIV.hidden = YES;
//            aLabel.hidden = YES;
//            
//        }else{
//            aIV.hidden = NO;
//            aLabel.hidden = NO;
//        }
        
        
        
        
        
    }
    
    
    if ([RCIM sharedRCIM].currentUserInfo.userId) {//登录了
        if (type==UsersTypeYouke) {

        }else if (type==UsersTypeCustomer){
            self.emptyConversationView = bgView;
            if (self.conversationListDataSource.count) {//有数据
                
            }else{//没数据
                self.emptyConversationView = bgView;
                
                [hud show:NO];
                aIV.hidden = NO;
                aLabel.hidden = NO;
                [hud removeFromSuperview];
            }
            
            
        }else if (type==UsersTypeSales){

            if (self.conversationListDataSource.count) {//有数据

                
            }else{//没数据
                self.emptyConversationView = bgView;
                
                [hud show:NO];
                aIV.hidden = NO;
                aLabel.hidden = NO;
                [hud removeFromSuperview];
            }
            
            self.emptyConversationView = bgView;
        }

    }else{//没登录
        self.emptyConversationView = bgView;
        if (self.conversationListDataSource.count) {//已登出，有数据
            aIV.hidden = YES;
            aLabel.hidden = YES;
            [bgView sendSubviewToBack:aIV];
            [bgView sendSubviewToBack:aLabel];
            [self.view bringSubviewToFront:self.conversationListTableView];
            [hud removeFromSuperview];
            
        }else{//已登出，没数据
             self.emptyConversationView = bgView;

            [hud show:NO];
            aIV.hidden = NO;
            aLabel.hidden = NO;
            [bgView bringSubviewToFront:aIV];
            [bgView bringSubviewToFront:aLabel];
            [hud removeFromSuperview];
        }

    }
    
    if ([self.view.subviews containsObject:bgView]) {
        
    }else{
        [self.view addSubview:self.emptyConversationView];
    }
//
}

//*********************插入自定义Cell*********************//

//插入自定义会话model
-(NSMutableArray *)willReloadTableData:(NSMutableArray *)dataSource{
    for (int i=0; i<dataSource.count; i++) {
        RCConversationModel *model = dataSource[i];
        if(model.conversationType == ConversationType_PRIVATE){
            model.conversationModelType = RC_CONVERSATION_MODEL_TYPE_CUSTOMIZATION;
        }
    }
    return dataSource;
}
#pragma mark
#pragma mark onSelectedTableRow
- (void)onSelectedTableRow:(RCConversationModelType)conversationModelType
         conversationModel:(RCConversationModel *)model
               atIndexPath:(NSIndexPath *)indexPath{
    ConversationViewController *_conversationVC = [[ConversationViewController alloc]init];
    _conversationVC.conversationType = model.conversationType;
    _conversationVC.targetId = model.targetId;
    for (RCUserInfo *userInfo in [AppDelegate shareAppDelegate].friendsArray) {
        if ([model.targetId isEqualToString:userInfo.userId]) {
            _conversationVC.userName = userInfo.name;
            _conversationVC.title =[userInfo.realName isEqualToString:@""]?[NSString stringWithFormat:@"%@",userInfo.name]:[NSString stringWithFormat:@"%@",userInfo.realName];
        }
    }
    [self.navigationController pushViewController:_conversationVC animated:YES];
}
#pragma mark
#pragma mark 禁止右滑删除
-(BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath{
    return YES;
}
//左滑删除
-(void)rcConversationListTableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath{
    //可以从数据库删除数据
    if (indexPath.row < self.conversationListDataSource.count) {
    RCConversationModel *model = self.conversationListDataSource[indexPath.row];
    [[RCIMClient sharedRCIMClient] removeConversation:ConversationType_PRIVATE targetId:model.targetId];
    [self.conversationListDataSource removeObjectAtIndex:indexPath.row];
    [self.conversationListTableView reloadData];
    [[RCDataManager shareManager] refreshBadgeValue];
    }
}
//高度
-(CGFloat)rcConversationListTableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
    return kCellHeight;
}

- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex{
    
    if ([UserInfoModel currentUserinfo].pushToken.length > 0 && [UserInfoModel currentUserinfo].userEncrypt.length > 0) {
        
        [Networking logout:^(NetworkModel *model) {
            
            if ([[NSString stringWithFormat:@"%@",model.result]isEqualToString:@"10000"]) {
                [UserInfoModel logoutUserinfo];
                
                BaseNavigationController  *chatNav = [AppDelegate shareAppDelegate].rootTabbar.viewControllers[2];
                chatNav.tabBarItem.badgeValue = nil;
                [UIApplication sharedApplication].applicationIconBadgeNumber = 0;
                
                [[RCIMClient sharedRCIMClient] clearConversations:@[@(ConversationType_PRIVATE)]];
                [[RCIMClient sharedRCIMClient] disconnect:NO];

                BaseNavigationController *baseVC = (BaseNavigationController *)[AppDelegate shareAppDelegate].rootTabbar.selectedViewController;
                [baseVC pushViewController:[[LoginViewController alloc]init] animated:YES];
            }
            else {
                [MSUtil showTipsWithHUD:model.msg inView:self.view];
            }
            
        } fail:^(NSError *error) {
            [MSUtil showTipsWithHUD:kTips_NetworkError inView:self.view];
        }];
    }
}

-(void)willDisplayConversationTableCell:(RCConversationBaseCell *)cell atIndexPath:(NSIndexPath *)indexPath{
    if ([cell isMemberOfClass:[RCCustomCell  class]]) {
        RCCustomCell *conversationCell = (RCCustomCell *)cell;
//        conversationCell.conversationTitle.text=@"";
        
        RCConversationModel *model = self.conversationListDataSource[indexPath.row];
//        conversationCell.conversationTitle.text = [[RCDataManager shareManager] currentNameWithUserId:model.targetId];
//        if (model) {
//            if (model.targetId) {
//                if ([[AppDelegate shareAppDelegate].friendsArray containsObject:[[RCDataManager shareManager] currentUserInfoWithUserId:model.targetId]]) {
//                    [[RCIM sharedRCIM] refreshUserInfoCache:[[RCDataManager shareManager] currentUserInfoWithUserId:model.targetId] withUserId:model.targetId];
//
//                }
//            }
//        }
        
        for (RCUserInfo *userInfo in [AppDelegate shareAppDelegate].friendsArray) {
            if ([model.targetId isEqualToString:userInfo.userId]) {
                NSDictionary *attributeDic = [NSDictionary dictionaryWithObjectsAndKeys:[UIFont systemFontOfSize:18], NSFontAttributeName, nil];
                CGSize nameLabelSize = [userInfo.realName boundingRectWithSize:CGSizeMake(MAXFLOAT, conversationCell.realNameLabel.frame.size.height) options:NSStringDrawingUsesLineFragmentOrigin attributes:attributeDic context:nil].size;
                conversationCell.realNameLabel.frame = CGRectMake(conversationCell.realNameLabel.frame.origin.x, conversationCell.realNameLabel.frame.origin.y, nameLabelSize.width, conversationCell.realNameLabel.frame.size.height);
                conversationCell.typeNameLabel.frame = CGRectMake(conversationCell.realNameLabel.frame.origin.x+conversationCell.realNameLabel.frame.size.width+10, conversationCell.realNameLabel.frame.origin.y, conversationCell.typeNameLabel.frame.size.width, conversationCell.realNameLabel.frame.size.height);
                if (indexPath.row==self.conversationListDataSource.count-1) {
                    conversationCell.seprateLine.hidden = YES;
                }else{
                    conversationCell.seprateLine.hidden = NO;
                }
            }
        }
        
        
    }
}
//*********************插入自定义Cell*********************//

-(RCConversationBaseCell *)rcConversationListTableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    if (self.conversationListDataSource.count&&indexPath.row < self.conversationListDataSource.count) {
        RCConversationModel *model = self.conversationListDataSource[indexPath.row];
        [[RCDataManager shareManager] getUserInfoWithUserId:model.targetId completion:^(RCUserInfo *userInfo) {
            NSLog(@"rcConversationListTableView 名字 ＝ %@  ID ＝ %@",userInfo.name,userInfo.userId);
        }];
        NSInteger unreadCount = model.unreadMessageCount;
        RCCustomCell *cell = (RCCustomCell *)[[RCCustomCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"RCCustomCell"];
       
        
        NSDate *date = [NSDate dateWithTimeIntervalSince1970:model.receivedTime/1000];
        NSString *timeString = [[MSUtil stringFromDate:date] substringToIndex:10];
        NSString *temp = [MSUtil getyyyymmdd];
        NSString *nowDateString = [NSString stringWithFormat:@"%@-%@-%@",[temp substringToIndex:4],[temp substringWithRange:NSMakeRange(4, 2)],[temp substringWithRange:NSMakeRange(6, 2)]];
        
        if ([timeString isEqualToString:nowDateString]) {
            NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
            [formatter setDateFormat:@"HH:mm"];
            NSString *showtimeNew = [formatter stringFromDate:date];
            cell.timeLabel.text = [NSString stringWithFormat:@"%@",showtimeNew];

        }else{
            cell.timeLabel.text = [NSString stringWithFormat:@"%@",timeString];
        }
        cell.ppBadgeView.dragdropCompletion = ^{
                    NSLog(@"VC = FFF ，ID ＝ %@",model.targetId);
            
            
            
           
            
            [[RCIMClient sharedRCIMClient] clearMessagesUnreadStatus:ConversationType_PRIVATE targetId:model.targetId];
            model.unreadMessageCount = 0;
            NSInteger ToatalunreadMsgCount = (NSInteger)[[RCIMClient sharedRCIMClient] getUnreadCount:@[@(ConversationType_PRIVATE)]];
            
            long tabBarCount = ToatalunreadMsgCount-model.unreadMessageCount;
            int notReadMessage = [[UserInfoModel currentUserinfo].notReadMessage intValue];
            
            if (tabBarCount > 0) {
                [AppDelegate shareAppDelegate].rootTabbar.selectedViewController.tabBarItem.badgeValue = [NSString stringWithFormat:@"%ld",tabBarCount];
                
                if (notReadMessage > 0) {
                    [UIApplication sharedApplication].applicationIconBadgeNumber = notReadMessage + tabBarCount;
                }
                else {
                    [UIApplication sharedApplication].applicationIconBadgeNumber = tabBarCount;
                }
            }
            else {
                [AppDelegate shareAppDelegate].rootTabbar.selectedViewController.tabBarItem.badgeValue = nil;
                
                if (notReadMessage > 0) {
                    [UIApplication sharedApplication].applicationIconBadgeNumber = notReadMessage;
                }
                else {
                    [UIApplication sharedApplication].applicationIconBadgeNumber = 0;
                }
            }
        };
        if (unreadCount==0) {
            cell.ppBadgeView.text = @"";
        
        }else{
            if (unreadCount>=100) {
                cell.ppBadgeView.text = @"99+";
            }else{
                cell.ppBadgeView.text = [NSString stringWithFormat:@"%li",(long)unreadCount];

            }
        }

        

        for (RCUserInfo *userInfo in [AppDelegate shareAppDelegate].friendsArray) {
            if ([model.targetId isEqualToString:userInfo.userId]) {
                
                cell.realNameLabel.text = [userInfo.realName isEqualToString:@""]?[NSString stringWithFormat:@"%@",userInfo.name]:[NSString stringWithFormat:@"%@",userInfo.realName];
                
                
                UsersType type = [MSUtil checkUserType];
                
              
                
                
                if (type==UsersTypeYouke) {
                    
                }else if (type==UsersTypeCustomer){
                    cell.typeNameLabel.text = @"专家";

                }else if (type==UsersTypeSales){
                    cell.typeNameLabel.text = @"客户";
                    cell.typeNameLabel.textColor = kFontColor_999999;
                }

                if ([userInfo.portraitUri isEqualToString:@""]||userInfo.portraitUri==nil) {
                    cell.avatarIV.image = [UIImage imageNamed:@"chatlistDefault"];
                    [cell.contentView bringSubviewToFront:cell.avatarIV];
                }else{
                    [cell.avatarIV sd_setImageWithURL:[NSURL URLWithString:userInfo.portraitUri] placeholderImage:[UIImage imageNamed:@"chatlistDefault"]];
                }
                
                if ([model.lastestMessage isKindOfClass:[RCTextMessage class]]) {
                    cell.contentLabel.text = [model.lastestMessage valueForKey:@"content"];
                    
                }else if ([model.lastestMessage isKindOfClass:[RCImageMessage class]]){
                    
                    if ([model.senderUserId isEqualToString:[RCIMClient sharedRCIMClient].currentUserInfo.userId]) {
                        //我自己发的
                        RCUserInfo *myselfInfo = [RCIMClient sharedRCIMClient].currentUserInfo;
                        
                        if ([[NSString stringWithFormat:@"%@",myselfInfo.realName] isEqualToString:@""]) {
                            cell.contentLabel.text =[NSString stringWithFormat:@"来自\"%@\"的图片消息，点击查看",myselfInfo.name];
                        }else{
                            cell.contentLabel.text =[NSString stringWithFormat:@"来自\"%@\"的图片消息，点击查看",myselfInfo.realName];
                            
                        }
                    }else{
                        
                        cell.contentLabel.text =[NSString stringWithFormat:@"来自\"%@\"的图片消息，点击查看",userInfo.realName] ;
                    }
                    
                }else if ([model.lastestMessage isKindOfClass:[RCVoiceMessage class]]){
                    if ([model.senderUserId isEqualToString:[RCIMClient sharedRCIMClient].currentUserInfo.userId]) {
                        //我自己发的
                        RCUserInfo *myselfInfo = [RCIMClient sharedRCIMClient].currentUserInfo;
                        if ([[NSString stringWithFormat:@"%@",myselfInfo.realName] isEqualToString:@""]) {
                            cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的语音消息，点击查看",myselfInfo.name];
                            
                        }else{
                            cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的语音消息，点击查看",myselfInfo.realName];
                        }
                    }else{
                        cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的语音消息，点击查看",userInfo.realName];
                    }
                }
                else if ([model.lastestMessage isKindOfClass:[RCLocationMessage class]]){
                    if ([model.senderUserId isEqualToString:[RCIMClient sharedRCIMClient].currentUserInfo.userId]) {
                        //我自己发的
                        RCUserInfo *myselfInfo = [RCIMClient sharedRCIMClient].currentUserInfo;
                        if ([[NSString stringWithFormat:@"%@",myselfInfo.realName] isEqualToString:@""]) {
                            cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的位置消息，点击查看",myselfInfo.name];
                        }else{
                            cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的位置消息，点击查看",myselfInfo.realName];
                        }
                    }else{
                        cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的位置消息，点击查看",userInfo.realName];
                    }
                }
                
            }
        }
        
        return cell;
    }
    else{
        
        return [[RCConversationBaseCell alloc]init];
    }
    
    
}
- (void)didSendMessage:(NSInteger)stauts content:(RCMessageContent *)messageCotent{
    NSLog(@"fffff %@",messageCotent);
    
}
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    
  return  self.conversationListDataSource.count;
}

#pragma mark - 收到消息监听
-(void)didReceiveMessageNotification:(NSNotification *)notification{
    __weak typeof(&*self) blockSelf_ = self;
    //处理好友请求
    RCMessage *message = notification.object;

    if ([message.content isMemberOfClass:[RCMessageContent class]]) {
        if (message.conversationType == ConversationType_PRIVATE) {
            NSLog(@"好友消息要发系统消息！！！");
            @throw  [[NSException alloc] initWithName:@"error" reason:@"好友消息要发系统消息！！！" userInfo:nil];
        }
        RCConversationModel *customModel = [RCConversationModel new];
        //自定义cell的type
        customModel.conversationModelType = RC_CONVERSATION_MODEL_TYPE_CUSTOMIZATION;
        customModel.senderUserId = message.senderUserId;
        customModel.lastestMessage = message.content;
        dispatch_async(dispatch_get_main_queue(), ^{
            //调用父类刷新未读消息数
            [blockSelf_ refreshConversationTableViewWithConversationModel:customModel];
            [super didReceiveMessageNotification:notification];
            [blockSelf_ resetConversationListBackgroundViewIfNeeded];
            [self notifyUpdateUnreadMessageCount];
            
            //当消息为RCContactNotificationMessage时，没有调用super，如果是最后一条消息，可能需要刷新一下整个列表。
            //原因请查看super didReceiveMessageNotification的注释。
            
        });
        
    }else if (message.conversationType == ConversationType_PRIVATE){
        //获取接受到会话
        RCConversation *receivedConversation = [[RCIMClient sharedRCIMClient] getConversation:message.conversationType targetId:message.targetId];
        
        //转换新会话为新会话模型
        RCConversationModel *customModel = [[RCConversationModel alloc] init:RC_CONVERSATION_MODEL_TYPE_CUSTOMIZATION conversation:receivedConversation extend:nil];
        dispatch_async(dispatch_get_main_queue(), ^{
            //调用父类刷新未读消息数
            [blockSelf_ refreshConversationTableViewWithConversationModel:customModel];
            //[super didReceiveMessageNotification:notification];
            [blockSelf_ resetConversationListBackgroundViewIfNeeded];
            [self notifyUpdateUnreadMessageCount];
            
            //当消息为RCContactNotificationMessage时，没有调用super，如果是最后一条消息，可能需要刷新一下整个列表。
            //原因请查看super didReceiveMessageNotification的注释。
            NSNumber *left = [notification.userInfo objectForKey:@"left"];
            if (0 == left.integerValue) {
                [super refreshConversationTableViewIfNeeded];
            }
        });
    } else {
        dispatch_async(dispatch_get_main_queue(), ^{
            //            调用父类刷新未读消息数
            [super didReceiveMessageNotification:notification];
            [blockSelf_ resetConversationListBackgroundViewIfNeeded];
            [self notifyUpdateUnreadMessageCount];
            
            //        super会调用notifyUpdateUnreadMessageCount
        });
    }
    [[RCDataManager shareManager] getUserInfoWithUserId:message.senderUserId completion:^(RCUserInfo *userInfo) {
        NSLog(@"didReceiveMessageNotification 名字 ＝ %@  ID ＝ %@",userInfo.name,userInfo.userId);
    }];
    [self refreshConversationTableViewIfNeeded];
}
- (void)didReceiveMemoryWarning {
    NSLog(@"ChatViewController ReceiveMemoryWarning");

    [super didReceiveMemoryWarning];
}
@end
     融云封装了这个聊天列表的table，但是也提供（或者说暴露）了接口API给我们使用，当然细心的读者可以发现，我用了自定义的cell，#import "RCCustomCell.h"
这个RCCustomCell是自定义的cell，继承RCConversationBaseCell，RCConversationBaseCell是会话cell的基类 ，看源代码
/**
 *  会话Cell基类
 */
@interface RCConversationBaseCell : UITableViewCell

/**
 *  会话数据模型
 */
@property(nonatomic, strong) RCConversationModel *model;

/**
 *  设置会话数据模型
 *
 *  @param model 会话数据模型
 */
- (void)setDataModel:(RCConversationModel *)model;
@end
```
我们可以用融云暴露出来的`API`，完成一些自己想要加的功能和逻辑，下面罗列一下一些基本的`api`和属性
这个是`dataSource`数组，里面装的全是我们聊天的`model`
`@property(nonatomic, strong) NSMutableArray *conversationListDataSource;`
这个就是我们的`table`了，想改变`frame`和`颜色`等，都可以对他下手
`@property(nonatomic, strong) UITableView *conversationListTableView;`
网络不好的时候，这个带一个红色提示的东东老是出现，我直接给他设置`frame`为`CGRectZero`，如果你觉得好，可以忽略这个`view``
`@property(nonatomic, strong) RCNetworkIndicatorView *networkIndicatorView;`
这个属性是告诉table，让他显示什么聊天的类型，当然我的是单聊 就这样设置喽` [self setDisplayConversationTypes:@[@(ConversationType_PRIVATE)]];`
`@property(nonatomic, strong) NSArray *displayConversationTypeArray;`

下面这个是会话为空的试图，那么如果你想自定义，就要处理这个逻辑了，还有方法配套使用的，方法名字就是
`- (void)showEmptyConversationView;`
```Object-C
/**
 *  会话列表为空时的视图
 */
@property(nonatomic, strong) UIView *emptyConversationView;


类似就是点击cell的事件
/**
 *  表格选中事件
 *
 *  @param conversationModelType 数据模型类型
 *  @param model                 数据模型
 *  @param indexPath             索引
 */
- (void)onSelectedTableRow:(RCConversationModelType)conversationModelType
         conversationModel:(RCConversationModel *)model
               atIndexPath:(NSIndexPath *)indexPath;

自定义cell的话，肯定是要实现返回自定义cell的方法
#pragma mark override
/**
 *  重写方法，可以实现开发者自己添加数据model后，返回对应的显示的cell
 *
 *  @param tableView 表格
 *  @param indexPath 索引
 *
 *  @return RCConversationBaseTableCell
 */
- (RCConversationBaseCell *)rcConversationListTableView:(UITableView *)tableView
                                  cellForRowAtIndexPath:(NSIndexPath *)indexPath;
改变cell的高度，简单，不解释
/**
 *  重写方法，可以实现开发者自己添加数据model后，返回对应的显示的cell的高度
 *
 *  @param tableView 表格
 *  @param indexPath 索引
 *
 *  @return 高度
 */
- (CGFloat)rcConversationListTableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath;
- 

点击删除的事件，可以处理一些高级逻辑，删除行，还要同时删除model，等等
#pragma mark override
/**
 *  重写方法，点击tableView删除按钮触发事件
 *
 *  @param tableView    表格
 *  @param editingStyle 编辑样式
 *  @param indexPath    索引
 */
- (void)rcConversationListTableView:(UITableView *)tableView
                 commitEditingStyle:(UITableViewCellEditingStyle)editingStyle
                  forRowAtIndexPath:(NSIndexPath *)indexPath;
```
如果你要用融云系统的cell，那么这里的API可以不写，紧紧设置一定属性就可以了，比如头像是圆的还是方的 
`[self setConversationAvatarStyle:RC_USER_AVATAR_CYCLE]; `
比如，显示单聊的还是群聊的`cell`，     
`[self setDisplayConversationTypes:@[@(ConversationType_PRIVATE)]];`
等等，其他的可以不写。当有需要的时候才去看这些`API`，并`override`，重写，重写之后处理你自己的逻辑。


 好了，上面已经把聊天列表和聊天界面都讲完了。更新头像都逻辑和方案以及你们最喜欢的代码都贴上了。
 聊天的东西到此为止。下面会带来自定义`cell`的方法和代码参考。持续关注，持续更新，谢谢！



>－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－
－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－
－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－

 ##说说融云即时通讯SDK开发篇之自定义cell(四)
 
 
   此章节专门讲述如何自定义聊天列表中的`cell`，`cell`，都可以自定义，高的，矮的，`cell`里面加了是男是女的标志，加了说话人的电话，加了说话人的公司，等等等等都可以解决。还有如何操作这个`cell`，这里的操作意思比较广泛了，比如给`cell`中的控件赋值，右滑动删除`cell`删除了还要处理逻辑，给`cell`加上类似`QQ好友`列表中的`Badge`，角标。
   废话少说，对于程序员来说，代码才最解渴，继续大尺度贴代码（我如果紧紧讲解理论让你们去自己写，那么和官方文档没区别了，也没了价值）。
    说一下大前提，用代码表示就是
    `
    if（你的融云的SDK已经集成功能了，可以聊天产生cell了）｛
     continue，继续阅读自定义cell
     ｝else｛
      return；
    ｝
    `
    
   首先我们要新建一个`cell`类，名字叫做`RCCustomCell`，然后继承融云的`RCConversationBaseCell`，这个是融云的会话`cell`的基类。
   我们来看看基类的源代码，如下:
   ```Object-C
   /**
 *  会话Cell基类
 */
@interface RCConversationBaseCell : UITableViewCell

/**
 *  会话数据模型
 */
@property(nonatomic, strong) RCConversationModel *model;

/**
 *  设置会话数据模型
 *
 *  @param model 会话数据模型
 */
- (void)setDataModel:(RCConversationModel *)model;
@end
```
 可以看到，每个`cell`里面对应一个`model`，这个`model`就是会话的`model`，这个`model`保存了很多的有关会话的信息，这些信息对我们自定义`cell`提示作用。这个`model`都保存了什么信息，继续揭开他的面纱，哎呀，其实就是点进去看源码，我们看到，`model`保存了会话的类型`RCConversationType`，还有用户自定义的数据`id extend`，还有会话的`id`，就是`targetId`，目标`id`就是对方的`userId`，还有会话中未读消息数unreadMessageCount，这个`unreadMessageCount`很有用，我们下面会用到，是每个`cell`中对应的model的未读消息数，不是总的未读消息数哦。还有`senderUserId`发送消息`用户id`，`senderUserName`，发送消息的用户名字，另外还有接收时间`long long receivedTime`和发送的时间`long long sentTim`e等等。
    这个是不是和我们从网络获取的`json`数据一样，然后用我们自己建的`model`保存起来，然后放数据源数组里面进行对`table`里面的每个`cell`的控件赋值，很像，其实就是啦，只不过`model`和`数据源数组`我们不用管理了，融云已经封装好了。岂不是更方便。

   那么有人说我的`cell`上面还想放置其他的东西，比如每个医生的地址和电话，还有的开发者需要放置每个用户的鲜花个数，粉丝数量等等，那么都是可以的，怎么实现呢，这些都没在`model`里面。我们这样想，你能拿到`model`，就可以拿到对方的`userId`，这个没问题，那么对方的`id`你都知道了，你还拿不到他的所有信息吗？肯定拿得到得呀，他不是在一个好友得数组里面吗？去数组里面找啊，那么我们得`RCDataManger`有两个方法，再拿过来看看
  ` -(BOOL)hasTheFriendWithUserId:(NSString *)userId;`
  ` -(RCUserInfo *)currentUserInfoWithUserId:(NSString *)userId;`
我们可以通过`userId`判断有没有这个好友，并且可以拿到这个人得RCUserInfo，就是他得所有信息了。这个RCUserInfo在初始化得时候要提供一个特殊得初始化方法，把每个好友得所有信息字段都传进去，这个方法前面有讲到。不会得同学要复习一下前面得东西了。


好了，说这么多了，终于把`cell`的理论讲解完了，终于可以任性的贴代码了
先看`.h`文件里面。
```Object-C
#import <RongIMKit/RongIMKit.h>
#import "PPDragDropBadgeView.h"
//cell的高度，这里是80各高度
#define kCellHeight 80

@interface RCCustomCell : RCConversationBaseCell
///头像
@property (nonatomic,retain) UIImageView *avatarIV;
///真实姓名
@property (nonatomic,retain) UILabel *realNameLabel;
///头衔
@property (nonatomic,retain) UILabel *typeNameLabel;
///时间
@property (nonatomic,retain) UILabel *timeLabel;
///内容
@property (nonatomic,retain) UILabel *contentLabel;
///分割线
@property (nonatomic,retain) UILabel *seprateLine;
///角标（UIView）这里我用了一个第三方，很好用，提供Github地址https://github.com/smallmuou/PPDragDropBadgeView
@property (nonatomic,retain) PPDragDropBadgeView *ppBadgeView;

@end
```
>再看`.m`文件里面的代码

```Object-C
#import "RCCustomCell.h"

#define kbadageWidth 20
#define kgap 10

/*
 
 avatarIV
 
 realNameLabel
 typeNameLabel
 timeLabel
 contentLabel
 
 */
@implementation RCCustomCell

-(instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier
{
    self = [super initWithStyle:style reuseIdentifier:reuseIdentifier];
    if (self) {
        
        

        
        //头像
        self.avatarIV = [[UIImageView alloc]initWithFrame:CGRectMake(kgap, kgap, kCellHeight-2*kgap, kCellHeight-2*kgap)];
        self.avatarIV.clipsToBounds = YES;
        self.avatarIV.layer.cornerRadius = 8;
//        self.avatarIV.image = [UIImage imageNamed:@"default_portrait_msg"];
        [self.contentView addSubview:self.avatarIV];
        //realName
        self.realNameLabel = [[UILabel alloc]initWithFrame:CGRectMake(self.avatarIV.frame.origin.x+self.avatarIV.frame.size.width+kgap, self.avatarIV.frame.origin.y+7, self.avatarIV.frame.size.width+40, self.avatarIV.frame.size.height/2-kgap/2)];
        self.realNameLabel.font = [UIFont systemFontOfSize:18];
        self.realNameLabel.textColor = kFontColor_333333;
//        self.realNameLabel.backgroundColor = [UIColor cyanColor];
        [self.contentView addSubview:self.realNameLabel];
        //头衔
        self.typeNameLabel = [[UILabel alloc]initWithFrame:CGRectMake(self.realNameLabel.frame.origin.x+self.realNameLabel.frame.size.width+kgap, self.realNameLabel.frame.origin.y, self.realNameLabel.frame.size.width*2, self.realNameLabel.frame.size.height)];
        self.typeNameLabel.font = [UIFont systemFontOfSize:15];
        self.typeNameLabel.textColor = kColor_TintRed;
//        self.typeNameLabel.backgroundColor = [UIColor magentaColor];
        [self.contentView addSubview:self.typeNameLabel];
        //时间
        self.timeLabel = [[UILabel alloc]initWithFrame:CGRectMake(kScreenWidth-90-5, self.typeNameLabel.frame.origin.y, 90, self.typeNameLabel.frame.size.height)];
        self.timeLabel.textAlignment = NSTextAlignmentRight;
        self.timeLabel.font = [UIFont systemFontOfSize:15];
        self.timeLabel.textColor = kFontColor_999999;
        [self.contentView addSubview:self.timeLabel];
        //内容
        self.contentLabel = [[UILabel alloc]initWithFrame:CGRectMake(self.realNameLabel.frame.origin.x, self.realNameLabel.frame.origin.y+self.realNameLabel.frame.size.height+2, kScreenWidth-self.avatarIV.frame.size.width-2*kgap-5-30, self.typeNameLabel.frame.size.height)];
//        self.contentLabel.backgroundColor = [UIColor purpleColor];
        self.contentLabel.textColor = kFontColor_999999;
        self.contentLabel.font = [UIFont systemFontOfSize:15];
        [self.contentView addSubview:self.contentLabel];
        //角标，这个用来显示每个好友的角标，实现类似QQ聊天列表中强大的拖曳角标的功能
        self.ppBadgeView = [[PPDragDropBadgeView alloc]initWithFrame:CGRectMake(self.contentLabel.frame.origin.x+self.contentLabel.frame.size.width, self.contentLabel.frame.origin.y, 25, 25)];
        self.ppBadgeView.fontSize = 12;
        [self.contentView addSubview:self.ppBadgeView];
        ///分割线
        self.seprateLine =[[UILabel alloc]initWithFrame:CGRectMake(self.realNameLabel.frame.origin.x, kCellHeight-2, kScreenWidth-self.avatarIV.frame.size.width-2*kgap, 0.5)];
        //        self.contentLabel.backgroundColor = [UIColor purpleColor];
        self.seprateLine.backgroundColor = [UIColor lightGrayColor];
        [self.contentView addSubview:self.seprateLine];
    }
    return self;
}
-(void)layoutSubviews{
    [super layoutSubviews];
    
}
@end
```
到这里，我们就把`cell`自定义代码写完了，那么下面就是使用这个`cell`了。怎么用，怎么替换系统的`cell`。继续往下看，到聊天列表`vc`中改代码。

```Object-C
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
  return  self.conversationListDataSource.count;
}
//高度
-(CGFloat)rcConversationListTableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath{
    return kCellHeight;
}
#pragma mark
#pragma mark 是否禁止右滑删除
-(BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath{
    return YES;
}
//左滑删除
-(void)rcConversationListTableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath{
    //可以从数据库删除数据
    if (indexPath.row < self.conversationListDataSource.count) {
    RCConversationModel *model = self.conversationListDataSource[indexPath.row];
    [[RCIMClient sharedRCIMClient] removeConversation:ConversationType_PRIVATE targetId:model.targetId];
    [self.conversationListDataSource removeObjectAtIndex:indexPath.row];
    [self.conversationListTableView reloadData];
    [[RCDataManager shareManager] refreshBadgeValue];
    }
}

 //*********************修改自定义Cell中model的会话model类型*********************//

//插入自定义会话model,这个很重要，一定要实现
-(NSMutableArray *)willReloadTableData:(NSMutableArray *)dataSource{
    for (int i=0; i<dataSource.count; i++) {
        RCConversationModel *model = dataSource[i];
        if(model.conversationType == ConversationType_PRIVATE){//如果是单例，那么我们改model的一个属性，就是把会话model的类型改为自定义
            model.conversationModelType = RC_CONVERSATION_MODEL_TYPE_CUSTOMIZATION;
        }
    }
    return dataSource;
}
```
 > 开始使用自定义cell
 
 //*********************插入自定义Cell*********************//
这个返回cell的方法，以前也贴出来过，不过注视比较少，这次专门来讲解这个返回cell的方法，这里就返回了我们自定义的cell
```Object-C
-(RCConversationBaseCell *)rcConversationListTableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    if (self.conversationListDataSource.count&&indexPath.row < self.conversationListDataSource.count) {
    //首先我们要从数据源数组中取出融云的model类，方便下面使用model的属性，大把资料要从model的属性中获取，或者间接获取到
        RCConversationModel *model = self.conversationListDataSource[indexPath.row];
        [[RCDataManager shareManager] getUserInfoWithUserId:model.targetId completion:^(RCUserInfo *userInfo) {
            NSLog(@"rcConversationListTableView 名字 ＝ %@  ID ＝ %@",userInfo.name,userInfo.userId);
        }];
        NSInteger unreadCount = model.unreadMessageCount;//每个cell的未读消息数量
        RCCustomCell *cell = (RCCustomCell *)[[RCCustomCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"RCCustomCell"];
       我们alloc 一个自己的cell，下面给他赋值，处理逻辑
        
        NSDate *date = [NSDate dateWithTimeIntervalSince1970:model.receivedTime/1000];
        NSString *timeString = [[MSUtil stringFromDate:date] substringToIndex:10];
        NSString *temp = [MSUtil getyyyymmdd];
        NSString *nowDateString = [NSString stringWithFormat:@"%@-%@-%@",[temp substringToIndex:4],[temp substringWithRange:NSMakeRange(4, 2)],[temp substringWithRange:NSMakeRange(6, 2)]];
        
        if ([timeString isEqualToString:nowDateString]) {
            NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
            [formatter setDateFormat:@"HH:mm"];
            NSString *showtimeNew = [formatter stringFromDate:date];
            //这里赋值时间，时间显示，融云mode里面的是long long类型的，用的时候一定要转化，方法和代码都有了，不多赘述
            cell.timeLabel.text = [NSString stringWithFormat:@"%@",showtimeNew];

        }else{
            cell.timeLabel.text = [NSString stringWithFormat:@"%@",timeString];
        }
        //处理角标的拖曳，这里拖曳完成之后回调一个block，在block里面我们要处理逻辑了，你把角标拖曳掉之后，要调用RCIMClient这个单例，设置某人的未读消息为0.因为你已经拖曳掉了这个角标啊。到此还不行，因为数据源里面model的属性没变，model的属性没变，滑动table，角标又一次出现了，治标不治本，那么我们直接设置model.unreadMessageCount = 0;那么我们这样就做到了，融云服务器上的某人的未读消息数量和本地cell上面的一致了；
        cell.ppBadgeView.dragdropCompletion = ^{
                    NSLog(@"VC = FFF ，ID ＝ %@",model.targetId);
            
            
            
           
            
            [[RCIMClient sharedRCIMClient] clearMessagesUnreadStatus:ConversationType_PRIVATE targetId:model.targetId];
            model.unreadMessageCount = 0;
            NSInteger ToatalunreadMsgCount = (NSInteger)[[RCIMClient sharedRCIMClient] getUnreadCount:@[@(ConversationType_PRIVATE)]];
            
            long tabBarCount = ToatalunreadMsgCount-model.unreadMessageCount;
            int notReadMessage = [[UserInfoModel currentUserinfo].notReadMessage intValue];
            
            if (tabBarCount > 0) {
                [AppDelegate shareAppDelegate].rootTabbar.selectedViewController.tabBarItem.badgeValue = [NSString stringWithFormat:@"%ld",tabBarCount];
                
                if (notReadMessage > 0) {
                    [UIApplication sharedApplication].applicationIconBadgeNumber = notReadMessage + tabBarCount;
                }
                else {
                    [UIApplication sharedApplication].applicationIconBadgeNumber = tabBarCount;
                }
            }
            else {
                [AppDelegate shareAppDelegate].rootTabbar.selectedViewController.tabBarItem.badgeValue = nil;
                
                if (notReadMessage > 0) {
                    [UIApplication sharedApplication].applicationIconBadgeNumber = notReadMessage;
                }
                else {
                    [UIApplication sharedApplication].applicationIconBadgeNumber = 0;
                }
            }
        };
        if (unreadCount==0) {
            cell.ppBadgeView.text = @"";
        
        }else{
            if (unreadCount>=100) {
                cell.ppBadgeView.text = @"99+";
            }else{
                cell.ppBadgeView.text = [NSString stringWithFormat:@"%li",(long)unreadCount];

            }
        }

        

        for (RCUserInfo *userInfo in [AppDelegate shareAppDelegate].friendsArray) {
            if ([model.targetId isEqualToString:userInfo.userId]) {
                //赋值名字属性
                cell.realNameLabel.text = [userInfo.realName isEqualToString:@""]?[NSString stringWithFormat:@"%@",userInfo.name]:[NSString stringWithFormat:@"%@",userInfo.realName];
                
                
                UsersType type = [MSUtil checkUserType];
                
              
                
                
                if (type==UsersTypeYouke) {
                    
                }else if (type==UsersTypeCustomer){
                    cell.typeNameLabel.text = @"头衔";
                    
                }else if (type==UsersTypeSales){
                    cell.typeNameLabel.text = @"客户";
                    cell.typeNameLabel.textColor = kFontColor_999999;
                }
    //赋值头像
                if ([userInfo.portraitUri isEqualToString:@""]||userInfo.portraitUri==nil) {
                    cell.avatarIV.image = [UIImage imageNamed:@"chatlistDefault"];
                    [cell.contentView bringSubviewToFront:cell.avatarIV];
                }else{
                    [cell.avatarIV sd_setImageWithURL:[NSURL URLWithString:userInfo.portraitUri] placeholderImage:[UIImage imageNamed:@"chatlistDefault"]];
                }
                
                if ([model.lastestMessage isKindOfClass:[RCTextMessage class]]) {
                    cell.contentLabel.text = [model.lastestMessage valueForKey:@"content"];
                    
                }else if ([model.lastestMessage isKindOfClass:[RCImageMessage class]]){
                    
                    if ([model.senderUserId isEqualToString:[RCIMClient sharedRCIMClient].currentUserInfo.userId]) {
                        //我自己发的
                        RCUserInfo *myselfInfo = [RCIMClient sharedRCIMClient].currentUserInfo;
                        
                        if ([[NSString stringWithFormat:@"%@",myselfInfo.realName] isEqualToString:@""]) {
                            cell.contentLabel.text =[NSString stringWithFormat:@"来自\"%@\"的图片消息，点击查看",myselfInfo.name];
                        }else{
                            cell.contentLabel.text =[NSString stringWithFormat:@"来自\"%@\"的图片消息，点击查看",myselfInfo.realName];
                            
                        }
                    }else{
                        
                        cell.contentLabel.text =[NSString stringWithFormat:@"来自\"%@\"的图片消息，点击查看",userInfo.realName] ;
                    }
                    
                }else if ([model.lastestMessage isKindOfClass:[RCVoiceMessage class]]){
                    if ([model.senderUserId isEqualToString:[RCIMClient sharedRCIMClient].currentUserInfo.userId]) {
                        //我自己发的
                        RCUserInfo *myselfInfo = [RCIMClient sharedRCIMClient].currentUserInfo;
                        if ([[NSString stringWithFormat:@"%@",myselfInfo.realName] isEqualToString:@""]) {
                            cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的语音消息，点击查看",myselfInfo.name];
                            
                        }else{
                            cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的语音消息，点击查看",myselfInfo.realName];
                        }
                    }else{
                        cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的语音消息，点击查看",userInfo.realName];
                    }
                }
                else if ([model.lastestMessage isKindOfClass:[RCLocationMessage class]]){
                    if ([model.senderUserId isEqualToString:[RCIMClient sharedRCIMClient].currentUserInfo.userId]) {
                        //我自己发的
                        RCUserInfo *myselfInfo = [RCIMClient sharedRCIMClient].currentUserInfo;
                        if ([[NSString stringWithFormat:@"%@",myselfInfo.realName] isEqualToString:@""]) {
                            cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的位置消息，点击查看",myselfInfo.name];
                        }else{
                            cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的位置消息，点击查看",myselfInfo.realName];
                        }
                    }else{
                        cell.contentLabel.text = [NSString stringWithFormat:@"来自\"%@\"的位置消息，点击查看",userInfo.realName];
                    }
                }
                
            }
        }
        
        return cell;
    }
    else{
        
        return [[RCConversationBaseCell alloc]init];
    }
    
    
}
```

到此，自定义`cell`应该没问题了，如果有问题，群里联系我 `479259423`

# 欢迎关注我的斗鱼直播间，用手机斗鱼TV，直接搜索“文明直播间”或者“极端恐惧”就可以找到我的直播。iOS技术分享直播。进来点一下关注，谢谢，开播会有推送到大家手机。（个人直播，非机构，适合初级iOS和中级iOS，高级iOS不用进来喷我了，谢谢）。
