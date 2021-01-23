---
title: 使用特里树匹配动态路由
date: 2021-01-23 15:05:21
tags: [算法,算法,特里树,路由匹配,Trie Tree]

---

### 使用特里树匹配动态路由
#### 问题
在使用网关（gateway）进行URL权限匹配的时候，需要根据客户端请求的URL进行路由匹配，然后进行权限校验；

参考：[微服务统一认证鉴权](https://www.cnblogs.com/haoxianrui/p/13719356.html)

一般的网关会把URL与权限对应关系存在数据库，类似这样：

|    NAME  |    URL                 |    ROLES |
|    ----  |    ----                |    ----  |
| 用户管理  | /user/list             |USER       |
| 用户管理  | /user/detail/{id}      |USER       |
| 资源管理  | /demo/demo/**          |USER       |
| 管理员    | /admin/**              |ADMIN      |

然后将权限数据加载进内存，根据每次请求时，匹配路由和权限，来决定是否放行；
```java
        //创建权限字典：
        Map<String, List<String>> paths = new HashMap<>();
        paths.put("/user/list", Arrays.asList("USER", "ADMIN"));
        paths.put("/user/detail/{id}", Arrays.asList("USER", "ADMIN"));
        paths.put("/demo/demo/**", Arrays.asList("ADMIN"));
        paths.put("/admin/**", Arrays.asList("ADMIN"));

        Set<String> auths = new HashSet<>();
        //路由匹配：
        AntPathMatcher pathMatcher = new AntPathMatcher();
        String path = exchange.getRequest().getURI().getPath();
        //遍历权限字典，进行匹配：
        paths.forEach((k, v) -> {
            String pattern = k;
            if (pathMatcher.match(pattern, path)) {
                auths.addAll(v);
            }
        });
```
*参考：https://github.com/imwower/spring-cloud-gateway/blob/main/src/main/java/com/example/gateway/config/AuthorizationManager.java*

但是每次调用，都需要遍历所有的权限字典数据；时间复杂度为：`n`，`n`为URL总数；
所以，当网关配置的URL很多时，效率肯定不行；

#### 解决思路

考虑使用特里树（Trie Tree）进行路由匹配；
特里树，是类似下图的一种数据结构：

![特里树](/images/trie-tree-and-route-match/trie-tree.jpg)

1. 将字符串重复的前缀部分，作为树结构的父节点；
2. 在进行路由匹配时，根据URL中的斜杠进行分割；然后根据每个分段与特里树的节点进行匹配；一级一级向下匹配即可；
3. 时间复杂度为：URL的斜杠的个数，也就是类似文件夹的层级；时间复杂度基本为常数；

一些路由规则：
- `/user/logout`  `/user/list` 这种路由作为静态路由；
- `/user/{id}`  这种路由作为动态路由；一般只允许有一个动态路由；
- `/admin/**`  这种路由则作为通配符路由；通配符路由一般也是只有一个；

#### 示例代码

以下是Java示例代码：
```java
    //子节点
   private class Node {
       private String path;
       private String segment;
       private Map<String, Node> staticRouters;
       private Node dynamicRouter;
       private boolean isWildcard; //是否是通配符路由
   }
```
```java
    public class Router {
        private Node root;
    
        public Router() {
            root = new Node();
            root.path = "/";
            root.segment = "/";
        }

        //新增路由
        public Router addRoute(String path) {
            if (!StringUtils.isEmpty(path)) {
                String strippedPath = StringUtils.strip(path, "/");
                String[] strings = StringUtils.split(strippedPath, "/");
                if (strings.length != 0) {
                    Node node = root;
                    //按照斜杠分割：
                    for (int i = 0; i < strings.length; i++) {
                        String segment = strings[i];
                        //添加节点：
                        node = addNode(node, segment);
                        if ("**".equals(segment))
                            break;
                    }
                    //结束时，设置子节点的path：
                    node.path = path;
                }
            }
            return this;
        }

        //添加节点：
        private Node addNode(Node node, String segment) {
            //如果是通配符节点，直接返回当前节点：
            if ("**".equals(segment)) {
                node.isWildcard = true;
                return node;
            }
            //如果是动态路由，则创建一个子节点，然后将子节点挂在当前节点下：
            if (segment.startsWith("{") && segment.endsWith("}")) {
                Node childNode = new Node();
                childNode.segment = segment;
                node.dynamicRouter = childNode;
                return childNode;
            }
    
            Node childNode;
            //静态路由，放到一个Map里，map的key为URL分段，value为新的子节点：
            if (node.staticRouters == null)
                node.staticRouters = new HashMap<>();
            if (node.staticRouters.containsKey(segment))
                childNode = node.staticRouters.get(segment);
            else {
                childNode = new Node();
                childNode.segment = segment;
                node.dynamicRouter = childNode;
                node.staticRouters.put(segment, childNode);
            }
            return childNode;
        }

        //匹配路由：
        public String matchRoute(String path) {
                if (!StringUtils.isEmpty(path)) {
                    String strippedPath = StringUtils.strip(path, "/");
                    String[] strings = StringUtils.split(strippedPath, "/");
                    if (strings.length != 0) {
                        Node node = root;
                        //按照斜杠分割：
                        for (int i = 0; i < strings.length; i++) {
                            String segment = strings[i];
                            node = matchNode(node, segment);
                            //如果没有匹配到，或者是通配符路由，退出：
                            if (node == null || node.isWildcard)
                                break;
                        }
                        if (node != null)
                            return node.path;
                    }
                }
                return null;
            }

        //匹配子节点：
        private Node matchNode(Node node, String segment) {
                if (node.staticRouters != null && node.staticRouters.containsKey(segment)) {
                    return node.staticRouters.get(segment);
                }
                if (node.dynamicRouter != null)
                    return node.dynamicRouter;
                if (node.isWildcard)
                    return node;
                return null;
            }

    }

```

使用示例：
```java
       Router router = new Router();
       router.addRoute("/user/**");
       router.addRoute("/user");
       router.addRoute("/user/{userId}");
       router.addRoute("/equipment/**");
       router.addRoute("/user/bg/**");

       System.out.println("OK");

       String route = router.matchRoute("/user/bg/**");
       System.out.println(route);
       route = router.matchRoute("/user/123");
       System.out.println(route);
       route = router.matchRoute("/user");
       System.out.println(route);
```

#### 其他
一些细节问题，比如，多个动态路由、单星号的通配符等，没有优化；有需要时再优化吧。

#### 参考
参考链接：
- [微服务统一认证鉴权](https://www.cnblogs.com/haoxianrui/p/13719356.html)
- [前缀树算法实现路由匹配原理解析](https://segmentfault.com/a/1190000021657573)
- [示例代码](https://github.com/imwower/router)