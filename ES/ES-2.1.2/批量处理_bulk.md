### 批量处理
1. 使用`_bulk API`实现
2. 批量处理的时候，**顺序执行**操作；如果中间有一条操作处理失败，仍会继续执行下面的指令操作，会在返回的时候说明失败的操作信息；
3. 

### 参考
https://www.elastic.co/guide/en/elasticsearch/reference/2.1/docs-bulk.html