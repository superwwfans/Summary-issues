#### 将搜索和SQLAlchemy结合

​	上篇文章讲了Python中如何使用Elasticsearch实现搜索功能，但是还有两个问题要解决：

​	1， 搜索结果为整数类型ID组成的列表，如何在模型类中找出对应的字段，来替换者个列表，然后通过者个结果查找出数据渲染到网页模板中；

​	2， 当项目中的模型类发生变化时，比如增加，修改，删除，Elasticsearch并不会检测到他的改变，造成两个数据库的不同不；

​	那么是不是在改变模型类中的数据同时触发Elasticsearch的方法改变索引，使得两者数据同步。

​	对于自动触发索引更改的问题，我决定从SQLAlchemy事件驱动更新到Elasticsearch索引，QLAlchemy提供了可以通知应用程序的大量事件列表。例如，每次提交会话时，都可以在由SQLAlchemy调用的应用程序中具有函数，并且在该函数中，我可以将与SQLAlchemy会话相同的更新应用于Elasticsearch索引。

​	为了实现这两个问题的解决方案，我将写一个mixin类。记得mixin类吗？在Flask-Login的UserMixin类添加到用户模型中，给它提供了Flask-Login所需的一些功能。对于搜索支持，定义自己的SearchableMixin类，当它连接到模型时，将使其能够自动管理与SQLAlchemy模型关联的全文索引。mixin类将SQLAlchemy和Elasticsearch两者相结合。

``` python
from app.search import add_to_index, remove_from_index, query_index

class SearchableMixin(object):
    @classmethod
    def search(cls, expression, page, per_page):
        """搜索"""
        ids, total = query_index(cls.__tablename__, expression, page, per_page)
        if total == 0:
            return cls.query.filter_by(id=0), 0
        when = []
        for i in range(len(ids)):
            when.append((ids[i], i))
        return cls.query.filter(cls.id.in_(ids)).order_by(
            db.case(when, value=cls.id)), total

    @classmethod
    def before_commit(cls, session):
        """ sqlalchemy提交数据之前检测变化 """
        session._changes = {
            'add': [obj for obj in session.new if isinstance(obj, cls)],
            'update': [obj for obj in session.dirty if isinstance(obj, cls)],
            'delete': [obj for obj in session.deleted if isinstance(obj, cls)]
        }

    @classmethod
    def after_commit(cls, session):
        """sqlalchemy提交数据后检测变化"""
        for obj in session._changes['add']:
            add_to_index(cls.__tablename__, obj)
        for obj in session._changes['update']:
            add_to_index(cls.__tablename__, obj)
        for obj in session._changes['delete']:
            remove_from_index(cls.__tablename__, obj)
        session._changes = None

    @classmethod
    def reindex(cls):
        """更新"""
        for obj in cls.query:
            add_to_index(cls.__tablename__, obj)
```

​	before_commit（）和after_commit（）方法将分别响应来自SQLAlchemy的两个事件，这两个事件分别在提交发生之前和之后触发。之前的处理程序很有用，因为会话尚未提交，所以我可以查看它并找出将要添加，修改和删除的对象，分别为session.new，session.dirty和session.deleted 。这些对象在会话提交后不再可用，所以我需要在提交之前保存它们。我正在使用session._changes字典将这些对象写入会话提交后仍然存在的位置，因为一旦会话提交，我将使用它们来更新Elasticsearch索引。

​	当调用after_commit（）处理程序时，会话已成功提交，因此这是在Elasticsearch端进行更改的适当时间。会话对象具有我在before_commit（）中添加的_changes变量，因此现在我可以迭代添加，修改和删除的对象，并对search.py中的索引函数进行相应的调用。

​	reindex（）类方法是一个简单的辅助方法，您可以使用它来刷新索引并使用关系端的所有数据。您看到我在上面的Python shell会话中执行类似的操作，将所有帖子初始加载到测试索引中。有了这个方法，Post.reindex（）将数据库中的所有帖子添加到搜索索引中。

在模型类中使用自定义的SearchableMixin类

``` python
class Post(SearchableMixin, db.Model):
    # ...

db.event.listen(db.session, 'before_commit', Post.before_commit)
db.event.listen(db.session, 'after_commit', Post.after_commit)
```

剩下的就是做一个搜索视图函数，获取用户提交搜索内容数据,渲染模板

``` python
@app.route('/search')
def search():
    if not g.search_form.validate():
        return redirect(url_for('main.explore'))
    page = request.args.get('page', 1, type=int)
    posts, total = Post.search(g.search_form.q.data, page,
                               current_app.config['POSTS_PER_PAGE'])
    next_url = url_for('main.search', q=g.search_form.q.data, page=page + 1) \
        if total > page * current_app.config['POSTS_PER_PAGE'] else None
    prev_url = url_for('main.search', q=g.search_form.q.data, page=page - 1) \
        if page > 1 else None
    return render_template('search.html', title=_('Search'), posts=posts,
                           next_url=next_url, prev_url=prev_url)
```

