# 项目结构

文档维护 | 辰枫
---|---
更新日期 | 2017-11-24
文档版本 | v1.0


```
│  main.go
│  README.md
├─app
│  ├─actions    # 业务
│  │  │  load.go
│  │  │
│  │  └─admin
│  │          2001.go
│  │          2001_test.go
│  │
│  ├─core
│  │      core.go
│  │      manager.go
│  │
│  ├─dal    # 数据访问层
│  │      areachatenv.go
│  │      areachatenv_test.go
│  │      db.go
│  │      db_test.go
│  │
│  ├─lang   # Panic语言
│  │      english.go
│  │      simplified.go
│  │      traditional.go
│  │
│  └─tools
│          http.go
│          jwt.go
│          jwt_test.go
│          netip.go
│          netip_test.go
│          tool.go
│          tool_test.go
│
├─config
│      base.go
│      base_test.go
│      config.go
│      config_test.go
│
└─error
        erros.go

```