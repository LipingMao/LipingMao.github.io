---
title: DevOps -- pg password
---

> PG密码里面$$ 会有问题（使用一个$没有问题）

PG密码配置：iku7ohE$$TB3mfMO
产生了如下错误日志：
```
Traceback (most recent call last):
  File "/usr/lib/python2.7/site-packages/patroni/postgresql/bootstrap.py", line 335, in post_bootstrap
    self.create_or_update_role(superuser['username'], superuser['password'], ['SUPERUSER'])
  File "/usr/lib/python2.7/site-packages/patroni/postgresql/bootstrap.py", line 328, in create_or_update_role
    self._postgresql.query(sql, *params)
  File "/usr/lib/python2.7/site-packages/patroni/postgresql/__init__.py", line 254, in query
    return self.retry(self._query, sql, *args)
  File "/usr/lib/python2.7/site-packages/patroni/utils.py", line 313, in __call__
    return func(*args, **kwargs)
  File "/usr/lib/python2.7/site-packages/patroni/postgresql/__init__.py", line 245, in _query
    raise e
SyntaxError: syntax error at or near "TB3mfMO"
LINE 6: ..."postgres" WITH SUPERUSER LOGIN PASSWORD 
```
