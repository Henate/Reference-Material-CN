此文档翻译于Drone官方README.md，如需查看最新的更新文档以及源代码，请移步[官方Github](https://github.com/drone/drone)

Drone是建立于容器技术上的持续交付系统。Drone使用一个简单的YAML配置文件，一个docker-compose的超集，去定义与执行Docker容器内部的Pipelines。

<br/>

<img src="https://github.com/drone/brand/blob/master/screenshots/screenshot_build_success.png" style="max-width:100px;" />

Pipeline配置的示例：:

```yaml
pipeline:
  backend:
    image: golang
    commands:
      - go get
      - go build
      - go test

  frontend:
    image: node:6
    commands:
      - npm install
      - npm test

  publish:
    image: plugins/docker
    repo: octocat/hello-world
    tags: [ 1, 1.1, latest ]
    registry: index.docker.io

  notify:
    image: plugins/slack
    channel: developers
    username: drone
```

文件以及其他链接：

* 安装文件 [docs.drone.io/installation](http://docs.drone.io/installation/)
* 使用文件 [docs.drone.io/getting-started](http://docs.drone.io/getting-started/)
* 插件索引 [plugins.drone.io](http://plugins.drone.io/)
* 获得帮助 [docs.drone.io/getting-help](http://docs.drone.io/getting-help/)
