# 我对领域驱动设计的理解历程

不可能指望自己看一遍书、做几个项目就能深入理解DDD。本文记录我对领域驱动设计的曲折的理解历程。

## 一、纯SQL

时间：大二开学前的小学期。

<!-- 大二开学前的小学期，我对数据库性能特别敏感，每天都在 -->

大二开学前的小学期那2周，我喜欢用纯SQL进行编程，或者是用C# linq2db库，借助表达式树这一特性编写SQL（实质还是SQL）。

其实我当时是会使用ORM的。那么，为什么我没有使用ORM呢？主要原因有以下2点：
1. SQL历史悠久，是稳定的选择。
2. ORM无法顺畅表达诸如`Update ... Where <CONDITION>`的这一类操作。如果真要以ORM实现，那么就得将符合`<CONDITION>`条件的行取回来，进行修改，然后保存回数据库。一取一存，性能会受到负面影响。
3. SQL是一种表达性非常好的语言，ORM无法表达顺畅表达`Update set X = X + 1`这种操作。这和上一条比较相似。


## 二、实体

时间：大二上学期

“实体框架”翻译成英文是什么？EntityFramework。当Microsoft使用EntityFramework (EF) 这一名称来命名ORM时，它向使用者传达了一种编程思想：实体思维。实体思维是比SQL更加高端的思维。使用实体思维可以更好地表达业务逻辑。

大二上学期，我回顾我小学期写的代码，发现是一坨屎。我同时发现，使用EntityFramework是一个不错的选择。此后，我便放弃SQL思维，转向实体思维。


## 三、DDD的魔力来源于充血

时间：大二上学期

### DDD惊鸿一瞥

由于对SQL的屎山的不满，在大二上学期，我使用EntityFramework重构了小学期项目。但是，重构后，我依然业务代码过于杂乱，遂于搜索引擎搜索诸如“如何写好业务逻辑”、“如何写好业务代码”的网页。

现在回想起来，可以看出，我的思维渐渐从“性能”转向“业务逻辑”。说句不相关的话：现在仍有很多程序员喜欢动不动就讨论性能，从论坛上的非科班程序员到经验丰富的大佬（例如赵劼），都热衷于讨论“性能”并且忽视业务逻辑。如果认真翻看《DDD》《IDDD》，或者是MartinFowler的博客，你几乎看不到关于性能的讨论；即使要讨论，也会将业务和性能严格区分开进行讨论。

让我感觉到DDD的力量的，是阿里云栖社区的一篇文章[《一文教会你如何写复杂业务代码》](https://developer.aliyun.com/article/712581)。当时，对于文章中的一些诸如“知识沉淀”的名称我是不太懂的。但是，这篇文章里面有一处关于使用DDD和不使用DDD的代码对比，则让我模糊地感受到了DDD的力量。

这篇文章并没有教给我太多关于DDD的知识。它只是让我相信：DDD的力量很强，能够让业务代码优雅。此后，我便在互联网上把所有能搜到的DDD资料都搜出来看（夸张）。


### 入坑DDD

《一文教会你如何写复杂业务代码》只是给我带来了模糊的认知和对DDD的向往，并未教我关于DDD的细节。真正让我入坑DDD的，是微软的系列教程：[Tackle Business Complexity in a Microservice with DDD and CQRS Patterns](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/)

从实践中掌握知识是最快的。我一边看微软的教程，一边使用IDE写代码，很快就掌握了DDD的相关概念。


### DDD的魔力来源于实体对业务逻辑的封装

DDD的魔力来源于实体对业务逻辑的封装。如果使用约定俗成的名词，那么这句话应该是“**DDD的魔力来源于充血**”。这一观点也是我当时坚信的观点。我的理由如下：

首先，不论是微软的DDD教程，还是MartinFowler的博客，都常常提及“贫血模型”、“充血模型”的概念。这就让我认为，是充血模型对于业务逻辑的封装，简化了其他地方的代码，让代码干净整洁。

其次，我自己也做了一些实验，来验证这一观点。

**实验**

在IM聊天软件里面，好友关系是如何存放的？我们不难想到，应当这样设计实体：

```cs
class Friendship {
    string UserId1, UserId2;
    string Nickname;
}

class FriendRequest {
    string SenderId, ReceiverId;
    string Message, QuestionAnswer;
    bool IsHandled, Agree, Blocked;
}
```

A用户和B用户的好友关系，应当使用2个Friendship实体进行表达。因为他们都要给对方取不同的备注（nickname字段）。示例数据如下图所示。

```cs
Friendship {
    UserId1: "A", UserId2: "B",
    Nickname: "..."
}

Friendship {
    UserId1: "B", UserId2: "A",
    Nickname: "..."
}
```

当然，以上的业务逻辑只是复杂业务逻辑的一小部分。如果简单思考一下QQ的业务逻辑，我们可以发现存在以下的业务逻辑需要表达：
- 如果A向B之前发送的`FriendRequest`未被处理，那么A向B新发送的`FriendRequest`就应当覆盖之前的。
- 如果A之前的FriendRequest被拉黑，那么A无法再向B发送。
- 用户可以修改加好友同意方式：直接同意、需要验证消息、需要回答问题作为验证、需要正确回答问题。
- 如果A之前的FriendRequest被拉黑，但是A又将同意方式修改为“直接同意”，那么A再加好友就直接通过，而不是被block。
- 如果A的验证方式是“需要回答问题作为验证消息”，B给A发送了好友请求，之后A修改了问题的内容和数量，那么之前的好友请求依然应当显式旧问题。

除此之外，还有一大堆复杂的业务逻辑。这时，我发现，使用对象的方式来管理业务逻辑是非常强大的。

```cs

class FriendRequest {
    ValidationType type; // 需要验证消息 | 需要回答问题作为验证
    string ValidationMessage;
    string[][] QuestionsAndAnswers;
}

class FriendshipDealer {
    string AId, BId;
    Friendship AtoBFriendship, BtoAFriendship;
    FriendRequest AtoBRequest, BToARequest;

    bool AreFriends() {
    }

    bool CanSendRequest(string id) {
    }
}

// 注：为了CQRS.Q的方便
// 例如为了让数据库系统知道AtoB和BToA的Scheme是一样的，
// 我们应当这样设计：
class FriendshipDealer {
    string[] Ids;
    Friendship[] Friendships;
    FriendRequest[] Requests;
}

```

这样设计是非常强大的。首先，我们将好友关系涉及到的所有需要考虑的要素放在一个AggregateRoot中，将各种判断放在方法里面，让大量业务聚集在一起，更容易维护。这种纯对象的东西，更加容易写单元测试，软件质量更高。另外，FriendRequest还存放了QuestionsAndAnswers，以应对用户中途修改验证方式所造成的Questions改变；如果使用传统的建模方法，不知道又得增加多少张表。

此外，在业务的讨论范围之外，这种方法还具备并发控制准确的优点。


### 我的认知状态

毫无疑问，上面的实验坚定了我内心的想法。我当时认为，DDD **90%** 的魔力来源于充血模型。

所以，我忽视了其他的很多需要拆分的地方。在开发过程中，我只管将尽量多的业务逻辑放进实体，对于其他的代码，我一律放进Application Layer，也就是Controller。

接下来，我将使用以代码为主的形式，串联起我对DDD的认知的变化。在此之前，还有一些和DDD看似不太相关，却又必须要知道的信息。

**Auth类**

首先要了解一个贯穿全文的类：`Auth`。可能它和这一节不相关，但是这却是当时我开始采用的设计。理解它，才能理解我对DDD的认知的进步。

Auth是在同一个请求scope的服务间共享的用于储存和读取登录信息的服务。我自己写的`AuthMiddleware`会读取Cookie、查询数据库，将UserId填入`Auth`，然后Controller通过Auth获取登录状态、用户Id等信息。是不是很方便？在后文，我们会发现，这种设计是幼稚的设计，会让领域服务和应用服务紧耦合。

```cs
class Auth {
    public string UserId {get; set;}
    public bool IsLoggedIn => UserId != null;
}
```

除了那些应当封装进实体的代码外，其余的代码我都放在Controller的方法中。现在，我的代码架构还未出现问题。接下来，随着业务的复杂化，我们的代码的问题会逐渐凸显。


## 四、按依赖分类，形成对模块化的指导

时间：大三下学期

请注意，从现在开始，我会结合代码架构的变更，逐步将主题引导到小标题“按依赖分类，形成对模块化的指导”上。

### 需求：查询用户名的接口
我们需要有一个查询用户名的接口。
```python
# GET /api/me
{
    "username": "chr"
}
```

所以，我们的AuthMiddleware在查询数据库后，应当将用户名也附在Auth上面。我们的Auth理所应当的修改为以下的结构：

```cs
class Auth {
    public string UserId {get; set;}
    public bool IsLoggedIn => UserId != null;

    public string Username {get; set;}
}
```

### 需求：请求限速

我们写了一个ApplicationService `LimiterService`进行请求限速。在我当时看来，限速和领域无关，由于和核心业务不相关，所以应当归类为ApplicationService。请求限速的策略有2种：按照IP限速、按照用户限速。

所以，理所应当地，Auth被设计为如下的结构：

```cs
class Auth {
    public string UserId {get; set;}
    public bool IsLoggedIn => UserId != null;

    public string Username {get; set;}
    
    public string IP {get; set;}
}
```

与此同时，`LimiterService`的设计如下：

```cs

class LimiterService {
    // Inject是伪代码，意思是依赖注入
    [Inject] Auth auth;

    // 令牌桶算法
    [Inject] IKvRepository<LimiterBucket> bucketRepo;

    void LimitUser() {
        // ... auth.UserId
    }

    void LimitIP() {
        // ... auth.IP
    }
}

```

### 发现问题：测试

在采取上述设计后，我的单元测试的编写出现了问题。问题来源于LimitService。

我之前的单元测试是这样写的：
```cs
var controller = resolve<RecordController>();
var repo = resolve<IRepository<Record>>();

var auth = resolve<Auth>();
auth.UserId = "1" // 这样来替代AuthMiddleware的行为

controller.CreateRecord(xxx);

// check repo ...

```

现在加入了LimitService，问题来了：单元测试有可能触发限速，导致失败。

解决方案：
- 方案1：增加一个开关类（例如在Auth里面添加一项），控制RateLimiter的行为
- 方案2：将CreateRecord单独拆分为DomainService，对DomainService进行测试。

虽然2种方案都能够解决问题，但是，从直觉上来看，我认为方案2更好。

### 拆分为Domain Service引发的思考

**Auth**

我们将`RecordController.CreateRecord`拆分到`RecordService`(Domain Service)中。但是，在拆分过程中，问题出现了——Auth混杂了来自Application层的东西：IP、Username。对于DomainService有用的，好像只有UserId。

那么，我们是否应当简单地增加一个服务`CurrentUserService { string UserId }`这样做呢？这样做确实解决了刚刚所说的“混杂”的问题，但是我还是觉得不太合适。因为DomainService层不应该存在“当前用户”这一概念。

为何会出现“当前用户”这一概念？“当前用户”是由应用层的结构决定的。应用层是http服务，采取request/response模式，那么就形成了“当前用户”这一概念。例如，假设应用层变成了批量处理，那么CurrentUser这一概念就会失效。所以如果为DomainService层加入一个CurrentUserService，那么就会违背DDD的规律。

让我们来看看IDDD这本书是怎么做的。IDDD里面使用了UserDescriptor这一ValueObject作为中介。其实，UserDescriptor是专门为身份上下文和其他的上下文相互交流而定义的结构。在设计时，不仅要注意不能将Application层的东西混入到UserDescriptor，还要注意，UserDescriptor里面的东西要尽量少，不能让其他上下文依赖不必要的字段。

在我的系统中，UserDescriptor的设计如下：

```cs
class UserDescriptor {
    public string UserId {get;}
    public bool IsLoggedIn => UserId != null;

    public UserSescriptor(string userId){
        UserId = userId;
    }
}
```

乍一看，UserDescriptor和CurrentUserService差别不大，真正存放数据的字段都是一样的：UserId。它们的区别在于：
1. 生命周期。CurrentUserService本质上是一个供服务间共享数据的服务，封装性有限。并且，它在被AuthenticationMiddleware修改之前，都不是一个有效的对象，这和面向对象的精神违背。而UserDescriptor是一个封装性良好的ValueObject。
2. 传递方式。UserDescriptor是DomainService的方法的参数；而CurrentUserService则是一个依赖。从这个角度来说，UserDescriptor更加灵活。

其他问题：
1. 为什么没有Username？查询Username的接口不是还需要吗？答：查询Username 的接口，属于CQRS.Query。CQRS.Query和DomainService关系不大。

**连接Application层与DomainService层**

既然采用了UserDescriptor的设计，那么，在Application层，我们就需要一些“胶水”，实现从cookie到UserDescriptor的桥梁。

所以，我设计了`AuthenticationService`这一ApplicationService，作为桥梁。

```cs
public class AuthenticationService
{
    public const string JwtSecret = "xxx";
    private readonly TokenAccessor tokenAccessor;

    // DomainService
    [Inject] public IdentityService IdentityService { get; }


    private UserDescriptor _userDescriptor;
    private string _username;

    // 这个方法是“桥梁”
    public async Task<UserDescriptor> UserDescriptor()
    {
        await EnsureTokenProcessedAsync();

        Debug.Assert(_userDescriptor != null);
        return _userDescriptor;
    }

    // 当然，这个方法只是顺便给application layer用
    public async Task<string> UsernameAsync()
    {
        await EnsureTokenProcessedAsync();

        return _username;
    }

    private async Task EnsureTokenProcessedAsync()
    {
        if (_userDescriptor != null)
        {
            return;
        }

        var user = await LoginWithToken();

        SetPropertiesUsingUser(user);
    }
}
```

Controller的代码长这样：
```cs
public async Task<IActionResult> OnPost()
{
    var ud = await authenticationService.UserDescriptor();

    await rateLimitService.LimitCreateRecordAsync(HttpContext.GetIP());

    var record = await recordService.CreateRecord(ud, Input.Name, Input.Text);

    this.FlashMessage($"Successfully created record {record.Name}. RecordId: {record.Id}");
    return RedirectToPage("/Index"); // TODO: redirect to record list
}
```

Controller的代码，就应该简简单单地调用各个ApplicationService、DomainService，将Entity和ValueObject在它们之间传递，就像胶水一样。“胶水”也符合Application层的定位。六边形架构也是如此，最外面的那一层永远是起“胶水”作用的适配器。


**限速真的不属于业务吗？**

在之前，我将请求限速设置为ApplicationService。但是，转念一想：限速真的不属于业务吗？我看未必。

限速也可以设置为一个领域服务。只不过，由于限速和主要业务关联真的不大，所以，我们将限速也分离到单独的子域中。

*提示：子域的范围应当是限界上下文的范围，所以，本文中这两个词的意义差不多。*

**限界上下文之间 vs 之内**

那么，我们要进行思考：同一个限界上下文（BC）内部，和不同BC间，到底有什么不同呢？

上文说了，限界上下文之间传递的Entity和ValueObject，应当明确定义，就像你的http接口一样，既要保持功能，又不能频繁变更。另外，MartinFowler也有佐证的观点。在Martin Fowler的 [BoundedContext](https://martinfowler.com/bliki/BoundedContext.html) 中，他提到了DDD deals with large models by dividing them into different Bounded Contexts and **being explicit about their interrelationships** .

**限界上下文之内：Domain-Repository-DomainService层的本质**

在限界上下文之内，这三层的本质是什么？它们从什么角度改善了软件开发？

请看文章：[DDD Math](./dddmath.md)

从这篇我之前的文章中可以看出，Domain-Repository-DomainService的分层的本质，其实是“**按依赖分类，形成对模块化的指导**”。这也是这一节的小标题。


## 五、限界上下文之间：按领域划分限界上下文

[DDD Math](./dddmath.md) 这篇文章从代码的数学结构方面简单地提及了限界上下文之间的关联。

在理解上一节的“按依赖分类，形成对模块化的指导”后，我们也不难看出，划分BC，实质上是避免单个BC过于膨胀。既要BC小，又要BC间的互动方式不会抖动，就需要凭借领域知识，按领域划分限界上下文。

除此之外，按照领域划分BC，对软件复杂度升高还有着天然的应对作用。以上面的限速子域、身份验证子域为例。注意，在下文中，身份验证子域既包含Authentication，又包含Authorization的功能。我为了方便，将这两个东西放一起。在实际过程中尽量拆分。

刚开始，我们Authorization的逻辑很简单。例如，想要判断一个用户是否对某一个Record实体有操作的权限，只需要判断`Record.OwnerId == UserDescriptor.Id`即可。但是，随着业务的复杂，我们的Authorization的逻辑也会变复杂。以下情况，均会增加业务的复杂性：
- 增加一个管理员角色，以便于管理内容
- 为了推出命令行工具的开箱即用的功能，现增加一个匿名用户，优点是无需注册开箱即用，缺点是仅仅能够进行部分操作

随着业务的不断复杂，我们Authorization的逻辑也不断复杂。我们的业务代码中，可能会充斥着这样的代码：

```python
# ud: UserDescriptor
def update_record(ud, record_id, title, text):
    record = record_repo.find_by_id(record_id)

    if ud.is_admin:
        xxx
    elif ud.is_anonymous:
        yyy
    else:
        zzz
```

假设在最开始，我们只需要增加管理员的判断，匿名用户的需求还未到来。那么，我们很可能就会图方便，直接在核心域的代码里面添加这种代码。随着后面需求越来越多，我们也会被温水煮青蛙，最终形成屎山。

如果我们在最开始就按照领域，将系统划分为核心域、身份认证子域（Authentication+Authorization），我们的代码会成为下面这种。

```python
def update_record(ud, record_id, title, text):
    record = record_repo.find_by_id(record_id)

    if ud.has_control_over_record(record):
        xxx
        update record
        yyy
    else:
        return error_object

def has_control_over_record(self: UserDescriptor, record):
    return self.user_id == record.owner_id
```

我们可以看到，在最开始，业务还很简单的时候，`has_control_over_record`只有一行函数。任何人（包括我）都有将这种函数打为“过度模块化”或者“过度工程化”。但是，随着业务的膨胀，这种一行函数也为逐渐复杂的业务逻辑提供安身之所，让复杂的业务逻辑不至于污染核心域：

```python
def has_control_over_record(self: UserDescriptor, record):
    if ud.is_admin:
        return True
    
    if ud.is_anonymous:
        return False
    
    return self.user_id == record.owner_id
```

题外话：只具备1行代码的函数是需要推崇的。在MartinFowler的[FunctionLength](https://martinfowler.com/bliki/FunctionLength.html) 这篇博客中，1行的函数占44%，1-2行的函数占比58%，1-3行的占比68%。

如果再加上“限速子域”，我们将更能够理解划分子域的优势。

```python
def limit_create_record(ud, ip):
    if ud.is_admin:
        return

    if ud.is_anonymous:
        limit_ip(ip)
    else:
        limit_user(ud)
```

然后，在核心域的代码中，我们只需要这样做：

```python
# ApplicationLayer 所做的工作可以被称作 adapter
ud, ip = adapter.get_ud(), adapter.get_ip()

limit_xxx(ud, ip)

record = get_record(record_id)

if not ud.has_control_over_record(record):
    return error_object

record_service.xxx(yyy)

# 其实，限速是由applayer调用，还是放进核心域（record_service），
# 都是见仁见智。毕竟这都是比较靠外层的一些“胶水”的工作。
```

我们从上面的实例中可以看到，我们的限速子域、身份认证子域，好像天生就具备收纳复杂业务代码、避免核心域膨胀的“魔力”。这种魔力的来源是什么？是不是随意进行子域的划分，都能够得到这样好的结果？

这种魔力来源于领域知识。只有深入理解领域，并**按领域划分**，才能够施展这种魔力。这也对应了小标题：**按领域划分限界上下文**。

