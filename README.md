# gretry

一个错误重试装饰器

# 安装

```
pip install gretry
```

# 参数介绍

- retry: 失败重试次数（默认 3 次）
- delay: 错误重试间隔（默认 0s）
- on_exceptions: 哪些报错才重试（默认 都重试）
- ignore_exceptions: 哪些报错不重试（默认 无）
- callback: 执行成功回调结果函数
- error_callback: 执行失败回调结果函数
- raise_exception: 一直失败，最后是否需要抛出错误（默认 True）

# 注意点

## 当有 callback 参数时

- 当有正确的结果时，会将结果作为参数放入 callback 并执行
- 无正确结果则直接抛出

## 当有 error_callback 参数时

- 当执行失败时，会在抛出前，将函数的执行参数，作为参数放入 error_callback 并执行
- 当执行正确时，会将正确的结果返回

## 错误的继承类的问题

比如 FileExistsError 类型是 OSError 的子类，这里判断的时候是 FileExistsError 就是 FileExistsError，不会向上识别

# 示例

## 示例 1

【demo】

```
from gretry import retry


def failed(name):
    print('failed', name)


def success(result):
    print('success', result)


@Retry(max_retry=3, delay=1, callback=success, error_callback=failed)
def run(name):
    raise FileExistsError('文件不存在！')


if __name__ == '__main__':
    run(name='郭一会儿')

```

【输出】

```
failed 郭一会儿
Traceback (most recent call last):
  File "D:\Program\Python\Pypi\gretry\test.py", line 18, in <module>
    run(name='郭一会儿')
  File "D:\Program\Python\Pypi\gretry\gretry\gretry.py", line 136, in wrapper
    return self.runner(func, *args, **kwargs)
  File "D:\Program\Python\Pypi\gretry\gretry\gretry.py", line 103, in runner
    raise exception
  File "D:\Program\Python\Pypi\gretry\gretry\gretry.py", line 74, in runner
    result = func(*args, **kwargs)
  File "D:\Program\Python\Pypi\gretry\test.py", line 14, in run
    raise FileExistsError('文件不存在！')
FileExistsError: 文件不存在！
```

## 示例 2

on_exceptions、ignore_exceptions 参数可以是错误的类型或者是列表

```
@retry(max_retry=3, on_exceptions=FileExistsError, ignore_exceptions=[OSError])
def run(name):
    raise FileExistsError('文件不存在！')
```