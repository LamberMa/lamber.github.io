!!! 参考
    - [Dockerhub](https://hub.docker.com/r/squidfunk/mkdocs-material/)

下载镜像

```shell
docker pull squidfunk/mkdocs-material
```

然后挂载就可以了，注意挂载到目录要保证对应的存在`mkdocs.yaml`这个文件，如：

```shell
# docker
docker run -d --restart=always -p 127.0.0.1:8000:8000 -v ~/Nutstore\ Files/wiki/:/docs   squidfunk/mkdocs-material
# lima
lima nerdctl run -d --restart=always -p 127.0.0.1:8000:8000 -v ~/Nutstore\ Files/wiki/:/docs   squidfunk/mkdocs-material
```