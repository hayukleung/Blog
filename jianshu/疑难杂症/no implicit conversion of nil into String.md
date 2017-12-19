## 报错信息
```
While executing gem ... (TypeError) 
    no implicit conversion of nil into String
```
## 解决方法
```
rm /usr/local/bin/update_rubygems
```
```
gem update --system
```