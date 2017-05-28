* [x] 快速入门
* [x] 一个最小的应用
* [x] 调试模式
* [x] 路由
     * [x] 变量规则
     * [x] 构造 URL
     * [x] HTTP 方法
* [x] 静态文件
* [x] 模板渲染
* [x] 访问请求数据
     * [x] 环境局部变量
     * [x] 请求对象
     * [x] 文件上传
     * [x] Cookies
* [x] 重定向和错误
* [x] 关于响应
* [x] 会话
* [x] 消息闪现
* [x] 日志记录
* [x] 整合 WSGI 中间件
* [x] 部署到 Web 服务器

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```
把它保存为 hello.py （或是类似的），然后用 Python 解释器来运行。 确保你的应用文件名不是 flask.py ，因为这将与 Flask 本身冲突。
那么，这段代码做了什么？

1. 首先，我们导入了 Flask 类。这个类的实例将会是我们的 WSGI 应用程序。
2. 接下来，我们创建一个该类的实例，第一个参数是应用模块或者包的名称。 如果你使用单一的模块（如本例），你应该使用 __name__ ，因为模块的名称将会因其作为单独应用启动还是作为模块导入而有不同（ 也即是 '__main__' 或实际的导入名）。这是必须的，这样 Flask 才知道到哪去找模板、静态文件等等。详情见 Flask 的文档。
3. 然后，我们使用 route() 装饰器告诉 Flask 什么样的URL 能触发我们的函数。
4. 这个函数的名字也在生成 URL 时被特定的函数采用，这个函数返回我们想要显示在用户浏览器中的信息。
5. 最后我们用 run() 函数来让应用运行在本地服务器上。 其中 if __name__ == '__main__': 确保服务器只会在该脚本被 Python 解释器直接执行的时候才会运行，而不是作为模块导入的时候。
欲关闭服务器，按 Ctrl+C。

调试模式
虽然 run() 方法适用于启动本地的开发服务器，但是你每次修改代码后都要手动重启它。这样并不够优雅，而且 Flask 可以做到更好。如果你启用了调试支持，服务器会在代码修改后自动重新载入，并在发生错误时提供一个相当有用的调试器。
有两种途径来启用调试模式。一种是直接在应用对象上设置:
```
app.debug = True
app.run()
或者
app.run(debug=True)
```
注意

尽管交互式调试器在允许 fork 的环境中无法正常使用（也即在生产服务器上正常使用几乎是不可能的），但它依然允许执行任意代码。这使它成为一个巨大的安全隐患，因此它 绝对不能用于生产环境 。

路由

```
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello World'
```
变量规则

要给 URL 添加变量部分，你可以把这些特殊的字段标记为 <variable_name> ， 这个部分将会作为命名参数传递到你的函数。规则可以用 <converter:variable_name> 指定一个可选的转换器。这里有一些不错的例子:
```
@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return 'User %s' % username

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return 'Post %d' % post_id
```

HTTP 方法

HTTP （与 Web 应用会话的协议）有许多不同的访问 URL 方法。默认情况下，路由只回应 GET 请求，但是通过 route() 装饰器传递 methods 参数可以改变这个行为。这里有一些例子:
```
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        do_the_login()
    else:
        show_the_login_form()
```
如果存在 GET ，那么也会替你自动地添加 HEAD，无需干预。它会确保遵照 HTTP RFC （描述 HTTP 协议的文档）处理 HEAD 请求，所以你可以完全忽略这部分的 HTTP 规范。同样，自从 Flask 0.6 起， 也实现了 OPTIONS 的自动处理。

你不知道一个 HTTP 方法是什么？不必担心，这里会简要介绍 HTTP 方法和它们为什么重要：

HTTP 方法（也经常被叫做“谓词”）告知服务器，客户端想对请求的页面 做 些什么。下面的都是非常常见的方法：

GET

浏览器告知服务器：只 获取 页面上的信息并发给我。这是最常用的方法。

HEAD

浏览器告诉服务器：欲获取信息，但是只关心 消息头 。应用应像处理 GET 请求一样来处理它，但是不分发实际内容。在 Flask 中你完全无需 人工 干预，底层的 Werkzeug 库已经替你打点好了。

POST

浏览器告诉服务器：想在 URL 上 发布 新信息。并且，服务器必须确保 数据已存储且仅存储一次。这是 HTML 表单通常发送数据到服务器的方法。

PUT

类似 POST 但是服务器可能触发了存储过程多次，多次覆盖掉旧值。你可 能会问这有什么用，当然这是有原因的。考虑到传输中连接可能会丢失，在 这种 情况下浏览器和服务器之间的系统可能安全地第二次接收请求，而 不破坏其它东西。因为 POST 它只触发一次，所以用 POST 是不可能的。

DELETE

删除给定位置的信息。

OPTIONS

给客户端提供一个敏捷的途径来弄清这个 URL 支持哪些 HTTP 方法。 从 Flask 0.6 开始，实现了自动处理。

有趣的是，在 HTML4 和 XHTML1 中，表单只能以 GET 和 POST 方法提交到服务器。但是 JavaScript 和未来的 HTML 标准允许你使用其它所有的方法。此外，HTTP 最近变得相当流行，浏览器不再是唯一的 HTTP 客户端。比如，许多版本控制系统就在使用 HTTP。

静态文件

动态 web 应用也会需要静态文件，通常是 CSS 和 JavaScript 文件。理想状况下， 你已经配置好 Web 服务器来提供静态文件，但是在开发中，Flask 也可以做到。 只要在你的包中或是模块的所在目录中创建一个名为 static 的文件夹，在应用中使用 /static 即可访问。

给静态文件生成 URL ，使用特殊的 'static' 端点名:
```
url_for('static', filename='style.css')
```
这个文件应该存储在文件系统上的 static/style.css 。

模板渲染

用 Python 生成 HTML 十分无趣，而且相当繁琐，因为你必须手动对 HTML 做转义来保证应用的安全。为此，Flask 配备了 Jinja2 模板引擎。

你可以使用 render_template() 方法来渲染模板。你需要做的一切就是将模板名和你想作为关键字的参数传入模板的变量。这里有一个展示如何渲染模板的简例:
```
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```
Flask 会在 templates 文件夹里寻找模板。所以，如果你的应用是个模块，这个文件夹应该与模块同级；如果它是一个包，那么这个文件夹作为包的子目录:

情况 1: 模块:
```
/application.py
/templates
    /hello.html
```
情况 2: 包:
```
/application
    /__init__.py
    /templates
        /hello.html
```
关于模板，你可以发挥 Jinja2 模板的全部实例。更多信息请见 Jinja2 模板文档 。

文件上传

用 Flask 处理文件上传很简单。只要确保你没忘记在 HTML 表单中设置 enctype="multipart/form-data" 属性，不然你的浏览器根本不会发送文件。

已上传的文件存储在内存或是文件系统中一个临时的位置。你可以通过请求对象的 files 属性访问它们。每个上传的文件都会存储在这个字典里。它表现近乎为一个标准的 Python file 对象，但它还有一个 save() 方法，这个方法允许你把文件保存到服务器的文件系统上。这里是一个用它保存文件的例子:
```
from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
    ...
```
如果你想知道上传前文件在客户端的文件名是什么，你可以访问 filename 属性。但请记住， 永远不要信任这个值，这个值是可以伪造的。如果你要把文件按客户端提供的文件名存储在服务器上，那么请把它传递给 Werkzeug 提供的 secure_filename() 函数:
```
from flask import request
from werkzeug import secure_filename

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/' + secure_filename(f.filename))
    ...
```
Cookies

你可以通过 cookies 属性来访问 Cookies，用响应对象的 set_cookie 方法来设置 Cookies。请求对象的 cookies 属性是一个内容为客户端提交的所有 Cookies 的字典。如果你想使用会话，请不要直接使用 Cookies，请参考 会话 一节。在 Flask 中，已经注意处理了一些 Cookies 安全细节。

读取 cookies:
```
from flask import request

@app.route('/')
def index():
    username = request.cookies.get('username')
    # use cookies.get(key) instead of cookies[key] to not get a
    # KeyError if the cookie is missing.
 ```
 存储 cookies:
 ```
 from flask import make_response

@app.route('/')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp
 ```
 可注意到的是，Cookies 是设置在响应对象上的。由于通常视图函数只是返回字符串，之后 Flask 将字符串转换为响应对象。如果你要显式地转换，你可以使用 make_response() 函数然后再进行修改。
 
 重定向和错误
 
 你可以用 redirect() 函数把用户重定向到其它地方。放弃请求并返回错误代码，用 abort() 函数。这里是一个它们如何使用的例子:
 ```
  from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
```
这是一个相当无意义的例子因为用户会从主页重定向到一个不能访问的页面 （401 意味着禁止访问），但是它展示了重定向是如何工作的。

默认情况下，错误代码会显示一个黑白的错误页面。如果你要定制错误页面， 可以使用 errorhandler() 装饰器:
```
from flask import render_template

@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```
注意 render_template() 调用之后的 404 。这告诉 Flask，该页的错误代码是 404 ，即没有找到。默认为 200，也就是一切正常。

关于响应

视图函数的返回值会被自动转换为一个响应对象。如果返回值是一个字符串， 它被转换为该字符串为主体的、状态码为 200 OK``的 、 MIME 类型是 ``text/html 的响应对象。Flask 把返回值转换为响应对象的逻辑是这样：

  1.如果返回的是一个合法的响应对象，它会从视图直接返回。
  2.如果返回的是一个字符串，响应对象会用字符串数据和默认参数创建。
  3.如果返回的是一个元组，且元组中的元素可以提供额外的信息。这样的元组必须是 (response, status, headers) 的形式，且至少包含一个元素。 status 值会覆盖状态代码， headers 可以是一个列表或字典，作为额外的消息标头值。
  4.如果上述条件均不满足， Flask 会假设返回值是一个合法的 WSGI 应用程序，并转换为一个请求对象。

会话

除请求对象之外，还有一个 session 对象。它允许你在不同请求间存储特定用户的信息。它是在 Cookies 的基础上实现的，并且对 Cookies 进行密钥签名。这意味着用户可以查看你 Cookie 的内容，但却不能修改它，除非用户知道签名的密钥。

要使用会话，你需要设置一个密钥。这里介绍会话如何工作:
```
from flask import Flask, session, redirect, url_for, escape, request

app = Flask(__name__)

@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form action="" method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))

# set the secret key.  keep this really secret:
app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'
```
日志记录

有时候你会处于这样一种境地，你处理的数据本应该是正确的，但实际上不是。 比如，你会有一些向服务器发送请求的客户端代码，但请求显然是畸形的。这可能是用户篡改了数据，或是客户端代码的粗制滥造。大多数情况下，正常地返回 400 Bad Request 就可以了，但是有时候不能这么做，并且要让代码继续运行。

你可能依然想要记录下，是什么不对劲。这时日志记录就派上了用场。从 Flask 0.3 开始，Flask 就已经预置了日志系统。

这里有一些调用日志记录的例子:
```
app.logger.debug('A value for debugging')
app.logger.warning('A warning occurred (%d apples)', 42)
app.logger.error('An error occurred')
```
部署到 Web 服务器

