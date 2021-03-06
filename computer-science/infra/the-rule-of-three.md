# 软件工程中的 “3” 的规则


wp_id: 550
Status: publish
Date: 2018-06-23 02:48:00
Modified: 2020-05-16 11:14:37


我注意到了一个神奇的软件工程法则：在你正确地解决问题之前，你至少需要3个例子。

具体说来是这样的：

1. 不要试图在两个类之间共享代码，至少等到你有三个类的时候。
2. 解决问题的前两次尝试一定会失败，因为你还没完全理解这个问题。第三次才行
3. 任何想要早期就能设计好的尝试都会导致对于巧合情形的过度拟合。

# 你在说什么？请给个例子

比如说你在实现一个类，从银行抓取数据。下面是一个非常傻瓜的版本，但是应该说明了问题：

```py
class ChaseScraper:
    def __init__(self, username, password):
        self._username = username
        self._password = password

    def scrape(self):
    	session = requests.Session()
        sessions.get("https://chase.com/rest/login.aspx",
	             data={"username": self._username,
		           "password": self._password})
	sessions.get("https://chase.com/rest/download_current_statement.aspx")
```

现在你想添加第二个类 CitiBankScraper 来实现相同的接口，但是改变了一些实现细节。实际上假设CitiBank只是有一个不同的 url 和表单元素名称而已。让我们来添加一个新的爬虫：

```py
class CitibankScraper:
    def __init__(self, username, password):
        self._username = username
        self._password = password

    def scrape(self):
    	session = requests.Session()
        sessions.get("https://citibank.com/cgi-bin/login.pl",
	             data={"user": self._username,
		           "pass": self._password})
        sessions.get("https://citibank.com/cgi-bin/download-stmt.pl")
```

因为经过了多年DRY原则的教育，这时候我们发现这两个类的代码几乎是重复的！我们应该重构一下，把所有的重复代码都放到一个基类中。在这里，我们需要Inserve of Control 模式，让基类来控制逻辑。

```py
class BaseScraper:
    def __init__(self, username, password):
        self._username = username
        self._password = password

    def scrape(self):
    	session = requests.Session()
        sessions.get(self._LOGIN_URL,
	             data={self._USERNAME_FORM_KEY: self._username,
		           self._PASSWORD_FORM_KEY: self._password})
        sessions.get(self._STATEMENT_URL)


class ChaseScraper(BaseScraper):
    _LOGIN_URL = "https://chase.com/rest/login.aspx"
    _STATEMENT_URL = "https://chase.com/rest/download_current_statement.aspx"
    _USERNAME_FORM_KEY = "username"
    _PASSWORD_FORM_KEY = "password"


class CitibankScraper(BaseScraper):
    _LOGIN_URL = "https://citibank.com/cgi-bin/login.pl"
    _STATEMENT_URL = "https://citibank.com/cgi-bin/download-stmt.pl"
    _USERNAME_FORM_KEY = "user"
    _PASSWORD_FORM_KEY = "pass"
```

这应该让我们删掉了不少代码。这已经是最简单的方法之一了。所以问题在哪里呢？（出去我们实现继承的方法不好之外）

问题是我们过度拟合了！过度拟合是什么意思呢？我们正在抽象出并不能很好泛化的模式！

![facepalm](https://erikbern.com/assets/facepalm.jpg)

为了验证这一点，假设我们又需要从第三个银行抓取数据。也许它需要如下几点：

- 他需要两步验证
- 密码是使用 JSON 传递的
- 登录使用了POST而不是GET
- 需要同时访问多个页面
- 要访问的url是根据当前日期动态生成的


…… 或者随便什么东西，有1000中方式让我们的代码不能工作。我希望你已经感觉到问题所在了。我们以为我们通过前两个爬虫发现了一个模式！然鹅悲剧的是，我们的爬虫根本不能泛化到第三个银行（或者更多，第n个）。也就是说，我们过拟合了。


# 过拟合到底是什么意思？

过拟合指的是我们在数据发现了一个模式，但是这个模式并不能很好地泛化。当我们在写代码的时候，我们经常对于优化代码重复非常警觉，我们会发现一些偶然出现的模式，但是如果我们查看整个程序的话，我们知道这些模式可能并不能很好地代表整个程序的模式。所以当我们实现了两个银行的爬虫之后，我们以为我们发现了一个广泛的模式，实际上并不是。

注意到，代码重复并不总是一件坏事。工程师们通常过分关注减少重复代码，但是也应该注意区分偶然的代码重复和系统性的代码重复之间的区别。

因此，让我来引入第一个 “3” 之规则。如果你只有两个类或者对象，不要过分关注代码重复。当你在三个不同的地方看到同一个模式的时候在考虑如何重构。


# “3” 之规则应用到架构上

同样的推理可以应用到系统设计上，但是会得出一个非常不同的结论。当你从头构建一个新的系统的时候，你不知道他最终会被如何使用，不要被假设所限制。在第一代和第二代产品上，我们认为需要的限制可能真的是需要的，但是当实现第三代产品的时候，我们会发现假设是完全错误的，并最终实现正确的版本。

比如说，Luigi就是解决问题的第三次尝试。前两个尝试解决了错误的问题，并且为错误的方向做了优化。比如第一个版本依赖于在 XML 中设计依赖图。但是很显然这是非常不友好的，因为你一般县要在代码里生成依赖图比较好。而且，在前两次设计中看起来很有用的一些新设计，比如任务解耦输出，最终只给一些非常少见的例子添加了支持，但是有添加了不少复杂度。

第一个版本中看起来很奇怪的问题可能在后来是很重要的问题，反过来也是。



I was reminded of this when we built an email ingestion system at Better. The first attempt failed because we built it in a poor way (basically shoehorning it into a CRUD request). The second one had a solid microservice design but failed for usability reasons (we built a product that no one really asked for). We’re halfway through the third attempt and I’m having a good feeling about it.

这个故事告诉了我们第二个 “3” 之规则——在系统设计上，直到第三次你才能够做对。


更重要的是，如果你的第一版有一些奇怪的位置问题，不要假设你需要搞定他们。走捷径。绕开奇怪的问题。估计你也不会运行这个系统很长时间——总有一天他会坏的。第二个版本大多数时候也是坏的。第三个版本值得你把它雕琢到完美。

![three cupcakes](https://erikbern.com/assets/three-cupcakes.jpg)

原文：https://erikbern.com/amp/2017/08/29/the-software-engineering-rule-of-3.html