---
title: golang -- viper
---



> 读取配置是几乎每个程序都会需要的，viper是golang中读取配置文件比较常用的库。本文大致介绍一些viper的基本用法。



### Why

使用yaml/json去定义struct解析配置文件也不麻烦，但是viper将其做的更加通用，如果后期有需求变化，例如从配置中心，环境变量中获取，配置文件动态加载等需求，使用viper可以更加简单一些。
viper提供了以下能力：
```
默认值设置/直接更新配置项目
支持不同格式配置文件，json，yaml等
从环境变量，flag，缓存中读取配置
支持配置中心读取配置（consul，etcd等）
支持监控配置变化，触发加载新配置
```



### How

使用viper基本用法:
```
db:
  port: "1234"
  addr: "1.2.3.4"
```

例如读取以上配置时，仅需要viper.GetString("db.port")。



以下是初始化配置文件的示例， path是文件目录，name是文件名，confType为类型，例如"yaml":

```
func InitialConf(path, name, confType string) error {
	viper.AddConfigPath(path)
	viper.SetConfigName(name)
	viper.SetConfigType(confType)
	if err := viper.ReadInConfig(); err != nil {
		return err
	}
	return nil
}
```



设置默认值, 可以将配置项Item设置默认值为value:

```
viper.SetDefault("Item", "value")
```



设置值,直接设置Item为value：

```
viper.Set("Item", "value")
```



更多使用方式见，[viper](https://github.com/spf13/viper)