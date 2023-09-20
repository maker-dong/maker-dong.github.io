# Python离线安装依赖包

# 依赖导出

````shell
pip freeze > requirement.txt
````

# 下载离线依赖包

````shell
pip download -d package_whl/ -r requirements.txt
````

如果下载过慢或网络出现问题，可以指定国内镜像源，此处以中科大源为例

```
pip download -d package_whl/ -r requirements.txt -i https://pypi.mirrors.ustc.edu.cn/simple/
```



# 离线安装依赖包

## 安装执行依赖

````shell
pip install --no-index --find-links=package_whl/* pyinstaller
````

## 安装全部依赖

````shell
pip install  package_whl/*
````

## 按照requirements文件安装依赖

将`requirement.txt`文件上传到依赖存储的目录并执行安装

```sell
pip install -r requirement.txt
```

