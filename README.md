#SinaStorage SDK

##概述
新浪云存储Python平台SDK为第三方应用提供了简单易用的API调用服务，使第三方客户端无需了解复杂的验证机制即可进行授权、上传、下载等文件操作。
>本文档详细内容请查阅：[SinaStorage’s documentation](http://sinastor.appsina.com)

##SDK 环境要求

要求**Python 2.5+** ，不支持Python 3。
完全由Python标准库开发。

##快速上手

###1.创建bucket访问对象：
```python
from sinastorage.bucket import SCSBucket
import sinastorage

#设置access_key,secret_key
sinastorage.setDefaultAppInfo('你的accesskey', '你的secretkey')
s = SCSBucket('bucket的名称')
print s  
#<SCSBucket ... at 'http://sinastorage.com/...'>
```
获取[access_key,secret_key](http://sinastor.appsina.com/?c=console)

###2.bucket 操作:
* 创建bucket
```python
s = SCSBucket('需要创建的bucket名称')
s.put_bucket()
```
* 删除bucket:
```python
s = SCSBucket('需要删除的bucket名称')
s.delete_bucket()
```
* 列出所有bucket:
```python
s = SCSBucket()							#此处不要填写任何bucket名称
buckets_generator = s.list_buckets()	#返回generator，直接迭代即可
for bucket in buckets_generator:
    print bucket

#(bucketName,creationDate)				#tuple类型
```

###3.object 操作:
* 上传文件/内容:
```python
#文件内容
s = SCSBucket('bucket的名称')
s.put('文件上传路径',u'文件内容')

#文件
s = SCSBucket('bucket的名称')
f = open("本地文件路径",'rb')
s.put("文件上传路径",f)
f.close()
```
* 秒传文件:
```python
s = SCSBucket('bucket的名称')
s.put_relax('文件上传路径', '被秒传文件的sina_sha1值', 被秒传文件的文件长度s_sina_length)
```
* 复制文件:
```python
s = SCSBucket('bucket的名称')
#注意：source    必须从bucket开始，如：'/bucketname/file.txt'
s.copy(source='/源文件bucket名称/源文件uri地址', key='文件上传路径')
```
* 列文件目录:
```python
s = SCSBucket('bucket的名称')
#返回generator对象
files_generator = s.listdir(prefix='10000', marker='10000/1007.txt', limit=10)

#相关信息，通过generator属性获得
print ('truncated : %r\n'
'marker:%r\n'
'prefix:%r\n'
'delimiter:%r\n'
'contents_quantity:%r\n'
'common_prefixes_quantity:%r\n'
'next_marker:%r'%(files_generator.truncated, 
                  files_generator.marker,
                  files_generator.prefix,
                  files_generator.delimiter,
                  files_generator.contents_quantity,
                  files_generator.common_prefixes_quantity,
                  files_generator.next_marker))
#列目录
for item in files_generator:
    print item

#(name, isPrefix, sha1, expiration_time, modify, owner, md5, content_type, size)
```
* 下载文件:
```python
s = SCSBucket('bucket的名称')
f = s['需要下载的文件路径']

#获取文件相关信息
print f.scs_info["mimetype"]
#'application/octet-stream'

print f.scs_info.keys()
#['mimetype', 'modify', 'headers', 'date', 'size', 'metadata']

#下载文件至本地
CHUNK = 16 * 1024
with open('本地目标文件地址', 'wb') as fp:
    while True:
        chunk = f.read(CHUNK)
        if not chunk: break
        fp.write(chunk)
```
* 删除文件:
```python
s = SCSBucket('bucket的名称')
del s["需要删除的文件路径"]
```
* 获取文件信息:
```python
s = SCSBucket('bucket的名称')
info = s.info('服务器端文件路径')
print info['mimetype']
#application/pdf
print info['size']
#2433230
print info
#{'mimetype': 'application/pdf', 'modify': datetime.datetime(2014, 4, 1, 6, 58, 58), 'headers': {'content-length': '2433230', ...}, 'date': datetime.datetime(2014, 4, 1, 9, 14, 57), 'metadata': {'crc32': 'DDEF42FA', ...}, 'size': 2433230}
```
* 修改文件meta信息:
```python
s = SCSBucket('bucket的名称')
s.update_meta('服务器端文件路径', {'自定义属性名':'自定义属性值','file_meta_key':'meta_value'})
```
* 获取文件acl信息:
```python
s = SCSBucket('bucket的名称')
print s.acl_info('服务器端文件路径')
#{u'Owner': u'SINA000000...', u'ACL': {u'GRPS000000ANONYMOUSE': [u'read'], u'GRPS0000000CANONICAL': [u'read_acp', u'write_acp']}}
```
* 修改文件acl信息:
```python
from sinastorage.bucket import SCSBucket,ACL
s = SCSBucket('bucket的名称')
acl = {}
acl[ACL.ACL_GROUP_ANONYMOUSE] = [ACL.ACL_READ]
acl[ACL.ACL_GROUP_CANONICAL] = [ACL.ACL_READ_ACP,ACL.ACL_WRITE_ACP]

s.update_acl('服务器端文件路径', acl)
```
###3.URL签名工具:
* 无签名信息URL:
```python
s = SCSBucket('bucket的名称')
print s.make_url('待生成url地址的文件路径')
```
* 含签名信息URL:
```python
s = SCSBucket('bucket的名称')
print s.make_url_authed('待生成url地址的文件路径')
```

For more detailed documentation, refer [here](http://sinastor.appsina.com)
