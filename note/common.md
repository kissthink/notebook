﻿mac
---
- crontab file 添加crontab；file需要自己编辑
- cmd + k 完整清屏
- alt + 方向，跨字符移动
- 使用 `sudo xattr -cr /Applications/xxx.app/` 可以避开mac的APP安全限制

代码规范
---
- 变量名由数据库字段映射，前后端使用尽量保持一致；
- 代码统一格式化，便于SVN对比；
- 一个类中：
属性顺序：const/public/protected/private；
方法顺序：增删改查顺序写；
- dao层尽量只使用save/delete/update/query或开头的名字；
- 配置信息尽量在类常量或静态变量中定义出来，由控制器导入到前端渲染；
- 常用函数尽量封装在一个常用类中，以静态函数形式。
- 有时可以多设置一个else来增强程序的健壮性，可以在else时来处理异常，不必抛出错误。

名词
---
- RPC Remote Procedure Call 远程过程调用
- ACK Acknowledgement 确认字符
- shebang  shell脚本最前面指定的解释器，以#!开头 如 #!/usr/bin （#!/usr/local/bin/php 可以限定使用PHP解析脚本）
- 鲁棒性：（Robustness）是指一个计算机系统在执行过程中处理错误，以及算法在遭遇输入、运算等异常时继续正常运行的能力。
- UGC (User Generated Content) 用户生成内容
- 回溯法：也称试探法，它的基本思想是：从问题的某一种状态（初始状态）出发，搜索从这种状态出发所能达到的所有“状态”，当一条路走到“尽头”的时候（不能再前进），再后退一步或若干步，从另一种可能“状态”出发，继续搜索，直到所有的“路径”（状态）都试探过。这种不断“前进”、不断“回溯”寻找解的方法，就称作“回溯法”。
- “蓝精灵命名法”：蓝精灵动漫中每个精灵说话时都会在前面带上自己的名字，引申为程序中大量带上一个模块的前缀来区别命令。
- DAU/MAU  Daily/Monthly Active User  用户日/月活
- 依赖倒置：避免依赖具体对象，而要依赖抽象。例如创建一个茶杯时，要考虑依赖杯子的抽象，不要只实现茶杯对象；
- 召回率和精度：两者都是用于分类检索和统计结果质量的度量值，召回率也叫查全率，是指结果数据占应有结果的比例；精度是指有效结果占结果的比例；
- CAP consistency available partition 帽子理论，三都不可能同时满足。

日志规范：
---
- 敏感操作记得加LOG；
- 对于一个大型脚本，最好专门封装一个记录日志的方法，传入每一个记录特定的字段。
- 为了便于查找问题或便于查找对应方法的日志，可对每一个大型模块添加一个模块名，给每个方法添加具体的标识，以方便查找。
- 在于外界交互时，最好记录接口输入和系统输出的原始信息，做到出了问题有信息可查。
- 八个等级的日志：debug、 info、 notice、 warning、 error、 critical、 alert 以及 emergency 。

map和reduce：
---
- map：将数组中的成员遍历处理，每次返回处理后的一个值，最后结果值为所有处理后值组成的多项数组；
- reduce：遍历数组成员，每次使用数组成员结合初始值处理，并将初始值返回，即使用上一次执行的结果，配合下一次的输入继续产生结果，结果值为 一项；

数据库锁 分布式锁 zookeeper
---
- mysql数据库实现分布式锁：
实现锁：数据表内的某一字段设为unique key 通过插入记录来加锁，通过删除记录来解锁。通过判断其mtime来实现超时时间，
避免同一操作者多次操作时自己锁死自己：添加锁定人字段，查询时判断锁定人是否是自己。
避免同一操作者多次操作时解锁可能影响到自己到后来的锁：添加计数器count字段，在count为0时才能删除锁。
解决数据库宕机无法解锁问题：使用主从复制，宕机时切换Master继续操作。

- redis缓存锁：
setnx 命令
解决超时问题：设置成功后用expire命令添加过期时间。
新版redis的set命令可以直接在不存在的时候设置属性和超时时间：set key 'value' [EX(超时：秒) || PX(超时：毫秒)] [NX(不存在时) || XX(存在时)]

- redis分布式锁
实现锁：应用竞争锁多台redis服务器，当得到超过一半的服务器(不可能两个应用同时得到锁)时才得到锁；
解决死锁：在竞争锁失败时依次解开每台redis服务器，然后重新竞争。
设置唯一ID，只有设置成功了唯一ID的应用才能得到锁。

- Zookeeper分布式锁：
zookeeper是一个拥有多个节点分布式协调服务。对zookeeper写入请求会转发到leader，leader写入完成，并同步到其他节点，直到所有节点都写入完成，才返回客户端写入成功。
zookeeper支持watcher机制，这样实现阻塞锁，可以watch锁数据，等到数据被删除，zookeeper会通知客户端去重新竞争锁。
zookeeper的数据可以支持临时节点的概念，即客户端写入的数据是临时数据，在客户端宕机后，临时数据会被删除，这样就实现了锁的异常释放。使用这样的方式，就不需要给锁增加超时自动释放的特性了。
zookeeper实现锁的方式是客户端一起竞争写某条数据，比如/path/lock，只有第一个客户端能写入成功，其他的客户端都会写入失败。写入成功的客户端就获得了锁，写入失败的客户端，注册watch事件，等待锁的释放，从而继续竞争该锁。
如果要实现tryLock，那么竞争失败就直接返回false即可。

项目分层：
---
- Service层用来处理复杂的业务逻辑，只关心需求数据的结构等(WHAT)，对数据进行分析处理，将业务逻辑结果返回；
- Data层用来路由数据获取途径，只关心从哪里获取(WHERE)，负责挑选数据源，将不同数据源的数据结合返回给上层；
- Dao层用来与数据库进行直接对接，只关心如何获取(HOW)，负责组织数据获取方式，并将取到的数据直接返回上层；
在网站开发中，Service Data Dao层共同构成Model层。

安全相关：
---
- CSRF 跨站点请求伪造（Cross Site Request Forgery），访问有害网站时会获取到访问正常网站时的cookie，从而获取身份信息，然后用此身份信息，请求正常网站；
预防：
       正常网站安全方面使用POST方式提交，获取时同样使用$_POST获取POST信息，对每个POST表单添加上一个随机token，存入session里，每次收到post请求时较验session里的token；
       验证码
        referer_check
- XSS 跨站点脚本攻击（Cross Site Script）
预防：
       输入检测与转义
- SQL注入
预防：
        PDO
        输入检测

面向对象的六大设计原则：
---
1、单一职责原则：避免职责分散，避免承担太多（SRP）
2、开闭原则：模块应对扩展开放，而对修改关闭（OCP）。
3、里氏代换原则：子类必须能替换掉父类（LSP）。
4、依赖倒转原则：父类不依赖子类，抽象不依赖具体（DIP）
5、接口隔离原则：职业单一，承诺最简（ISP）
6、组合复用原则：尽量使用组合，避免滥用继承（CRP）

RESTFUL API
---
1. 主体针对资源本身，而不是操作。
2. 其URI表示资源本身,用queryString来过滤掉部分资源;
3. GET/POST/PUT/DELETE/PATCH(更新部分)/HEAD(请求资源的元数据（文件属性）)/OPTIONS(用于探测：请求以确定针对某个目标地址的请求必须具有怎样的规则，然后根据规则用其他方法请求)基本方法来访问URI；
4. 状态转移放在客户端，服务器端只提供特定的接口。

Git
---
- Git保存的不是文件的变动，而是保存了每次`修改前后文件`的整体内容为一个 `blob`；
- Git通过`指针`的形式，指向每次修改的文件，表示一个版本。
- 每一个文件的不同版本，都有一个40位的sha1校验和。

搜索
---
- 使用 `“this is key word” `来搜索字符串，避免字符串的自动分词；
- 使用 - 来排除搜索； `a - b `搜索符合a，而不带有b的内容；
- 使用 `intitle:keyword `搜索标题中的关键词；
- 使用 `keyword site:domain.com` 来搜索domain.com内的keyword；
- 使用 `keyword filetype:type` 来搜索文件类型为type的keyword；

RPC
---
RPC 的全称是 Remote Procedure Call 是一种进程间通信方式。 它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，本质上编写的调用代码基本相同。
RPC 的主要目标是让构建分布式计算（应用）更容易，在提供强大的远程调用能力时不损失本地调用的语义简洁性。

分类：
- 同步调用：客户端等待调用执行完成并获取到执行结果。
- 异步调用：客户端调用后不用等待执行结果返回，但依然可以通过回调通知等方式获取返回结果。若客户端不关心调用返回结果，则变成单向异步调用，单向调用不用返回结果。

主要结构：
1. RpcServer 负责导出（export）远程接口：导出是指暴露远程接口的意思，只有导出的接口可以供远程调用。
2. RpcClient 负责导入（import）远程接口的代理实现：导入相对于导出而言，客户端代码为了能够发起调用必须要获得远程接口的方法或过程定义。
3. RpcProxy 远程接口的代理实现：协议指 RPC 调用在网络传输中约定的数据封装方式，包括三个部分：编解码、消息头 和 消息体。

    调用编码：
    - 接口方法 包括接口名、方法名
    - 方法参数 包括参数类型、参数值
    - 调用属性 包括调用属性信息，例如调用附加的隐式参数、调用超时时间等
   返回编码：
    - 返回结果 接口方法中定义的返回值
    - 返回码 异常返回码
    - 返回异常信息 调用异常信息
4. RpcInvoker
客户端：负责编码调用信息和发送调用请求到服务端并等待调用结果返回
服务端：负责调用服务端接口的具体实现并返回调用结果
5. RpcProtocol 负责协议编/解码
6. RpcConnector 负责维持客户端和服务端的连接通道和发送数据到服务端
7. RpcAcceptor 负责接收客户端请求并返回请求结果
8. RpcProcessor 负责在服务端控制调用过程，包括管理调用线程池、超时时间等
9. RpcChannel 数据传输通道

垃圾回收(GC)
---
垃圾（Garbage）就是程序需要回收的对象，如果一个对象不在被直接或间接地引用，那么这个对象就成为了「垃圾」，它占用的内存需要及时地释放，否则就会引起「内存泄露」。有些语言需要程序员来手动释放内存（回收垃圾），有些语言有垃圾回收机制（GC）。

1. 标记清除法
    过程需要两次扫描，扫描从根开始，第一次对对象标记，第二次执行清除。
    第一次扫描时被根引用的标记为非垃圾，接着递归扫描非垃圾所引用的对象，直至结束。
    第二次扫描将没有“非垃圾”标记的对象清除。
    两次扫描耗时较长。
2. 复制收集法
    过程只需要一次扫描，但需要额外内存来保存非垃圾对象
    从根开始扫描，将非垃圾对象复制收集到一块新的对象空间，然后将旧空间的数据全部清除；
    一次扫描速度快，但需要额外的内存空间。
3. 引用计数
    给每个对象添加一个引用数的值，扫描后将引用数为0的对象清除
    可能会有循环引用导致对象无法被回收的状况，而且在并行时，操作引用数的值锁开销会很大。

线程安全
---
fork()进程时，子进程会获取父进程的整个虚存空间的拷贝，但是只能获取到执行fork()命令的那一个线程的拷贝。
如果父进程有多个线程，且某一线程持有未释放的素质锁，在fork()之后，只保留了一个线程而无法释放互斥锁，会形成死锁。

在多线程环境下，执行 fork() 函数是不安全的。也因此，必须慎重使用多进程和多线程混搭的模型。


测试
---
Mock : 模拟和操纵，一个 mock 对象可以根据自己的意愿，模拟出不同的行为和表现。
Stub : 桩, 一个 stub 对象可以接收请求信息，但不会做出任务反应，只是用来表示存在。

内存溢出和内存泄漏
---
内存溢出是指内存不够用了，申请内存时报`Out of Memory`错误；
内存泄漏是指分配出去的内存不再使用，且无法回收。