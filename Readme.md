# HTTP-304状态码

刚刚开始使用Fiddler的用户经常会对Fiddler的网络会话(Web Sessions)列表中的HTTP/304响应感到困惑:
![](http://pic002.cnblogs.com/images/2012/116671/2012111610163842.jpg)

Screenshot of Fiddler Web Sessions list showing 304s

如果客户端发送的是一个条件验证(Conditional Validation)请求,则web服务器可能会返回HTTP/304响应,这就表明了客户端中所请求资源的缓存仍然是有效的,也就是说该资源从上次缓存到现在并没有被修改过.条件请求可以在确保客户端的资源是最新的同时避免因每次都请求完整资源给服务器带来的性能问题.

辨别条件请求

当客户端缓存了目标资源但不确定该缓存资源是否是最新版本的时候,就会发送一个条件请求.在Fiddler中,你可以在Headers Inspector查找相关请求头,这样就可以辨别出一个请求是否是条件请求.

在进行条件请求时,客户端会提供给服务器一个If-Modified-Since请求头,其值为服务器上次返回的Last-Modified响应头中的日期值,还会提供一个If-None-Match请求头,值为服务器上次返回的ETag响应头的值:

Fiddler Request Headers Inspector screenshot

![](http://pic002.cnblogs.com/images/2012/116671/2012111610163856.jpg)
服务器会读取到这两个请求头中的值,判断出客户端缓存的资源是否是最新的,如果是的话,服务器就会返回HTTP/304 Not Modified响应,但没有响应体.客户端收到304响应后,就会从缓存中读取对应的资源.

另一种情况是,如果服务器认为客户端缓存的资源已经过期了,那么服务器就会返回HTTP/200 OK响应,响应体就是该资源当前最新的内容.客户端收到200响应后,就会用新的响应体覆盖掉旧的缓存资源.

只有在客户端缓存了对应资源且该资源的响应头中包含了Last-Modified或ETag的情况下,才可能发送条件请求.如果这两个头都不存在,则必须无条件(unconditionally)请求该资源,服务器也就必须返回完整的资源数据.

为什么要使用条件请求

当用户访问一个网页时,条件请求可以加速网页的打开时间(因为可以省去传输整个响应体的时间),但仍然会有网络延迟,因为浏览器还是得为每个资源生成一条条件请求,并且等到服务器返回HTTP/304响应,才能读取缓存来显示网页.更理想的情况是,服务器在响应上指定Cache-Control或Expires指令,这样客户端就能知道该资源的可用时间为多长,也就能跳过条件请求的步骤,直接使用缓存中的资源了.可是,即使服务器提供了这些信息,在下列情况下仍然需要使用条件请求:

在超过服务器指定的过期时间之后
如果用户执行了刷新操作的话
在上节给出的图片中,请求头中包含了一个Pragma: no-cache.这是由于用户使用F5刷新了网页.如果用户按下了CTRL-F5 (有时称之为“强刷-hard refresh”),你会发现浏览器省略了If-Modified-Since和If-None-Match请求头,也就是无条件的请求页面中的每个资源.

避免条件请求

通常来说,缓存是个好东西.如果你想提高自己网站的访问速度,缓存是必须要考虑的.可是在调试的时候,有时候需要阻止缓存,这样才能确保你所访问到的资源是最新的.

你也许会有个疑问:“如果不改变网站内容,我怎么才能让Fiddler不返回304而返回一个包含响应体的HTTP/200响应呢?”

你可以在Fiddler中的网络会话(Web Sessions)列表中选择一条响应为HTTP/304的会话,然后按下U键.Fiddler将会无条件重发(Unconditionally reissue)这个请求.然后使用命compare命令对比一下两个请求有什么不同,对比结果如下,从中可以得知,Fiddler是通过省略条件请求头来实现无缓存请求的:

Screenshot of Windiff of conditional and unconditional requests

![](http://pic002.cnblogs.com/images/2012/116671/2012111610163910.jpg)

如果你想全局阻止HTTP/304响应,可以这么做:首先清除浏览器的缓存,可以使用Fiddler工具栏上的Clear Cache按钮(仅能清除Internet Explorer缓存),或者在浏览器上按CTRL+SHIFT+DELETE(所有浏览器都支持).在清除浏览器的缓存之后,回到Fiddler中,在菜单中选择Rules > Performance > Disable Caching选项,然后Fiddler就会:删除所有请求中的条件请求相同的请求头以及所有响应中的缓存时间相关的响应头.此外,还会在每个请求中添加Pragma: no-cache请求头,在每个响应中添加Cache-Control: no-cache响应头,阻止浏览器缓存这些资源.
