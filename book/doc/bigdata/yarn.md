## 常用命令

| 命令                         | 说明               |
| ---------------------------- | ------------------ |
| yarn application --help      | 帮助信息           |
| yarn application -list       | 查看正在运行的 app |
| yarn application -kill appid | 杀死 app           |

```shell
# 根据关键词 杀死所有 app
for i in  `yarn application  -list | grep -w  test | awk '{print $1}' | grep application_`; do yarn  application -kill $i; done
```
