# 面向理解编程（UOP）

用 AI 写代码的人越来越多。AI 写得快，但生成的代码遵循大厂主流规范，
对普通人来说极难 review，更难维护。代码能跑，但看着那些代码，
往往不知道 AI 为什么要这么写，自己也不敢改。

你应该遇到过这种情况：接手一个项目，代码风格很好，结构很清晰，但你就是不知道从哪里开始看。找了半天，终于在某个 `service` 文件夹里找到一个方法，点进去，是个接口，再找实现，在另一个文件夹。

不是代码写得差。是它写给了"懂行的人"，没写给"刚来的你"。

***

## 我们一直搞错了一件事

> **大多数代码都在迁就最懂的人，但维护它的往往是最普通的人。**

绝大多数编程规范首先服务的是**机器和专业开发者**，其次才轮到"人"。

UOP（Understandable Oriented Programming，面向理解编程）想把这个顺序彻底反过来。

如果零基础新人、产品经理、半年没碰代码的自己，都能大致看懂业务流程——这份代码才算及格。

***

## "那岂不是要写得很烂？"

几乎每个人第一次听到 UOP，都会有这个反应。

"为了让小白看懂，代码不就得写得啰嗦、低效、没法维护？"

**恰恰相反。**

真正成熟的 UOP 实践追求的是：更高的长期可维护性，更强的复用性与扩展性，更低的整体心智负担，更少的隐性知识。

它只是把"理解成本"放在了第一位。其他所有工程指标，都在这个前提下优化，不是被它牺牲。

排序变了，要求没有降。

***

## 面向理解编程 vs 面向对象编程：一张表看清本质差异

| 维度           | 主流面向对象编程 / Clean Code | UOP 面向理解编程                         |
| -------------- | ----------------------------- | ---------------------------------------- |
| 第一排序目标   | 可执行性、抽象质量、专业美学  | 任意背景的人的理解速度                   |
| 注释的使用     | 尽量少，只写"为什么"          | 大量尾随大白话注释，把复杂代码翻译成人话 |
| 紧凑写法的使用 | 鼓励                          | 鼓励，但必须配直白注释解释               |
| 函数长度甜区   | 3–15 行                       | 4–10 行                                  |
| 命名倾向       | 领域精确、适度简洁            | 初中英语常用词，宁可稍长也不让人猜       |
| 典型适用场景   | 中大型生产系统、框架、平台    | 中小型产品、跨职能协作、AI vibe coding   |

***

## UOP 的工程要求

规范可以很长，但执行靠的是记得住的东西。UOP 只有这几条：

### 命名用最常见的简单英语（看见即懂，不用想）
`add / make / get / find / show / list / update / remove`
用 `result` 不用 `metrics`，用 `get` 不用 `fetch`，用 `user` 不用 `usr`，用 `manager` 不用 `mgr`。宁可名字稍长，也不要让人猜。`id`、`api` 这类大家都懂的缩写除外。

函数名用"动词+名词"，一眼知道它在做什么事：
```python
sendEmail()       # 发邮件
checkLogin()      # 检查登录
getUserInfo()     # 获取用户信息
addBookToList()   # 把书加进列表
```

布尔值用疑问句，读起来像在问问题：
```python
isReady          # 准备好了吗
hasPermission    # 有没有权限
canEdit          # 能不能编辑
isLoggedIn       # 登录了吗
```

（大小写风格跟随语言惯例即可，Python 用 snake_case，JavaScript 用 camelCase。这里的示例统一用 camelCase 是为了更直观展示"动词+名词"结构，还有我个人比较喜欢阅读驼峰命名，实际写代码时以所用语言的惯例为准即可，可选情况下尽量使用驼峰命名，易读）

### 函数控制在 4–10 行，主次分明

包含空行和注释一起算。太短显得零碎，太长读起来累。4–10 行是甜区，不是硬性上限，偶尔超出没关系，但超过了就值得想想能不能拆。

一个函数要么是**统筹流程的主功能**，要么是**干脏活的辅助功能**，不要混在一起：

逻辑从上到下顺着读，不要绕回去，不要跳来跳去。

### 允许"紧凑写法"，但强制配尾随注释

一行搞定的写法没问题，但后面必须跟一句注释解释它在做什么，不然别人看不懂：

```python
# 筛选
valid_items = [item for item in items if item.price > 0 and item.stock > 0]
# 把那些价格是0或者没货的都去掉

# 排序
sorted_books = sorted(books, key=lambda b: b.borrowCount, reverse=True)
# 按借阅次数从多到少排，最热门的排第一

# 汇总
total = sum(order.price * order.qty for order in orders)
# 把每个订单的单价乘以数量，然后全部加起来
```

### 注释必须是大白话，解释人话

```python
# ❌ 废话注释，复述了代码
i = i + 1  # i 加 1

# ✅ 有用注释，解释业务意图
i = i + 1  # 跳到下一页

# ❌ 废话注释
user.status = 0  # 把 status 设为 0

# ✅ 有用注释
user.status = 0  # 封号，0 表示封禁状态

# ✅ 坑点注释，防止后人误删
time.sleep(0.2)  # 必须等 200ms，接口有限流，去掉会报 429 错误
await delay(100) # iOS 键盘收起有动画延迟，少了这个下面的滚动会位置不对
```

在比较长的函数里，用注释把代码分成几个段落，充当小标题，方便粗读：

```python
def processOrder(orderId):
    # 拿到订单数据
    order = getOrderById(orderId)
    items = getOrderItems(orderId)

    # 计算价格
    subtotal = calcSubtotal(items)
    discount = calcDiscount(order.userId)
    total = subtotal - discount

    # 通知用户
    sendConfirmEmail(order.userId, total)
    print("订单处理完成，总价：" + str(total))
```

### 文件按"人类能理解的大块"划分

```
user.py      # 所有跟用户有关的事
book.py      # 所有跟书有关的事
order.py     # 所有跟订单有关的事
report.py    # 所有报表和统计
utils.py     # 放最通用的工具函数
```

文件数量少于 4 个，直接平铺，不需要建子文件夹。超过了再按大类做一层分类，但嵌套不超过两层。不要建只有一个文件的文件夹。

### 视觉节奏很重要
功能段落之间空 1–2 行，相关的代码挨在一起，不相关的代码中间隔开：

```python
# ✅ 有节奏，一眼能分出"这里做了几件事"
def addBook(self, bookName, howMany):
    print("现在添加新书...")

    if bookName in self.books:
        self.books[bookName] += howMany
    else:
        self.books[bookName] = howMany

    print(bookName + " 加好了，现在有 " + str(self.books[bookName]) + " 本")


# ❌ 没有节奏
def addBook(self, bookName, howMany):
    print("现在添加新书...")
    if bookName in self.books:
        self.books[bookName] += howMany
    else:
        self.books[bookName] = howMany
    print(bookName + " 加好了，现在有 " + str(self.books[bookName]) + " 本")
```

不要把太多函数堆在同一个屏幕里，每个函数之间空两行，给眼睛一个喘息的地方。

### 提前阻断：卫语句前置（Early Return）

函数开头先把所有"不能继续"的情况处理掉，直接 return + 警告，不要往下走。这样主逻辑就不需要层层嵌套，缩进保持在两层以内。

❌ 嵌套版本：
```python
def borrow(self, bookName, userId):
    if bookName in self.books:
        if self.books[bookName] > 0:
            if userId in self.users:
                self.books[bookName] -= 1
                return True
```

✅ 卫语句版本：
```python
def borrow(self, bookName, userId):
    if bookName not in self.books:   # 没这本书，直接结束
        print("没有这本书：" + bookName)
        return False

    if self.books[bookName] <= 0:    # 借光了，直接结束
        print("借不了，已经借光了")
        return False

    if userId not in self.users:     # 用户不存在，直接结束
        print("用户不存在：" + userId)
        return False

    self.books[bookName] -= 1        # 走到这里说明什么问题都没有，可以借
    return True
```

### 复用克制：绝不提前抽象

未知的复用都是猜想。
```python
# ❌ 提前抽象，实际可能不会用第二次
def buildQueryParams(filters, sort, page):  # 猜测未来会复用这个
    ...

# ✅ 先直接写，等真的出现第二次再考虑抽取
def getUserList(page):
    # 直接写这里的查询逻辑
    ...

def getBookList(page):
    # 直接写这里的查询逻辑，等两边真的一样了再合并
    ...
```
一段代码在业务中真的出现了2次以上，再去抽取它吧。

***

## 同一个需求，两种写法对比

需求很简单：一个图书馆借书系统。新增用户、添加图书、借书、还书、查看所有书、生成报表。

先看文件结构。

**面向对象版本（主流大厂规范）**

```
library_system/
├── domain/
│   ├── user.py
│   └── book.py
├── repository/
│   ├── user_repository.py
│   └── book_repository.py
├── service/
│   ├── borrow_service.py
│   └── report_service.py
├── utils/
│   ├── logger.py
│   └── exceptions.py
├── main.py
├── config.py
└── ……
```

非常专业，模块化，但是非常简单的东西，用了 10+ 个文件，2–3 层文件夹。
新人打开项目的第一反应：*从哪里开始看？*

**面向理解编程版本（UOP规范）**

```
easyLibraryBook/
├── user.py      # 所有用户相关的操作
├── book.py      # 所有书相关的操作
├── report.py    # 所有报表相关的操作
└── main.py      # 编排整个程序
```

4 个文件，0 层嵌套。
新人打开项目的第一反应：*哦，我大概知道这个系统有什么了。*

***

再看核心代码。以"借书"这个功能为例：

**面向对象版本**

```python
# domain/book.py
from dataclasses import dataclass

@dataclass
class Book:
    name: str
    stock: int

# repository/book_repository.py
from typing import Optional

class BookRepository:
    def __init__(self):
        self._store: dict[str, Book] = {}

    def save(self, book: Book) -> None:
        self._store[book.name] = book

    def find_by_name(self, name: str) -> Optional[Book]:
        return self._store.get(name)

# service/borrow_service.py
class BorrowService:
    def __init__(self, repo: BookRepository):
        self._repo = repo

    def borrow(self, book_name: str) -> bool:
        book = self._repo.find_by_name(book_name)
        if not book or book.stock <= 0:
            raise ValueError("Book not available")
        book.stock -= 1
        self._repo.save(book)
        return True

# 调用方式
repo = BookRepository()
repo.save(Book(name="小王子", stock=3))
service = BorrowService(repo=repo)
service.borrow("小王子")
```

同样的借书功能，UOP 只需要一个文件："——给读者一个落脚点，再看代码。

**UOP 版本**

```python
# book.py —— 这个文件只管书的事

class Book:
    def __init__(self):
        self.books = {}                   # 用字典存所有书，key是书名

    def add(self, bookName, howMany):
        print("现在添加新书...")
        if bookName in self.books:
            self.books[bookName] += howMany
        else:
            self.books[bookName] = howMany
        print(bookName + " 加好了，现在有 " + str(self.books[bookName]) + " 本")

    def borrow(self, bookName):
        print("现在有人要借书...")

        if bookName not in self.books:    # 根本没这本书，直接结束
            print("没有这本书：" + bookName)
            return False

        if self.books[bookName] <= 0:     # 有这本书但借光了，直接结束
            print("借不了，" + bookName + " 已经借光了")
            return False

        self.books[bookName] -= 1         # 走到这里说明可以借，减一本
        print("借书成功！" + bookName + " 还剩 " + str(self.books[bookName]) + " 本")
        return True


# 调用方式就这么简单
book = Book()
book.add("小王子", 3)
book.borrow("小王子")
```

效果对比就出来了：

- 面向对象的写法，调用前要先搞清楚 `Book` / `BookRepository` / `BorrowService` 三个东西的关系，再手动把它们组装起来，最后才能借一本书，很严谨
- 面向理解编程UOP的写法，只需要 `book = Book()`，然后 `book.borrow("小王子")` 就可以借一本书，很直接

***

## 检验

UOP 不是降低代码质量的借口。判断一段代码是否达标，需要三个条件**同时成立**：

1. **代码片段截图丢群里**，不了解项目背景的人，能大致讲出这段代码在做什么
2. **中级开发者接手后**，修改和扩展的速度比接手"漂亮但难懂的代码"更快
3. **半年后原作者自己回来**，几乎不需要翻文档和问别人，就能快速定位和理解

反正大概就是随便拉一个编程基础很弱的人，让他看着代码和注释，能不能大概说出"这段代码在做什么"？

能，好代码。不能，重构。

***

## 最后

UOP 不是要否定面向对象编程、函数式编程、Clean Code 等实践方法。这些都是成熟且有价值的实践。

UOP 只是提醒我们一个最容易被遗忘的事实：

**代码写出来是给人审查的。人的心智有限。**

当我们把排序从"机器 → 高手 → 普通人"调整为"普通人 → 高手 → 机器"，很多长期困扰我们的维护性、可读性、协作效率问题，反而会自然而然地得到缓解。

***

在 AI 时代，生成代码的成本无限趋近于零，但理解代码的成本并没有降低。相反，由于 AI 产出的代码量爆炸，人类阅读和维护的负担其实在加重。

> 最高级的代码，不是只有天才才能看懂的代码。 而是随便拉来一个新人，都能自信地对它进行修改的代码。

再回到开头那个接手项目的你。

如果那个项目是用 UOP 写的，你打开文件夹，看到 book.py，点进去，看到borrow函数，就找到了。

不用猜从哪里看，不用跳来跳去找实现，不需要任何前置知识。

代码写给了"刚来的他"。

## 附录：给 AI 的提示词

如果你在用 AI 辅助编程，可以把下面这段直接复制到你的 AI 编码助手的系统提示词里。

```markdown
# 面向理解编程（UOP）代码生成规范

## 核心原则
把"任意背景的人能快速读懂"作为第一优先级。所有命名、结构、注释、文件组织决策，都以这个标准为准。

## 命名规则
函数名使用"动词+名词"结构：add / make / get / find / show / list / update / remove / check / send 是首选动词。
布尔变量写成疑问句形式：isReady、hasPermission、canEdit、isLoggedIn。
变量名使用完整常见英语单词：result、user、manager、config、index、items。
id、api 等约定俗成的缩写保留，其余一律写完整单词。
大小写风格跟随所用语言惯例，可选情况下优先使用驼峰命名，易读性高。
类名用名词，直接代表它管理的对象：Book、User、Order、Report。

## 函数规则
每个函数聚焦单一职责，长度以 4–10 行为甜区，超出时优先考虑拆分。
函数开头先用卫语句（Early Return）处理所有异常情况，每种情况加注释说明原因后直接 return，主逻辑写在所有卫语句之后。
缩进保持在两层以内，卫语句负责把嵌套压平。
一个函数要么做核心操作逻辑，要么做工具函数，两者分开写。
操作开始时用 print / console.log 打印正在做什么，操作完成后打印结果，让调用者能看到每一步发生了什么。
对同一对象操作的函数放在一起，用类封装，简化函数命名，类名即被操作对象名，调用方式保持最简：`book = Book(); book.add("UOP", 3); book.borrow("UOP")`。

## 注释规则
所有紧凑写法（列表推导、lambda、三元表达式、链式调用等）后面跟一行注释，用大白话解释这行在做什么、哪些参数可以改。
注释写业务意图和背后原因，解释"为什么这么做"。
代码行为看起来奇怪的地方（限流等待、延迟处理、魔法数字、特殊顺序等）必须加注释说明原因，保护不误删。
较长的函数用注释行做段落小标题，把函数分成几个语义段落，方便快速扫读，不需要逐行阅读。
文件注释写在文件开头，用大白话解释文件在做什么、有哪些方法可以被调用，怎么调用。

## 文件规则
一个文件只负责一类业务，文件名直接说明内容：如user.py、book.py、order.py；report.js、utils.js。
文件在 4 个以内时直接平铺，无需建子文件夹。
超过 4 个文件时，按业务大类建一层文件夹分组，嵌套不超过两层。
每个文件夹内至少放 3 个文件，禁止出现只有一个文件的文件夹。
utils 文件只放真正通用的工具函数，业务逻辑放到对应业务文件里。

## 视觉规则
功能段落之间空 1–2 行，相关代码紧靠在一起，不相关的代码之间留空行隔开。
每个函数之间空两行，给阅读者视觉喘息空间。
卫语句每条之间空一行，主逻辑前再空一行，视觉上明确区分卫语句和主逻辑。

## 复用规则
同一段逻辑在代码库中出现两次及以上时，再提取为公共函数。
第一次出现时直接内联书写，保持上下文完整可读。
提前为"可能复用"而抽象的函数一律不写，等需求真实出现再处理。
```