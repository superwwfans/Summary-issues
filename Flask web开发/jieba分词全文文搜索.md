```
#! usr/bin/env python
# coding=utf-8
# FileName = ''
# time:
# author: huang-xin-dong
# about: 
# ---------------------------------------------------------
from __future__ import unicode_literals

import os

from whoosh.index import create_in, open_dir
from whoosh.fields import *
from whoosh.qparser import QueryParser
from jieba.analyse import ChineseAnalyzer

from model.artilce_model import Article


sys.path.append("../")


def search_libs(content):
    """全文搜索
    :param content:
    :return:
    """
    analyzer = ChineseAnalyzer()

    schema = Schema(title=TEXT(stored=True), path=ID(stored=True), content=TEXT(stored=True, analyzer=analyzer))
    if not os.path.exists("tmp"):
        os.mkdir("tmp")

    ix = create_in("tmp", schema)

    results_list = list()

    for i in Article.query.all():
        writer = ix.writer()  # 写入文件
        writer.add_document(
            title=i.title,
            path='/{}'.format(i.id),
            content=i.content
        )
        writer.commit()
        searcher = ix.searcher()
        parser = QueryParser("content", schema=ix.schema)

        for keyword in analyzer(content.strip('')):  # jieba分词
            q = parser.parse(keyword.text)
            results = searcher.search(q)
            if results.__len__() != 0:
                results_list.append(i)
    results_list = set(results_list)
    return {"status": True, "msg": "查找成功", "data": results_list}
```