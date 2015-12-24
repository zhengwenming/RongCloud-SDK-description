# RongCloud-SDK-description-
下面两篇文章分别介绍融云的SDK即时通讯机制和集成步骤，由于国内CSDN博客封杀我这样的文章（说我打广告），所以只能去国外发表了。希望大家支持我，谢谢。
 
 说说融云即时通讯SDK理论篇(一) 

标签： 即时通讯聊天融云sdkiOS
2015-12-24 10:35 13人阅读 评论(0) 收藏 编辑 取消置顶  删除
 分类：
即时通讯
版权声明：本文为博主原创文章，未经博主允许不得转载。

本人用过融云的1.4.4和2.2.4版本的SDK，分别用于对两款APP的即时通讯功能开发集成，主要用了IMKit类，功能为单聊（聊天室和群聊不涉及，理解了单聊，聊天室和群聊也就不攻而破）,吐槽一下关于iOS端融云SDK集成资料很少，只能看融云官方的开发文档，然并卵，官方文档不及时更新，语言和关键词用的也比较官方专业化，我想通俗易通，雅俗共赏的资料，没用！没用！真的没有！博客之类的资料只能找到安卓的文章，无奈只能看iOS小屌丝看安卓大神写的关于融云集成的android代码，小伙伴们看的是很蛋疼。现在福利来了，iOS端的融云集成博客这边看，会不断更新，持续关注。想集成融云聊天功能的小伙伴不防先普及下理论知识，防止以后走弯路又或者总是问些不靠谱的问题，“我为什么不能聊天”，“为什么头像和名字出不来”等等一系列问题，文章中如果有错误的地方希望大家提出，本人会及时改正，避免误人子弟，高手勿喷，哈哈。。。

        融云的SDK可以解决很多app中即时通讯的功能，无论是安卓还是iOS，都可以考虑用第三方去实现聊天的功能，而且现在app中集成聊天的功能很火很时髦，不是吗？收废品、旧家电的老板都恨不得为自己的APP加上即时通讯翅膀，让自己的app飞起来。废话少说，先来理解下融云的即时通讯机制，也就是理论部分。

        融云不维护好友关系，只负责转发消息。不维护好友关系是什么意思？我用融云的SDK集成聊天，好友之间都可以进行聊天啊，怎么会不维护好友关系呢！？那么我们来理解下：“不维护好友关系，只负责转发消息”的意思是融云的服务器给我们转发消息，融云的SDK完成发送消息的功能（SDK中的方法实现，后台也可以调用融云提供的接口发送消息），消息从一个人fromUserId（每个人都有一个id，后面会详细讲到）转发到另一个人toUserId那里，那么每个用户的好友关系和群组关系是要保存在自己公司的服务器的，这点就解释了上面的话，融云提供一个转发消息的平台（服务器和SDK以及接口等），因为要满足广大开发者的需求，所以不可能为没个开发者都维护这个好友关系和群组关系。举个栗子，用户A有100个好友，那么这个融云是不知道的，也不会关心用户A的好友关系，谁知道呢？我们开发者自己的后台要知道，我们自己的服务器要维护好友关系和群组关系，具体做法就是要用接口去建立好友关系，然后再用另外的一个接口去保存用户A的所有好友。这样的话，用户A登录之后就可以调用好友列表的接口拿到自己的所有好友，然后就可以为所欲为了，其实也就是和自己的好友发送消息了，发送消息的时候，不是走自己公司的服务器发送消息，注意，这里发送消息是走融云的服务器，怎么会走融云的服务器呢？奇怪，融云怎么知道我登录了呢？别急，下面会讲到token，以及connect连接融云服务器。消息的流向是这样的，从自己这里fromUserId（自己的id）走到融云的服务器，然后融云服务器转发到好友toUserId那里，因为消息体中含有很多的属性，消息体本身可以附带信息，比如targetId（就是toUserId，这里两者一样），extra附加信息，用于提供给开发者拓展功能，你想让消息带什么信息都可以带，但是大小有限制，一般带字符串或json数据，我用这个字段解决了好友头像的及时更新功能，这个字段的用法放开发篇章中细细讲解。

现在说说userId，用户的“身份证号码”。每个APP的注册用户都会被分配一个userId，就是用来区分用户的，就好像我们的身份证号码一样，不会重复，具有唯一性。那么再说说谁去分配用户A的userId呢？融云会分配吗？答案是NO。还是那句话融云只负责转发消息，分配userId这个任务还是要我们后台自己来搞，当用户A注册的时候，后台分配给A一个userId并且返回给前端开发人员使用，前端开发人员拿到A的userId之后，就要用这个A用户的唯一的userId去登录融云的服务器，当然不是任何人有个userId都可以随便连接融云的服务器的哦，大家都知道任何第三方公司都没有那么随便，不安全，会被攻击，是要Token验证的。有这个Token就让你连接，没有token的话，滚蛋。整个登录的流程是这样滴，开发者去融云官网注册个账号---->登录融云官网---->创建一个应用---->拿到融云分配给应用的appkey---->app中代码先注册融云的appKey---->获取token获取token---->拿token去connect融云服务器。大概就是这样，但是开发过程中要注意以下几点：

（一）appkey包括正式上线的appkey和开发环境的appkey。现在的新SDK取消用appSecret，我们代码中用的是appkey，不是appSecret，所以暂时不要管appSecret，但是不要随便刷新这个秘钥，后果自负。

（二）token怎么获取，哪里去获取？token的获取是要自己公司的后台提供一个获取融云token的专门接口，自己的后台开发人员去看文档做接口，后台相关事情这里不做解答（一句话：去看融云开发文档）。获取到token后，就可以拿一个用户A的userId去连接融云的服务器了，融云的方法叫connectWithToken，connect成功了之后会返回登录人的userId，失败了会返回error的描述（一个枚举，详细的解释connect失败的原因）。具体看下面代码，注意，我的做法是每次都获取token，还有其他的处理token方式，具体的看融云开发指南，我这里提供一种解决方案。

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


连接成功了，继续看。要想和好友聊天，我们还要有好友，有了好友才能正常聊天，我的好友是保存在一个全局的数组里面的，app启动的时候，我网络请求自己的后台，拿到我们后台维护的好友数组，然后放数组里面，这样我全局可以拿到。另外，还要实现一个代理方法，这个代理叫做RCIMUserInfoDataSource（2.2.4版本的叫RCIMUserInfoDataSource，以前的老版本比较麻烦些，要实现两个代理

RCIMFriendsFetcherDelegate,RCIMUserInfoFetcherDelegagte

）这个代理方法是提供好友信息的，给谁提供好友信息？你和你的好友聊天，系统会自动走代理方法，去拿好友信息，所以那些说“我的好友头像和名字怎么出不来”的同学要注意了，遵守代理了吗？是不是你的代理没设置，是不是没实现代理方法，方法里面有没有问题，有没有配置好你的好友信息？不然好友头像和名字怎么会出不来呢！附上代理方法名字和实现
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




最后开始发消息给好友，比如点击一个cell（或者一个页面的Button），拿到这个cell对应人的userId，拿到对方的userId后，就可以建立一个会话了，那么开始上代码

ConversationViewController *_conversationVC = [[ConversationViewControlleralloc]init];

_conversationVC.conversationType = ConversationType_PRIVATE;//单聊（也叫私聊），会话类型

_conversationVC.targetId = [NSStringstringWithFormat:@"%@",model.data[@"id"]];//消息的目标id，对方的userId，就是上文说的toUserId

_conversationVC.userName = [NSStringstringWithFormat:@"%@",model.data[@"agentTeamName"]];//会话对方的人名字

_conversationVC.title = [NSStringstringWithFormat:@"%@",model.data[@"realName"]];//会话导航上的title

[self.navigationControllerpushViewController:_conversationVCanimated:YES];



ConversationViewController是开发者自己建立的VC，继承融云的RCConversationViewController类，继承之后，子类就可以用重写父类的方法了，想用什么就点进去找什么方法

好了，现在可以发送消息了，因为RCConversationViewController类已经实现了UI和功能。



理论篇到此结束。下面会有开发篇，谢谢。有问题在我的开发群里提问，群号487599875.






 说说融云即时通讯SDK开发篇(二) 



理论知识理解之后，就可以快速集成了。有多快？常常有开发者问，一天可以集成融云的聊天功能吗？我想说我现在一天都搞不定，你不用处理逻辑的吗？你仅仅是写个demo还是做项目啊？如果是写个demo，我1个小时可以集成聊天功能了。所以大家还是要按部就班的来，欲速测不达。

好了，不扯淡了，开始集成。

一，去融云官网（http://www.rongcloud.cn）下载SDK。补充一句最好用官网最新的SDK，千万不要倒退啊，最新的SDK比较完善，把以前旧版本的SDK的bug都修复了，当然肯定还有bug存在，但是已经比较少了，而且你也不一定就踩到雷了，如果踩到了，去给融云发工单，每个工单都会得到回答，大部分会令你满意的，不满意也没办法。好了，下载完之后找到SDK的文件夹，打开，然后自己在自己工程中建立一个新的文件夹比如我的就是RongCloud_SDK_2_2_4，这样清晰明了的看到了作用和版本号，再然后就是拖SDK进来工程中的文件夹RongCloud_SDK_2_2_4，除去release_notes_ios.txt不用拖进来，其他的都拖进来，不然你就拖了两个frameWork，后面开发中肯定你会叫，为什么我的表情😊出不来，为什么我的是英文的，各种奇葩bug出现了，愿意就在此，你要把emoji的plist文件和语言国际化的东东也一起拉进工程。记得点击copy选项。

二，添加frameWork，而且要全面。在第一步的基础上添加融云SDK的依赖库，都是系统的，官网有库列表，慢慢加。

AssetsLibrary.framework
AudioToolbox.framework
AVFoundation.framework
CFNetwork.framework
CoreAudio.framework
CoreGraphics.framework
CoreLocation.framework
CoreMedia.framework
CoreTelephony.framework
CoreVideo.framework
ImageIO.framework
libc++.tbd
libc++abi.tbd
libsqlite3.tbd
libstdc++.tbd
libxml2.tbd
libz.tbd
MapKit.framework
OpenGLES.framework
QuartzCore.framework
SystemConfiguration.framework
UIKit.framework

三、导入头文件
#import <RongIMKit/RongIMKit.h>

这个一般应该是在appdelegate里面倒入，因为程序入口，我们要初始化融云的appkey（理论篇有详细介绍，开发篇不再赘述）。
在这个方法中初始化
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions ，最好写一个方法专门放置融云的逻辑，initRongCloud方法放置融云的初始化

- (void)initRongCloud{

    self.friendsArray = [[NSMutableArray alloc]init];

    //测试环境          AppKey XXXXXAppKey        AppSecret XXXXXXAppSecret

    //正式上线环境       AppKey XXXXXAppKey       AppSecret  XXXXXAppKey

    NSString *rongYunKey = @"yourAPPKey";

    if ([kNetwork_Host isEqualToString:@"http://weixintest.ihk.cn"]) {//正式环境用正式key，开发测试环境用测试的key

        rongYunKey = @"yourAPPKey";

    }

    [[RCIM sharedRCIM] initWithAppKey:rongYunKey];

    [RCIM sharedRCIM].userInfoDataSource = [RCDataManager shareManager];

/**enableMessageAttachUserInfo

 *  默认NO，如果YES，发送消息会包含自己用户信息。

 */


    [RCIM sharedRCIM].enableMessageAttachUserInfo = YES;



    [[RCDataManager shareManager] loginRongCloud];

}

初始化完成后就是拿token，然后用一个人的userId去connect了，那么这个逻辑我没有把代码写在appdelegate，因为业务相同的逻辑应该分离出来，要不全部代码写appdelegate里面太乱，太难维护，我们把有关融云的逻辑抽离出来，放一个类RCDataManager中去管理，那么我的这个RCDataManager完成了什么功能，什么逻辑，具体怎么设计这个类，为什么要这样设计。下面详细介绍下怎么把融云的逻辑模块分离出来，写出高一点质量的代码。
RCDataManager是个单例类，主要功能就是登录融云，刷新好友列表，设置好友信息提供者代理等等，设置tabbar的角标等等。他的好处是把融云的逻辑分离出来，业务逻辑分离，方便维护工程，我们用的时候就一句代码就可以了，比如登录就这样 [[RCDataManager shareManager] loginRongCloud];具体RCDataManager类的代码如下：

@interface RCDataManager : NSObject<RCIMUserInfoDataSource>







@property(nonatomic,assign)BOOL success;

+(RCDataManager *) shareManager;

- (void)getUserInfoWithUserId:(NSString*)userId completion:(void (^)(RCUserInfo*))completion;

/**

 *  从服务器同步好友列表

 */

-(BOOL)hasTheFriendWithUserId:(NSString *)userId;

-(void)loginRongCloud;

-(void) syncFriendList:(void (^)(NSMutableArray * friends,BOOL isSuccess))completion;

-(void)refreshBadgeValue;

-(NSString *)currentNameWithUserId:(NSString *)userId;

-(RCUserInfo *)currentUserInfoWithUserId:(NSString *)userId;

@end

－－－－－－－－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－－－－－－
再看实现文件里面的代码

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

    UsersType type = [MSUtil checkUserType];

    if (type==UsersTypeYouke) {//游客

    //不能聊天

        NSLog(@"游客 发送通知");



        [[NSNotificationCenter defaultCenter] postNotificationName:@"alreadyLogin" object:nil];

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

这个类就一个model类，保存了一个人的用户信息，有三个，userId（唯一id），name名字，portraitUri头像地址url string，类中提供了一个初始化方法，传三个参数就可以了，头像可以用nil，传nil就用默认头像。大家看看我的RCDataManager类中用到的RCUserInfo的初始化方法，是不是不一样，
        RCUserInfo *aUser = [[RCUserInfo alloc]initWithUserId:myselfInfo.userId name:myselfInfo.realNameportrait:myselfInfo.photo phone:myselfInfo.phone addressInfo:myselfInfo.companyrealName:myselfInfo.realName];

我category了RCUserInfo添加了我要的属性进去，比如公司名字，地址，真是名字等等。在开发中如果觉得RCUserInfo类中的属性不够用，可能开发者还需要每个用户带有更多的属性，那么就可以用我的方法，这样一来，每个用户身上就绑定了很多的属性，想怎么用就怎么用。
那么紧接着我把RCUserInfo的给大家展示下，这个可能大部分开发者都有这个需求。我是之一，废话不说，直接上代码看.h

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


－－－－－－－－－－－－－－－－－－－－－－－－－－－－－华丽丽的分割线－－－－－－－－－－－－－－－－－－－－－－－－－－－－－


再看RCUerInfo的category的.m文件。这里用了runtime的知识，不懂的请自行百度
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


好，到此处，已经把融云的逻辑分离出来，并且登录了融云服务器。下一篇章该讲如何聊天，生成聊天列表，显示头像名字等。
持续关注，持续更新，谢谢。博主原创，不得转载，谢谢。

