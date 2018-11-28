name: inverse
layout: true
class: center, middle, inverse

---
template: inverse

# Mongoengine

The ...

---
layout: false

## Overview

from mongoengine import connect
from pymongo import ReadPreference

connect(
    alias=conf.social_mongodb_alias,
    host=conf.social_mongodb_host,
    replicaSet=conf.social_mongodb_replicaset,
    connect=False,
    read_preference=ReadPreference.SECONDARY_PREFERRED
)


class Bar(Document):
    meta = {
        'db_alias': 'demo',
        'collection': 'foo',
        'strict': False,
    }

    name = fields.StringField()
    data = fields.DictField()


>> db.foo.insert({'name': 'kimqi', 'data': {'pos': 10}})                            <-- 插入一条数据


Bar.objects                                                                         <-- 一般操作 & 一些类型
    .update_one()
    .delete()
    .first()
    .insert()

one_bar = Bar.objects.first() -> Bar.objects[0] -> instance of Bar

bars = Bar.objects.filter() -> mongoengine.queryset.queryset.QuerySet -> (instance of Bar) list

>> type(Bar.objects) -> mongoengine.queryset.queryset.QuerySet


[mongoengine/base/metaclasses.py]
class TopLevelDocumentMetaclass(DocumentMetaclass):

        ...

        attrs['_meta'].update(attrs.get('meta', {}))                                <-- strict: False 等一些东西.. 有一些可设置的..

        if 'objects' not in dir(new_class):
            new_class.objects = QuerySetManager()                                   <-- Bar.objects 这个 QuerySetManager

        for attr_name, attr_value in attrs.iteritems():                             <-- 所有 field 继承自 BaseField, 遍历 Bar 中定义的所有 Field
            if not isinstance(attr_value, BaseField):
                continue
            attr_value.name = attr_name
            if not attr_value.db_field:
                attr_value.db_field = attr_name
            doc_fields[attr_name] = attr_value

        # Set _fields and db_field maps
        attrs['_fields'] = doc_fields                                               <-- 这个东西后面很多地方用, 取数据套结构
        attrs['_db_field_map'] = {k: getattr(v, 'db_field', k)
                                  for k, v in doc_fields.items()}


[mongoengine/queryset/base.py]
class BaseQuerySet(object):
    def __init__(self, document, collection):
        self._document = document
        self._collection_obj = collection
        ...

    def __call__(self, q_obj=None, class_check=True, read_preference=None,
                 **query):
        query = Q(**query)
        ...
        return queryset

    def __getitem__(self, key):
        """
        >>> User.objects[0]
        <User: User object>
        """
        ...

        return queryset._document._from_son(                                        <--  _document 就是 Document 下面贴了 _from_son.
            queryset._cursor[key],                                                  <--  pymongo 返回的 dict
            _auto_dereference=self._auto_dereference,
            only_fields=self.only_fields
        )

    def filter(self, *q_objs, **query):
        """An alias of :meth:`~mongoengine.queryset.QuerySet.__call__`
        """
        return self.__call__(*q_objs, **query)

    def create(self, **kwargs):
        return self._document(**kwargs).save(force_insert=True)

    def first(self):
        queryset = self.clone()
        try:
            result = queryset[0]
        except IndexError:
            result = None
        return result

    def insert(self, doc_or_docs, load_bulk=True,
               write_concern=None, signal_kwargs=None):
        ...

    def count(self, with_limit_and_skip=False):
        ...

    def delete(self, write_concern=None, _from_doc_delete=False,
            cascade_refs=None):
        ...


[mongoengine/base/document.py]
class BaseDocument(object):

    def __init__(self, *args, **values):

        # Set passed values after initialisation
        if self._dynamic:
            dynamic_data = {}
            for key, value in values.iteritems():
                if key in self._fields or key == '_id':
                    setattr(self, key, value)                                       <-- _dynamic 时直接 setattr
                elif self._dynamic:
                    dynamic_data[key] = value
        else:
            for key, value in values.iteritems():
                key = self._reverse_db_field_map.get(key, key)
                if key in self._fields or key in ('id', 'pk', '_cls'):              <-- metaclass 中 _fields 即 Bar Document 中定义的 Field
                    if __auto_convert and value is not None:
                        field = self._fields.get(key)
                        if field and not isinstance(field, FileField):
                            value = field.to_python(value)                          <-- field.to_python(value)  !!!!!!!!!!!  每个 field 都有自己的 to_python 方法..
                    setattr(self, key, value)
                else:
                    self._data[key] = value


    @classmethod
    def _from_son(cls, son, _auto_dereference=True, only_fields=None, created=False):

        if son and not isinstance(son, dict):                                       <-- 读上来的数据就是 dict
            raise ValueError("The source SON object needs to be of type 'dict'")

        data = {}
        for key, value in son.iteritems():
            key = str(key)
            key = cls._db_field_map.get(key, key)                                   <-- metaclass 中 _db_field_map
            data[key] = value

        obj = cls(__auto_convert=False, _created=created, __only_fields=only_fields, **data)

        return obj
