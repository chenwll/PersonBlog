# Buffer

`Buffer` 对象用于表示固定长度的字节序列，和我们日常使用的数组不一样，数组长度是可以更改的。

utf-8是啥





# process

## process.argv

它的作用是获取执行进程后面的参数，返回是一个数组。数组第一项是node可执行文件的绝对路径，数组第二项是文件的绝对路径，其他项是参数





# path

## path.join()

可以用来拼接路径

```node
path.join('/foo','/cxk','/ikun')
// /foo/cxk/ikun
```

同时，可以支持 `.. ` `../` `./` 这些操作符

```node
path.join('/foo','/cxk','/ikun','../')
// /foo/cxk/
```



