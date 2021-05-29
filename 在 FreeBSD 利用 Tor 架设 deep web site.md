在FreeBSD安装Tor可以直接透过ports安装，难度为零，就不多说。

安装完成后，设定档torrc会出现在/usr/local/etc/tor这目录下。

接着就找写着「############### This section is just for location-hidden services ###」的区段来新增一些内容：
『

1|############### This section is just for location-hidden services ###

2|

3|## Once you have configured a hidden service, you can look at the 

4|## contents of the file ".../hidden_service/hostname" for the address

5|## to tell people.

6|##

7|## HiddenServicePort x y:z says to redirect requests on port x to the

8|## address y:z.  

9|

10|HiddenServiceDir /usr/local/etc/tor/sites/mysite1

11|HiddenServicePort 80 127.0.0.1:10000

12|

13|HiddenServiceDir /usr/local/etc/tor/sites/mysite2

14|HiddenServicePort 80 127.0.0.1:10001
                                                                     
』

在这段之前的设定只要照常识设定就行了，想挡什么的话就在设定档挡，设定档不方便挡的用防火墙挡。

HiddenServiceDir这个参数指定好，让Tor重读设定档后，在指定的目录下就会出现hostname和private_key这两个档案。

其中hostname就是你的主机名称，所以就得在http daemon的server name字段设定这个名称。

你会看到有些.onion的网址开头会是一串有意义的字母，那是经过长时间运算刻意碰出来的，如果你吃饱太闲也可以做一个来玩玩，但必须自行学习和留意一些安全原则。

HiddenServicePort 80 127.0.0.1:10000的意思，就是将从Tor network上流进本机port 80的流量转送到127.0.0.1:10000去。

换句话说，你的http daemon对应的virtual host必须去listen 127.0.0.1:10000，这样就可以正确接收到http request。

如果想让wordpress的blog可以同时支持多网域，在wp-config.php设定$table_prefix之后放进这两行就行了：

『

1|define('WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST']);

2|define('WP_HOME', 'http://' . $_SERVER['HTTP_HOST']);

』

当然网站上的图片网址和超链接也必须使用/开头，而不是整串从http://开始，否则别人开下去就是从clearnet上开了，这样他还必须经过exit nodes出来才开得到。

如果你吃饱太闲想写plugin或者去找现成的plugin做自动代换，那也没什么问题。

只是这网站没什么犯法的东西，加上我还挂着clearnet才有的disqus，所以我是没做得那么彻底。

要让一般人浏览.onion虚拟网域下的deep web，基本上是他必须也有装Tor相关软件才行，譬如安装Tor Browser。

但是要注意这玩意会跟Firefox打架，因为它本身就是Firefox。

另一种方法是走现成的Tor2Web服务，也可以连到这里来，算是一个折衷方法，缺点就是会出现他们的广告。

目前我这台装的security/tor-devel版本是有一些bug，架出来的deep web逛一逛可能就喷这信息然后停掉了：

「

Nov 15 01:40:20.000 [err] void tor_assertion_failed_(const char *, unsigned int, const char *, const char *)(): Bug: src/or/connection.c:3541: connection_handle_event_cb: Assertion connection_state_is_connecting(conn) failed; aborting.
Nov 15 01:40:20.000 [err] Bug: Assertion connection_state_is_connecting(conn) failed in connection_handle_event_cb at src/or/connection.c:3541. (Stack trace not available)

」


根据他们的Trac上的说明，在make config的时候关掉BUFFEREVENTS这个选项就行了：https://trac.torproject.org/projects/tor/ticket/4697

不过这是很老的版本了，他们当时也宣称已经修复，只是我在2.6.x还是会遇到。
