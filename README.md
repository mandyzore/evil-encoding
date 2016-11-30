# 中文编码处理
-------------------------------------------

编码虐我千百遍，我待中文如初恋。

-------------------------------------------

## boilerpipe

更改源python/site-packages/boilerpipe/extract/__init__.py文件中```__init__```方法：

```
def __init__(self, extractor='DefaultExtractor', **kwargs):
        if kwargs.get('url'):
            request     = urllib2.Request(kwargs['url'], headers=self.headers)
            connection  = urllib2.urlopen(request)
            self.data   = connection.read()
            encoding    = connection.headers['content-type'].lower().split('charset=')[-1]
            if encoding.lower() == 'text/html':
                encoding = charade.detect(self.data)['encoding']
                if encoding=="ISO-8859-2":
                    # print("! ISO-8859-2") ## 有的中文网页也是醉了
                    encoding = 'utf-8'
            if encoding is None:
                # print("!!! NONE")
                encoding = 'utf-8'
            try:
                self.data = str(self.data, encoding)
                # print("!!{}".format(encoding))
            except UnicodeError:
                # print("!!! {}".format(encoding))
                self.data = self.data.decode(encoding, 'ignore')

            # self.data = str(self.data, encoding)
        elif kwargs.get('html'):
            self.data = kwargs['html']
            if not isinstance(self.data, str):
                try:
                    self.data = str(self.data, charade.detect(self.data)['encoding'])
                    # print("!!{}".format(encoding))
                except UnicodeError:
                    encoding = charade.detect(self.data)['encoding']
                    # print("!!! {}".format(encoding)) 
                    self.data = self.data.decode(encoding, 'ignore') 
            else:
                raise Exception('No text or url provided')

        try:
            # make it thread-safe
            if threading.activeCount() > 1:
                if jpype.isThreadAttachedToJVM() == False:
                    jpype.attachThreadToJVM()
            lock.acquire()
            
            self.extractor = jpype.JClass(
                "de.l3s.boilerpipe.extractors."+extractor).INSTANCE
        finally:
            lock.release()
    
        reader = StringReader(self.data)
        self.source = BoilerpipeSAXInput(InputSource(reader)).getTextDocument()
        self.extractor.process(self.source)
```

## pdfMiner bug

/usr/local/lib/python2.7/dist-packages/pdfminer/utils.py

bug函数:

```
def enc(x, codec='ascii'):
    """Encodes a string for SGML/XML/HTML"""
    x = x.replace('&', '&amp;').replace('>', '&gt;').replace('<', '&lt;').replace('"', '&quot;')
    return x.encode(codec, 'xmlcharrefreplace')

```

修正：

```
def enc(x, codec='ascii'):
    """Encodes a string for SGML/XML/HTML"""
    x = x.replace('&', '&amp;').replace('>', '&gt;').replace('<', '&lt;').replace('"', '&quot;')
    try:
        return x.encode(codec, 'xmlcharrefreplace')
    except UnicodeDecodeError as e:
        return ''
```


