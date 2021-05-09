配置代理

```
// 查看
git config --global https.proxy 
// 配置
git config --global https.proxy http://proxy1.bj.petrochina:8080
git config --global http.proxy http://proxy1.bj.petrochina:8080
```



查看当前仓库地址

```git
git remote show origin
```

设置新的仓库地址

```
git remote set-url origin url...
```

