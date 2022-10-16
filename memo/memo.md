SQLite3のselect結果をdataclass型+__getitem__で返す薄いラッパーを書く【Python】

　前回は[namedtuple][]で返した所を[dataclass][]で返す。

<!-- more -->

# ブツ

* [リポジトリ][]

[リポジトリ]:https://github.com/ytyaru/Python.sqlite3.row_factory.dataclass.20221016164153
[DEMO]:https://ytyaru.github.io/Python.sqlite3.row_factory.dataclass.20221016164153/

[sqlite3]:https://docs.python.org/ja/3/library/sqlite3.html
[row_factory]:https://docs.python.org/ja/3/library/sqlite3.html#sqlite3.Connection.row_factory
[sqlite3.Row]:https://docs.python.org/ja/3/library/sqlite3.html#sqlite3.Row
[__getitem__]:https://docs.python.org/ja/3/reference/datamodel.html#object.__getitem__
[cursor.description]:https://docs.python.org/ja/3/library/sqlite3.html#sqlite3.Cursor.description
[namedtuple]:https://docs.python.org/ja/3/library/collections.html#collections.namedtuple
[dataclass]:https://docs.python.org/ja/3/library/dataclasses.html
[dataclasses.make_dataclass]:https://docs.python.org/ja/3/library/dataclasses.html#dataclasses.make_dataclass
[mypy]:https://github.com/python/mypy

## 実行

```sh
NAME='Python.sqlite3.row_factory.dataclass.20221016164153'
git clone https://github.com/ytyaru/$NAME
cd $NAME/src
./run.py
./test.py
```

## コード例

```python
```

## ポイント

　列の値は以下のように参照できる。

```python
row.id
row[0]
row['id']
```

　本来[dataclass][]には[__getitem__][]が実装されていないため`[0]`や`['列名']`は使えない。このラッパーで[dataclass][]に[__getitem__][]を実装したので上記3通りの参照ができる。

# 実装

## 前回との差異

　[row_factory][]コールバック関数で[dataclass][]を返すようにした。[dataclasses.make_dataclass][]で[dataclass][]が作成できる。型アノテーションは未設定のためデフォルト値`typing.Any`型。

```python
def _dataclass_factory(self, cursor, row):
    return self._make_getitem(dataclasses.make_dataclass('Row', list(tuple(map(lambda d: d[0], cursor.description)))))(*row)
def _make_getitem(self, typ): #https://stackoverflow.com/questions/45326573/slicing-a-namedtuple
    def getitem(self, key):
        if isinstance(key, str): return getattr(self, key)
        elif isinstance(key, int): return getattr(self, row.__annotations__.keys()[key])
        else: raise TypeError('The key should be int or str type.')
    typ.__getitem__ = getitem
    return typ
```

　[__getitem__][]については実装されていないためここで実装する。`['列名']`のようにキーが`str`だったり、`[0]`のように`int`型だったりしたらそれに合った値を返す。

