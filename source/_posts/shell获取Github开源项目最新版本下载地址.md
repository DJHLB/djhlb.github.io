---
title: shell获取Github开源项目最新版本下载地址
date: 2024-01-22 17:55:44
tags:
---
## 下载指定的打包文件

```
releases_url=https://api.github.com/repos/open-tdp/tdp-cloud/releases/latest
tag_name=`wget -qO- $releases_url | grep tag_name | cut -f4 -d "\""`

wget https://github.com/open-tdp/tdp-cloud/releases/download/${tag_name}/tdp-cloud-android-arm64.gz
```

## 批量下载所有打包文件

```
releases_url=https://api.github.com/repos/open-tdp/tdp-cloud/releases/latest
url_list=`wget -qO- $releases_url | grep releases/download | cut -f4 -d "\""`

for url in $url_list; do
    wget $url
done
```