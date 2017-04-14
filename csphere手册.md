模板函数

- [函数](#函数)
  + [userdata](#userdata)
  + [env](#env)
  + [label](#label)
  + [instance](#instance)
  + [service](#service)
  + [container](#container)
  + [searchContainers](#searchContainers)
  + [其他常用函数](#其他常用函数)
- [内置变量](#内置变量)
- [用例](#用例)
  + [haproxy负载均衡](#haproxy)
  + [Redis sharding cluster](#redis)
  + [Percona XtraDB Cluster](#pxc)



函数
------------

### userdata 
获取用户设定的模板变量值
```liquid
{{ userdata "Name" }}
{{ $name := userdata "Name" }}
```


### env 
获取容器的环境变量

获取当前容器的环境变量
```liquid
{{ env "MYENV" }}
```

获取其他容器的环境变量,首个参数使用容器名或ID均可
```liquid
{{ env "r1-db-1" "CSPHERE_NODE_CPU_NUMBER" }}
```

##### 可用的内置容器变量：
| 变量名 | 说明 | 值示例 | 
|------|---------|------|
| `CSPHERE_SERVICE_NAME` | 服务名 | app |
| `CSPHERE_INSTANCE_NAME` |实例名 | testr4 |
| `CSPHERE_INSTANCE_DOMAIN` |实例域名 | testr4.csphere.local | 
| `CSPHERE_TEMPLATE_NAME` | 模板名 | ccfg | 
| `CSPHERE_TEMPLATE_REVISOIN` | 模板版本 | 3764e7dd72c487d8ef9e7df276ae08fe68b29fb5 |
| `CSPHERE_TEMPLATE_TAGS` | 模板Tags | latest |
| `CSPHERE_NODE_NAME` | 节点主机名 | node1 | 
| `CSPHERE_NODE_IP` | 节点主机IP地址 | 192.168.159.101 |
| `CSPHERE_NODE_LABELS` | 节点主机标签 | operatingsystem=COS 723.3.0+2016-02-02-0238,csphere_svrpoolid=56a6e0dde0323a02a8000006,host=node1,storagedriver=overlay,executiondriver=native-0.2,kernelversion=4.0.5 |
| `CSPHERE_NODE_CPU_NUMBER` | 节点主机CPU核心个数 | 2 |
| `CSPHERE_NODE_MEMORY_SIZE` | 节点主机内存大小 | 805232640 |
| `CSPHERE_CONTAINER_NAME` | 容器名 | testr4-app-2 |
| `CSPHERE_CONTAINER_DOMAIN` | 容器域名 | 2.app.testr4.csphere.local | 
| `CSPHERE_CONTAINER_SEQ` | 容器在服务中的序号 | 2 |
| `CSPHERE_SERVICE_CREATE_CONTAINERNUM` | 服务在首次创建时的容器数量 | 3 |


### label
获取容器的标签

获取当前容器的标签
```liquid
{{ label "MyLabel" }}
```

获取指定容器的标签,首个参数使用容器名或ID均可
```liquid
{{ label "r1-app-1"  "MyLabel" }}
```

##### 可用的容器内置标签：
| 变量名 | 说明 | 值示例 |
|------|---------|-------|
| `csphere_templateid` | 模板ID | 56b1ce61e0323a2f460000eb|
| `csphere_containerseq` | 容器在服务中的序号  | 2 |
| `csphere_instanceid` | 实例ID  | 56b32e39e0323a288b000046|
| `csphere_instancename` | 实例名  | testr4 |
| `csphere_nodeid` | 节点主机ID  | 56a6e182e0323a02a8000026 |
| `csphere_servicename` | 服务名 | app |

### instance
获取实例数据

获取当前实例的数据
```liquid
{{ instance }}
{{ $ins := instance }}
```

获取指定实例的数据
```liquid
{{ instance "r1" }}
{{ $ins := instance "r1" }}
```

获取到的实例数据结构如下：
```liquid
{
    Name           string                      // 实例名
    Domain         string                      // 实例域名
    Services       []string                    // 实例的服务
    Containers     []Container                 // 实例的所有容器，Container结构体详情见下面的 container 函数
    DeployInfo     map[string][]Container      // 实例各个服务的容器，Container结构体详情见下面的 container 函数
}
```

### service
获取实例服务数据

获取当前实例，当前服务的数据
```liquid
{{ service }}
{{ $svr := service }}
```

获取当前实例，指定服务的数据
```liquid
{{ service  "db" }}
{{ $svr := service  "db" }}
```

获取指定实例，指定服务的数据
```liquid
{{ service  "r2" "lb" }}
{{ $svr := service  "r2" "lb" }}
```

获取到的服务数据结构如下：
```liquid
{
        Name            string             // 服务名
        Instance        string             // 所属的实例名
        Containers      []Container        // 服务中的容器，Container结构体详情见下面的 container 函数
        ContainerNum    uint               // 服务中的容器个数
}
```

### container
获取容器数据

获取当前容器的数据,使用容器名或ID均可
```liquid
{{ container }} 
{{ $c := container }} 
```

获取指定容器的数据,使用容器名或ID均可
```liquid
{{ container "abc" }}
{{ $c := container "abc" }}
```

获取到的容器数据结构如下：
```liquid
{
        ID            string               // 容器ID
        Name          string               // 容器名
        Domain        string               // 容器域名
        Seq           string               // 容器序号
        Image         string               // 镜像
        Pid           int                  // 容器PID， 首次创建的时候为0
        IPAddr        string               // 容器IP地址，首次创建的时候为空
        IpPrefixLen   int                  // 容器IP掩码，首次创建的时候为0
        Gateway       string               // 容器网关地址，首次创建的时候为空
        CpuSet        []string             // CPU核心数配额, CPU编号
        CpuSetNum     int                  // CPU核心数配额
        Memory        int64                // 内存配额,首次创建的时候为空
        DiskSize      int64                // 磁盘空间配额, 首次创建的时候为空
	    Node		  string               // 容器所在的主机ID 
}
```

**注意: 如下情况, 容器的`IPAddr` `IpPrefixLen` `Gateway` 字段获取不到**

> 首次部署, 引用本服务内容器的IP地址, 且容器不是静态指定IP  
> 首次部署, 引用启动级别比当前服务低的容器IP地址  
> 升级部署, 同属于变化升级的服务, 且引用启动级别比当前服务低的容器IP地址  

### searchContainers
根据条件搜索容器列表  
参数:
  - 主机ID
  - 实例名 
  - 服务名 (当指定了实例名时有效)
  - limit个数 (<0 不限制)

```liquid
{{ $cs := searchContainers "5770f1f4e0323a1ca5000019" "blog" "db" -1 }}
```

示例:
```liquid
{{ $node := label "csphere_nodeid" }}
{{ $ins := label "csphere_instancename" }}
{{ $svr := label "csphere_servicename" }}

local services containers: 
{{ $cs := searchContainers $node $ins $svr -1 }}git s
{{ toJSONPretty $cs }}

other services containers:
{{ $cs := searchContainers $node $ins "db" 10 }}
{{ toJSONPretty $cs }}
```

获取到的容器列表数据结构如下:
```liquid
[
  { 单个容器数据结构 },
  { 单个容器数据结构 },
  ...
]
```

### 其他常用函数

| 函数名 | 说明 | 示例 |
|---|---|---|
| **时间日期** |
| timestamp | 格式化为时间戳，如果没有传入参数则返回当前时间戳 | |
| **字符处理**  |
| toTitle | 单词首字母大写 | `{{toTitle .Name}}` |
| toLower | 转换为小写字母 | `{{toLower .Name}}` |
| toUpper | 转换为答谢字母 | `{{toUpper .Name}}` |
| join | 合并数组为字符串， 参数：字符串数组[]string, 连接符string，结果：string | `{{join .Members "-"}}` |
| split | 分割字符串为数组， 参数：字符串string，分隔符string，结果：[]string | `{{split .Members ","}}` |
| trimSpace | 去首尾空字符 | `{{trimSpace " He llo "}}`  == `He llo` |
| regexReplaceAll | 替换所有正则匹配项 | `{{regexReplaceAll "vsphere and msphere" "[a-z]{2}phere" "csphere"}}` == `csphere and csphere` |
| replaceAll | 替换字符串 | `{{regexReplaceAll "vsphere and msphere" "phere" ""}}` == `vs and ms` |
| regexMatch | 返回是否能匹配 | `{{regexMatch "csphere" "phere" }}`  == `true` |
| **类型转换** |
| toBool / parseBool | 格式化参数为Bool。 参数：string, 结果：bool， 参数转换为小写后==`true`的一律为true，否则为false | `{{toBool "..."}}` == false, `{{toBool "True"}}` == true |
| toFloat / parseFloat | 格式化参数为float。 参数：string，float，int, 结果：float |`{{toFloat "" }}` == 0, `{{toFloat "10" }}` == 10 |
| toInt / parseInt | 格式化参数为int， 参数：string,float,int, 结果：int |  `{{toInt "" }}` == 0, `{{toInt "99.999" }}` == 99 |
| toUint / parseUint | 格式化参数为uint， 参数：string,float,int,uint, 结果：uint |  `{{toUint "" }}` == 0, `{{toUint "99.999" }}` == 99 |
| **JSON** |
| parseJSON | 格式化参数为JSON对象。参数：string， 结果：json对象 | |
| toJSON | 格式化参数为JSON字符串。参数：任意类型，结果：string | |
| toJSONPretty | 同`toJSON`， 额外带美观格式化 | |
| **数学计算** |
| calc | 数学计算器， 参数：任意类型不定长参数，结果为float | 例1： `{{calc "(1+1+2) * 3 % 5"}}` == 2, 例2： `{{ calc .Em1 "+" .Em2 "*" .Em3 }}` == 60`（"Em1": float64(10), "Em2": string("10"),  "Em3": int(5)） |
| calcInt | 数学计算器，结果为int |  |
| min | 求数组中最小值， 参数： int数组或float数组，结果：int或float（根据传入的参数决定） | |
| max | 求数组中最大值，参数： int数组或float数组，结果：int或float（根据传入的参数决定）  | |
| sum | 求数组所有值的和， 参数： int数组或float数组，结果：int或float（根据传入的参数决定） | |
| avg | 求数组的算术平均数， 参数： int数组或float数组，结果：int或float（根据传入的参数决定）| |
| num | 求数组中元素个数， 参数： 任意类型数组，结果：int | |
| **保留只为保持兼容,不再推荐使用,可使用calc/calcInt替代** | 
| add | 算数加, 参数: int/uint/float, 结果: int,uint或float(根据传入的参数决定) | `{{ add 10 99 }}` == 109, `{{ add 10 11.11 }}` == 21.11 |
| subtract| 算数减, 参数int/uint/float, 结果: int,uint或float(根据传入的参数决定) | `{{ subtract 10 1 }}` == -9, `{{ subtract 10 1.1 }}` == "-8.9" |
| multiply | 算数乘, 参数int/uint/float, 结果: int,uint或float(根据传入的参数决定) | `{{ multiply 10 3 }}` == "30", `{{ multiply 10 2.5 }}` == "25" |
| divide | 算数除, 参数int/uint/float, 结果: int,uint或float(根据传入的参数决定) | `{{ divide 3 12 }}` == "4", {`{{ divide 4.0 10 }}` == "2.5" |
| mod | 算数取余, 参数int/uint, 结果: int,uint(根据传入的参数决定) | {`{{ mod 3 10 }}` == "1", {`{{ mod 3 10 }}` == "-1", |
| **其他** |
| in | 判断是否包含， 参数：列表，元素 （都是任意类型）。 结果：bool | 举例说明： `"abs" "s" == true`, `[]int{1,2,3}  1 == true`, `[]int{1,2,3} "1" == false`, `[]string{"f1","f2"}  "f1" == true`   |
| loop | 循环  | `{{ range (loop 5 8 )}}{{print .}}{{end}}` 输出 "567" ，`{{ range (loop 5 ) }}{{ print . }}{{end}}`输出"01234"|
| mkSlice | 构造数组，参数：不定长`相同类型`参数，结果： int数组/float数组/string数组/interface{}数组（根据传入的首个参数决定）  | `{{ $sli := mkSlice "a" "b" "c" 1 2 3 }}` |
| mkMap | 构建Map，参数：不定长参数，奇数为string，偶数类型不限，结果：map[string]interface{}  | `{{ $map := mkMap "a" 1 "b" 2 "c" 3 "d" 4 }` |



### 内置变量
------------
| 内置变量 | 说明 |
|----------|------|
| `.Instance` | 当前实例对象 {{ .Instance.Name }}  == {{ $ins := instance }} {{ $ins.Name }}  |
| `.Service` | 当前服务对象 {{ .Service.Name }}  == {{ $svr := service }} {{ $svr.Name }} |
| `.Container` | 当前容器对象 {{ .Container.Name }} == {{ $c := container }} {{ $c.Name }} |

注意：如果用户指定的变量和内置变量同名，将会被内置变量覆盖



用例
------------

### haproxy
haproxy负载均衡配置
```liquid
# define backend
backend backend_servers

	# balance with roundrobin
    balance            roundrobin

    # get user defined port
    {{ $port := userdata "BACKEND_PORT" }}

    # define backend servers
    {{ $backendSvr := env "BackendSvr" }}
    {{ $svr := service $backendSvr  }}
    {{ range $container := $svr.Containers }}
            server   {{ $container.Name }}   {{ $container.Domain }}:{{$port}}  check
    {{end}}
```
### redis
Redis sharding cluster配置，模板整体配置如下：
![redis sharding模板](https://github.com/qidunhu/docs/blob/master/images/redis2.jpg)
从节点模板

![从节点](https://github.com/qidunhu/docs/blob/master/images/redis3.jpg)



该模板中定义了
```REOLICAS_NUM```
```QUIET_MODE```
这两个环境变量，可以在配置文件中以```{{.REPLICAS_NUM}}```的方式去引用定义好的变量，下面为从节点的配置文件。
```
REPLICAS={{.REPLICAS_NUM}}
{{ $rs := service "redis" }}
NODES="{{range $i,$rc := $rs.Containers}} {{$rc.IPAddr}}:6379{{end}}"
```


  
  ### pxc 
  >(Percona XtraDB Cluster)
  
  PXC配置，模板整体如下
  
  ![pxc](https://github.com/qidunhu/docs/blob/master/images/1.png)
该模板定义了两个服务，一个pma服务和pxc和服务，下图为对pma服务的定义
![pam ](https://github.com/qidunhu/docs/blob/master/images/2.png)

里面定义了pma登录的账号和密码两个变量，由于在cSphere平台里定义的变量是可以交叉引用的，所以在pxc服务里可以直接去引用这两个变量

![pxc](https://github.com/qidunhu/docs/blob/master/images/3.png)


引用其他服务定义的变量方式为```{{.service.服务名.变量名}}``` 	 // 此处的服务名是指模板中定义的服务名称，如图中的pam服务。

在配置文件中引用变量和使用Go template语法灵活配置整个集群
![6](https://github.com/qidunhu/docs/blob/master/images/5.png)



图中的
```
wsrep_cluster_address = gcomm://{{range $i,$c := .Service.Containers}}{{if ne $i 0}},{{end}}{{$c.Domain}}{{end}}
``` 
利用了Go template语法获取其他节点的IP，灵活配置集群。

  [1]: https://github.com/qidunhu/docs/blob/master/images/redis2.jpg "redis2.jpg"
  [2]: https://github.com/qidunhu/docs/blob/master/images/redis3.jpg "redis3.jpg"
  [3]: https://github.com/qidunhu/docs/blob/master/images/1.png "1.png"
  [4]: https://github.com/qidunhu/docs/blob/master/images/2.png "2.png"
  [5]: https://github.com/qidunhu/docs/blob/master/images/3.png "3.png"
  [6]: https://github.com/qidunhu/docs/blob/master/images/5.png "5.png"
