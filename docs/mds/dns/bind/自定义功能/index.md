## Bind Configuration Parser

> 主要功能是解析bind的配置文件，可以实现文件的增删改查，并可以根据抽象出来的内容结合数据库等内容进行管理，让管理端可以使用mysql，但是实际业务提供走BindDNS自己的内容和功能。

bind的配置文件和Nginx的是比较类似的，因此我们也可以看Nginx配置管理的一些实现脚本。

### 参考附录：

- [Ruby写的GlobalDNS，仅供参考](https://github.com/globocom/GloboDNS/blob/master/lib/zonefile.rb)
- [namedparser](https://github.com/hachibeeDI/namedparser/blob/master/namedparser/parser.py)
- [bind2other](https://github.com/hdais/bind2other/blob/master/bind2other.py)
- [bindparser](https://github.com/khattori/bindparser/blob/master/bindparser/parser.py)
- [bind9_parser](https://github.com/egberts/bind9_parser)

nginx配置相关的解析工具，可以用来参考

- [https://github.com/lytics/confl.git](https://github.com/lytics/confl.git)
- [https://github.com/recoye/config](https://github.com/recoye/config)
- [https://github.com/Sid-Sun/nginx-auto-config.git](https://github.com/Sid-Sun/nginx-auto-config.git)
- [https://github.com/bilalcaliskan/nginx-conf-generator.git](https://github.com/bilalcaliskan/nginx-conf-generator.git)

zone配置管理

- [https://github.com/jforman/binder](https://github.com/jforman/binder)
- [https://github.com/oznu/dns-zone-blacklist.git](https://github.com/oznu/dns-zone-blacklist.git)
- [https://github.com/bwesterb/go-zonefile](https://github.com/bwesterb/go-zonefile)
- [https://github.com/tjl2/dnsadmin](https://github.com/tjl2/dnsadmin)
- [https://github.com/cego/cfzone/blob/master/recordCollection.go](https://github.com/cego/cfzone/blob/master/recordCollection.go)
- [https://github.com/ant30/bind9-zone-editor](https://github.com/ant30/bind9-zone-editor)
- [https://github.com/shahwahed/BindZoneMaker](https://github.com/shahwahed/BindZoneMaker)
- [https://github.com/igor47/dnsparser/blob/master/dnsparser.py](https://github.com/igor47/dnsparser/blob/master/dnsparser.py)
- [https://github.com/GertBurger/zoneparser/blob/master/zoneparser/__init__.py](https://github.com/GertBurger/zoneparser/blob/master/zoneparser/__init__.py)
- [https://github.com/plinss/bindtool](https://github.com/plinss/bindtool)
- [https://github.com/joemiller/dns_compare](https://github.com/joemiller/dns_compare)
