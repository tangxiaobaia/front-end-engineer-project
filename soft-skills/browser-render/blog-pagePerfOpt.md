# [页面性能优化](https://blog.csdn.net/riddle1981/article/details/78681191)

1. 减少资源请求次数和压缩数据内容
   * 资源打包；
   * 雪碧图；
2. 高效合理的css选择符，css是逆向解析的
   * 避免使用 '* {}'；
   * 使用class替代标签选择；
   * 不使用标签限定id或者类选择符，如ul#nav或者ul.nav应该去掉ul；
   * 尽量少的使用后代选择器；
   * 考虑继承属性，可以避免重新创建规则节点；
3. js层面优化
   * 解决渲染阻塞，在不涉及到页面DOM操作的script标签中使用'async'，'defer'特性；
   * 减少对DOM的操作，操作DOM会导致大量的页面repaint和reflow，合理使用js变量存储内容，循环结束后再一次写入DOM；
   * 减少对DOM元素的查询和修改，查询时可将其赋值给局部变量；
   * 使用JSON格式进行数据交互；
   * 让经常需要修改的节点脱离文档流；
4. 使用cdn加速
5. 精简css和js文件