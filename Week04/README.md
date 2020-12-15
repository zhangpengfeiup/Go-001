学习笔记

没有使用Go写过项目，所以参考了krotas的目录结构

|- api                  # api目录是对外保留的proto文件及生成的pb.go文件
|- cmd		        # 项目主干，main所在
|   |-- myapp
|      |--- main.go
|- configs		 # configs 为配置文件目录
| --db.toml					
|- internal              # 项目内部包
|   |--dao               # dao层，用户数据库、cache、MQ等资源访问
|   |--di	         # 依赖注入层 采用wire静态分析依赖
|      |--- wire.go      # wire 声明
|      |--- wire_gen.go  # go generate 生成的代码
|   |--model		 # model 层，用于声明业务结构体
|   |--server            # server层，用于初始化grpc和http serverå
|   |--service           # service层，用于业务逻辑处理
|- test                  # 测试资源层

