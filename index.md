## 普通接口
一般为GET请求，比如获取新闻列表 GET http://Example.com/index.php?module=news&action=list，为了防止采集或者暴力查询，我们PC端一般做如下处理:
`1.防止本站被它站file_get_contents，所以要识别user_agent，如果不是通过浏览器来访问的话直接不给看。`
`2.如果别人通过伪造user_agent来访问的话，就通过单位时间ip的访问量来控制抓取方，可以写一套算法，如果再一个ip在前后一分钟多于多少次访问量来处理。但是，会有一种情况，即某个小区或公司内都是使用某一个IP的外网的话，这样搞就会自寻死路，所以还要配合浏览器中的cookie来处理`
总结: 请求头可以伪造，IP地址可以变更，cookie可以清空，基本上PC端是很难防这个问题的，比如淘宝，点评等大站的数据我也是经常去采的。


### Markdown

那APP端如何处理这个问题呢?我们可以抓点评APP的http请求包来看一下:

GET http://114.80.165.113/mapi/ugcuserfeeds.bin?filtertype=5&userid=129059048&token=73114c7e9a4485319542039cdff854d989f61e5821d306b3abf0fc9904eb51ff&start=0 HTTP/1.1
Host: 114.80.165.113
Accept: */*
pragma-appid: 351091731
pragma-newtoken: c2032338f6abf96c8e2984db1655f2bac73b88f799e49aab4a426d414f994b5f
pragma-token: 73114c7e9a4485319542039cdff854d989f61e5821d306b3abf0fc9904eb51ff
pragma-dpid: 9214560561001942797
pragma-device: 566fe5aeb75a827967fbad8356608134ba98a4a6
Proxy-Connection: keep-alive
pragma-os: MApi 1.1 (dpscope 7.5.0 appstore; iPhone 8.3 iPhone7,1; a0d0)
Accept-Language: zh-cn
network-type: wifi
User-Agent: MApi 1.1 (dpscope 7.5.0 appstore; iPhone 8.3 iPhone7,1; a0d0) Paros/3.2.13 
当你直接访问http://114.80.165.113/mapi/ugcuserfeeds.bin?filtertype=5&userid=129059048&token=73114c7e9a4485319542039cdff854d989f61e5821d306b3abf0fc9904eb51ff&start=0的时候,直接从服务器端给挡住,并返回450错误;

`PHP的服务器一般为Apache或Nignx,我们也可以在配置项中根据与客户端开发人员约定的一些自定义的Request头信息,比如上面的parama-*,在服务器配置项中可以获取到这些自定义的Request头信息,然后根据是否为约定好的Request信息,如果不是就rewrite到450`;

但是,我们通过抓包既可获得全部请求头信息,这时可以完全模拟请求头信息来获取数据;
很多APP最多到此步既可获得该API接口的数据,而且是非常便于处理的json格式,而点评APP到此处直接返回的是一堆看上去是经过`压缩的乱码数据`

# 表单接口
用验证码,pc端存session,app端存在cache
# 会员接口
- 与APP端开发人员约定特定的md5组合算法，然后两端比对一下，`如果相同就allow，不相同就deny`；
**不安全的，如果APP程序被反编译，这些约定的算法就会暴露，特别是在安卓APP中，有了算法，完全就可以模拟接口请求通过验证；**
- `数据库会员表的password是带上了随机密窜并经过双重加密的md5值;在用户登录的时候,我返回会员相应的uid和password`,password虽然是明文的,别人知道也不能登录,毕竟是经过加密的,然后每次请求接口的时候user_id=333&token=aa37e10c7137ac849eab8a2d5020568f,`通过主键uid可以很快的找到当前uid对应的token`,然后再来比对;
**一旦知道了这个token,除非用户更改密码,否则也可以一直通过这个token来操作该会员的相关接口;**

# 对称加密
- 通过对称加密算法，该加密算法对`uid+网站公钥`进行时效加密，在一定时效内可用。在会员登录成功时，`服务器端对该ID加密后返回给客户端，客户端每次请求接口的时候带上该参数，服务器端通过解密认证`；
** 防外不防内 **


# 用token
- 会员登录的时候请求登录接口，然后服务器端返回给客户端一个token，该token生成的规则是 `网站公钥 + 当前uid + 当前时间戳 + 一段随机数双重加密`，根据需求决定是把该token放进cache等一段时间自动失效，还是放进数据库（如果要放进数据库的话，单独拎出一张表来，顺便记录用户的登录，登出时间），`在用户登录的时候改变一下，确保该token只能在用户人为登出登录之间有用`。
`为保安全，应保证让用户在一段时间内自动退出`；此方案配合Linux和数据库的权限管理可以防外又防内；

