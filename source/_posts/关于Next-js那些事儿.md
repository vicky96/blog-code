---
title: 关于Next.js那些事儿
date: 2019-01-08 14:48:58
tags:
---

为什么需要服务端渲染呢？
主要是便于seo优化，搜索引擎无法收录js渲染出来的内容，像react、vue都是用js渲染的，因此这些普通的单页面应用并不利于seo优化，所以需要服务端渲染。

了解到有一个基于react的服务端渲染的框架Next.js，由于最近正好在研究react，对react比较熟悉，而且为了更进一步了解react相关内容，所以我决定使用Next.js来改造我们公司的官网( http://www.artstep.cn )。不出所料，与react有些不同的地方，使用过程遇到了不少坑。。。

1.同一个页面tab切换和分页切换的时候，直接重新调用getInitialProps的时候并不能在服务端渲染
  这是因为`getInitialProps`方法只会在服务端渲染一次，只有在页面初始化渲染或者路由跳转的时候才会触发，而且不能在子组件中使用。我的解决方法是在切换的时候带着所需的参数重新跳转这个页面。不过需要去`server.js`中配置路由
  ```
    server.get(`/news/:listType/:page`, (req, res) => {
      const actualPage = '/news';
      const queryParams = { listType: req.params.listType, page: req.params.page };
      app.render(req, res, actualPage, queryParams);
    });
  ```
  然后用getInitialProps提供的参数`context`去接收就好啦
  ```
    static async getInitialProps(context){
      ...
    }
  ```
2.外部样式引入报错
  其实官方推荐的是使用`css-in-js`的形式，因为外部引入的css可能会出现一些问题
  ```
     <style jsx>{`
        .banner{
          background: url('../static/img/index/banner.png') no-repeat;
          background-size: 100%;
        }
        ...
    `}</style>
  ```
3.自定义404页面
  next.js会自动包含一个404页面，但是样式和功能并不符合网站需求，所以需要在pages文件夹中自己创建`_error.js`去重写404页面

4.部署到线上之后，需要用`pm2`进程管理工具管理next.js的进程
  配置pm2需要创建一个`pm2.json`
  ```
    {
      "apps": [
        {
          "name": "artstep",
          "cwd": "./",
          "script": "./server.js",
          "log_date_format": "YYYY-MM-DD HH:mm Z",
          "error_file": "/log/node-app/node-app.stderr.log",
          "out_file": "/log/node-app.stdout.log",
          "pid_file": "pids/node-geo-api.pid",
          "instances": 1,
          "min_uptime": "200s",
          "max_restarts": 100,
          "max_memory_restart": "1M",
          "cron_restart": "1 0 * * *",
          "merge_logs": true,
          "watch":true,
          "exec_interpreter": "node",
          "exec_mode": "cluster"
        }
      ]
    }
  ```
  然后在`package.json`中添加启动命令
  ```
    "pm2": "pm2 start pm2.json"
  ```
  这样就可以通过`pm2 start server.js --name artstep`来启动啦

以上只是一部分坑。。。第一次使用next.js，在服务端渲染方面很是方便，但确实也遇到了一些问题，不过做完整个项目之后，发现其实还是挺容易的，只是在react的基础上改变了一些内容而已～