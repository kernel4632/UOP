# 面向理解编程（UOP）

你应该遇到过这种情况：接手一个项目，代码风格很好，结构很清晰，但你就是不知道从哪里开始看。找了半天，终于在某个 `service` 文件夹里找到一个方法，点进去，是个接口，再找实现，在另一个文件夹。

&#x20;

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

**1. 命名用最常见的简单英语**
`add / make / get / find / show / list / update / remove`
(反正不要过度领域化，除非那个词本身就是最通俗的说法)

**2. 函数控制在 4–10 行**
包含空行和注释。目的只有一个：制造节奏感，不让读者的看晕。

**3. 允许"紧凑写法"，但强制配尾随注释**

```python
valid_items = [item for item in items if item.price > 0 and item.stock > 0]
# 把那些价格是0或者没货的都去掉
```

**4. 注释必须是大白话**
不写"过滤无效记录"，写"把那些价格是 0 或者没货的都去掉"。
不写"计算折扣后总价"，写"先打完折再把所有东西加起来"。

**5. 文件按"人类能理解的大块"划分**
`user.py` / `book.py` / `order.py` / `report.py` / `utils.py`
一个文件名，一秒钟知道里面装的是什么。

**6. 视觉节奏很重要**
功能段落之间空 1–2 行。重要步骤前加一行 `print` 或注释标题。不要把十几个小函数塞在同一个屏幕里，毕竟人的注意力有限。

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

## 三个同时成立，才算及格

UOP 不是降低代码质量的借口。判断一段代码是否达标，需要三个条件**同时成立**：

1. **代码片段截图丢群里**，不了解项目背景的人，能大致讲出这段代码在做什么
2. **中级开发者接手后**，修改和扩展的速度比接手"漂亮但难懂的代码"更快
3. **半年后原作者自己回来**，几乎不需要翻文档和问别人，就能快速定位和理解


***

## 最后

UOP 不是要否定面向对象编程、函数式编程、Clean Code 等实践方法。这些都是成熟且有价值的实践。

UOP 只是提醒我们一个最容易被遗忘的事实：

**代码写出来是给人审查的。人的心智有限。**

当我们把排序从"机器 → 高手 → 普通人"调整为"普通人 → 高手 → 机器"，很多长期困扰我们的维护性、可读性、协作效率问题，反而会自然而然地得到缓解。

***

回到开头那个接手项目的人。

如果那个项目是用 UOP 写的，他打开文件夹，看到 book.py，点进去，看到borrow函数，找到了。

不用猜从哪里看，不用跳来跳去找实现，不需要任何前置知识。

代码写给了"刚来的他"。
