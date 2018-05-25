使用Elasticsearch实现全文搜索

### 一，安装Elasticsearch服务

​	[Ubantu安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html) 

### 二，安装python的Elasticsearch 客户端

​		`(venv) $ pip install elasticsearch`

### 三，ELasticsearch教程

为了熟悉Elasticsearch，我们将在python shell中使用Elasticsearch进行一些基本的操作。

创建   ` Elasticsearch`类的实例，将连接Elasticsearch服务端的的URL作为参数传入

```python
>>> from elasticsearch import Elasticsearch
>>> es = Elasticsearch('http://localhost:9200')
```

 在Elasticsearch中数据是被写入indexes中，和关系型数据库不同的是， 数据格式只能是JSON对象。下面的列子中将一个对象写入，字段名为 `text` ，索引名位 `my_index` 。

```python
>>> es.index(index='my_index', doc_type='my_index', id=1, body={'text': 'this is a test'})
```

如果需要，索引可以存储不同类型的文档，在这种情况下，可以根据这些不同的格式将 `doc_type` 参数设置为不同的值。我将要存储具有相同格式的所有文档，因此我将文档类型设置为索引名称。

对于存储的每个文档，Elasticsearch都使用唯一的ID和JSON对象与数据

让我们存储第二条数据，

```python
>>> es.index(index='my_index', doc_type='my_index', id=2, body={'text': 'a second test'})
```

现在，该索引中有两个文档，可以进行自由的搜索。在这个例子中，搜索关键词 ` this test`：

```python
>>> es.search(index='my_index', doc_type='my_index',
... body={'query': {'match': {'text': 'this test'}}})
```

`es.search` 返回的结果是Python中的字典：

```python
{
    'took': 1,
    'timed_out': False,
    '_shards': {'total': 5, 'successful': 5, 'skipped': 0, 'failed': 0},
    'hits': {
        'total': 2, 
        'max_score': 0.5753642, 
        'hits': [
            {
                '_index': 'my_index',
                '_type': 'my_index',
                '_id': '1',
                '_score': 0.5753642,
                '_source': {'text': 'this is a test'}
            },
            {
                '_index': 'my_index',
                '_type': 'my_index',
                '_id': '2',
                '_score': 0.25316024,
                '_source': {'text': 'a second test'}
            }
        ]
    }
}
```

可以看出搜索的结果是两个文档，每个文档都有一个分数，分数最高的文档包含搜索关键词的两个单词，而另一个文档只包含一个单词。你可以看到，即使是最好的结果也没有很好的分数，因为这些单词与文本不完全一致。

现在，如果搜索 ` secondg` 关键词，结果就是这样：

```python
>>> es.search(index='my_index', doc_type='my_index',
... body={'query': {'match': {'text': 'second'}}})
{
    'took': 1,
    'timed_out': False,
    '_shards': {'total': 5, 'successful': 5, 'skipped': 0, 'failed': 0},
    'hits': {
        'total': 1,
        'max_score': 0.25316024,
        'hits': [
            {
                '_index': 'my_index',
                '_type': 'my_index',
                '_id': '2',
                '_score': 0.25316024,
                '_source': {'text': 'a second test'}
            }
        ]
    }
}
```

仍然得到相当低的分数，因为我的搜索与本文中的文本不匹配，但由于这两个文档中只有一个包含“second”这个词，所以其他文档根本不显示。

Elasticsearch查询对象有更多的选项，都有详细记录，并且包括分页和排序等选项，就像关系数据库一样。

随意为此索引添加更多条目并尝试不同的搜索。完成试验后，可以使用以下命令删除索引。

````
>>> es.indices.delete('my_index')
````

### 四，Elasticsearch配置

将Elasticsearch集成到应用程序中是Flask功能的一个很好的例子。这是一个服务和Python包，与Flask没有任何关系，但是，我将从配置开始获得相当不错的集成级别，我将在app.config字典中编写该配置来自Flask：

```
class Config(object):
    # ...
    ELASTICSEARCH_URL = os.environ.get('ELASTICSEARCH_URL')
```

与许多其他配置条目一样，Elasticsearch的连接URL将来自环境变量。如果该变量未定义，我将设置为 `None ` ，然后将其用作禁用Elasticsearch的信号。这主要是为了方便起见，所以当您在应用程序上工作时，尤其是在运行单元测试时，您不必强制Elasticsearch服务启动并运行。因此，为了确保使用该服务，我需要直接在终端中定义 `ELASTICSEARCH_URL` 环境变量，或者将其添加到.env文件中，如下所示：

```
ELASTICSEARCH_URL=http://localhost:9200
```

但是Flask没有提供关于Elasticsearch的扩展。我不能像在上面的例子中那样在全局范围内创建Elasticsearch实例，因为要初始化它，我需要访问 `app.config `，它只在调用 `create_app（）` 函数后变得可用。所以我决定在应用程序工厂函数中为应用程序实例添加一个 `elasticsearch `属性：

``` python
from elasticsearch import Elasticsearch

# ...

def create_app(config_class=Config):
    app = Flask(__name__)
    app.config.from_object(config_class)

    # ...
    app.elasticsearch = Elasticsearch([app.config['ELASTICSEARCH_URL']]) \
        if app.config['ELASTICSEARCH_URL'] else None

    # ...
```

为应用程序实例添加一个新属性可能看起来有些奇怪，但是Python对象的结构并不严格，可以随时添加新属性。你也可以考虑的另一种方法是创建一个 `Flask` 的子类（也许称为 `Microblog` ），并在其 `__init __（）` 函数中定义 `elasticsearch` 属性。

请注意，如果未在环境中定义Elasticsearch服务的URL，我如何使用条件表达式使用Elasticsearch实例。

### 五，可扩展的全文搜索

正如我在本章的介绍中所说的，我希望能够轻松地从Elasticsearch切换到其他搜索引擎，并且我也不希望将此功能专门用于搜索博客文章，我更愿意设计一个未来的解决方案如果我需要，我可以轻松扩展到其他模型。出于所有这些原因，我决定为搜索功能创建一个抽象对象。所以为了让其通用可扩展，所以我不会假设Post模型是唯一需要被索引的模型，我也不会假设Elasticsearch是选择的索引引擎。但是如果我不能对任何事情做出任何假设，我怎么才能完成这项工作？

我需要的第一件事是以某种方式找到一种通用的方式来指示哪个模型以及其中的哪个或哪些字段将被索引。我要说任何需要索引的模型都需要定义`__searchable__` 类属性，该属性列出了需要包含在索引中的字段。对于Post模型，这些变化是：

```python
class Post(db.Model):
    __searchable__ = ['body']
    # ...
```

所以我在这里说的是这个模型需要有它的 `body` 字段索引。但为了确保这一点非常清楚，我添加的这个 `__searchable__` 属性只是一个变量，它没有任何与它相关的行为。它只会帮助我以通用的方式编写索引函数。

我将编写与app / search.py模块中的Elasticsearch索引交互的所有代码。这么做是为了将所有Elasticsearch代码保留在这个模块中。应用程序的其余部分将使用这个新模块中的函数来访问索引，并且不会直接访问Elasticsearch。这很重要，因为如果有一天我决定不再使用Elasticsearch并想切换到不同的引擎，我所需要做的就是重写这个模块中的函数，并且应用程序将继续像以前一样工作。

对于这个应用程序，我决定需要三个与文本索引相关的函数：条目添加到全文索引中，从索引中删除条目（假设有一天删除博客文章），并且执行搜索查询。下面是app / search.py模块，使用方法在Python shell 中展示实现Elasticsearch的这三个函数：

```python
from flask import current_app

def add_to_index(index, model):
    if not current_app.elasticsearch:
        return
    payload = {}
    for field in model.__searchable__:
        payload[field] = getattr(model, field)
    current_app.elasticsearch.index(index=index, doc_type=index, id=model.id,
                                    body=payload)

def remove_from_index(index, model):
    if not current_app.elasticsearch:
        return
    current_app.elasticsearch.delete(index=index, doc_type=index, id=model.id)

def query_index(index, query, page, per_page):
    if not current_app.elasticsearch:
        return [], 0
    search = current_app.elasticsearch.search(
        index=index, doc_type=index,
        body={'query': {'multi_match': {'query': query, 'fields': ['*']}},
              'from': (page - 1) * per_page, 'size': per_page})
    ids = [int(hit['_id']) for hit in search['hits']['hits']]
    return ids, search['hits']['total']
```

这些函数都是通过检查app.elasticsearch是否为 `None` 来开始的，如果为`None` 返回而不做任何事情。这样当Elasticsearch服务器未配置时，应用程序将继续运行而没有搜索功能，也不会出现任何错误。这只是在开发或运行单元测试期间的便利问题。

这些函数接受索引名称作为参数，在传递给Elasticsearch的所有调用中，我将这个名称用作索引名称，并且还将其用作文档类型，就像我在Python sehell 示例中所做的那样。添加和删除索引项的函数将SQLAlchemy模型类作为第二个参数。 `add_to_index()` 函数使用我添加到模型中的 `__searchable__` 变量来构建插入到索引中的文档。Elasticsearch文档还需要一个唯一的标识符，为此，刚好是使用SQLAlchemy模型的 `id` 字段也是唯一的。使用SQLAlchemy和Elasticsearch的相同id值在运行搜索时非常有用，因为它允许我链接两个数据库中的数据项。之前没有提到的是，如果尝试添加一个带有现有 `id` 的数据项，Elasticsearch将新的数据项替换旧的数据项， `add_to_index（）` 可以用于增加新对象以及修改对象。

没有向你展示在 `remove_from_index（）` 中使用的 `es.delete（）` 函数。该功能删除存储在给定 `id`  下的文档。下面是使用相同 `id` 来链接两个数据库中条目的便利性的一个很好的例子.

`query_index()` 函数使用索引名和文本搜索，以及分页控制，这样搜索结果就可以像Flask-SQLAlchemy结果一样被分页。已经在Python shell中看到了`es.search（) ` 函数的示例用法，和这里的调用是相似的，不同的是没有使用 `match` 查询类型，而是使用 `multi_math` 查询，因为它可以跨多个字段搜索。使用 `*` 来传递字段名称，告诉Elasticsearch查询所有字段，基本上搜索所有索引。这对于使该函数具有复用性很有用，因为不同的模型在索引中可以具有不同的字段名称。

除了查询本身之外， `es.search（）` 的 `body` 参数还包括分页参数。 `from` 和 `size` 参数控制需要返回整个结果集的哪个子集。Elasticsearch没有像Flask-SQLAlchemy那样提供一个很好的分页对象，所以要得到分页，就要计算 `from` 值。

 `query_index（）` 函数中的 `return` 语句有点复杂。它返回两个值：第一个是搜索结果的 `id` 元素列表，第二个是结果总数。两者都从 `es.search（）` 函数返回的Python字典中获取。如果您不熟悉用于获取ID列表的表达式，这称为列表推导式，并且是Python语言的一个特点，它允许您将列表从一种格式转换为另一种格式。在这种情况下，我使用列表推导式从Elasticsearch提供的更大的结果列表中提取id值。

这太混乱了吗？也许从Python控制台演示这些功能可以帮助你更多地理解它们。在接下来的会话中，手动将数据库中的所有帖子添加到Elasticsearch索引。在测试数据库中，有几个帖子中包含数字“1”，“2”，“3”，“4”和“5”，因此将其用作搜索查询。您可能需要调整您的查询以匹配数据库的内容：

``` python
>>> from app.search import add_to_index, remove_from_index, query_index
>>> for post in Post.query.all():
...     add_to_index('posts', post)
>>> query_index('posts', 'one two three four five', 1, 100)
([15, 13, 12, 4, 11, 8, 14], 7)
>>> query_index('posts', 'one two three four five', 1, 3)
([15, 13, 12], 7)
>>> query_index('posts', 'one two three four five', 2, 3)
([4, 11, 8], 7)
>>> query_index('posts', 'one two three four five', 3, 3)
([14], 7)
```

查询返回了七个结果。当我询问第1页时，每页有100项，我得到了全部七项，但接下来的三个例子显示了如何为Flask-SQLAlchemy所做的非常相似的方式对结果进行分页，除了结果将作为ID列表而不是SQLAlchemy对象。

最后实验完成：

```python
>>> app.elasticsearch.indices.delete('posts')
```

