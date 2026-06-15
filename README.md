flowchart TB
    subgraph USER["用户操作"]
        A["访问主页"] --> B["输入目录路径"]
        B --> C["点击扫描"]
        C --> D["选择图片"]
        D --> E["点击打标签"]
        D --> F["搜索/筛选"]
        D --> G["删除图片"]
        D --> H["导出数据库"]
    end

    subgraph API["API层"]
        I["POST scan"] --> J["验证路径"]
        J --> K["递归扫描目录"]
        K --> L["过滤白名单扩展名"]
        L --> M["返回文件列表"]

        N["POST tag"] --> O["验证文件存在"]
        O --> P["创建任务ID"]
        P --> Q["提交到线程池"]
        Q --> R["返回任务ID"]

        S["GET progress_stream"] --> T["订阅进度队列"]
        T --> U["推送SSE事件"]

        V["GET memes"] --> W["分页查询"]
        W --> X["支持筛选/搜索"]

        Y["DELETE memes-id"] --> Z["级联删除记录"]

        AA["GET stats"] --> AB["统计各状态数量"]

        AC["GET file_preview"] --> AD["返回图片二进制"]

        AH2["GET export"] --> AI2["导出数据库文件"]
    end

    subgraph BG["后台任务处理"]
        AE["ThreadPoolExecutor"] --> AF["遍历文件列表"]
        AF --> AG["计算MD5哈希"]
        AG --> AH["获取或创建Meme记录"]
        AH --> AI["状态 pending 转 processing"]
        AI --> AJ["调用AI分析"]
    end

    subgraph AISVC["AI标签分析"]
        AK["读取图片文件"] --> AL["Base64编码"]
        AL --> AM["构建API请求"]
        AM --> AN["设置系统提示词"]
        AN --> AO["发送到LLM API"]
        AO --> AP{"请求成功?"}
        AP -- "失败" --> AQ{"重试小于3次?"}
        AQ -- "是" --> AO
        AQ -- "否" --> AR["返回错误"]
        AP -- "成功" --> AS["解析JSON响应"]
        AS --> AT["提取标签列表"]
        AT --> AU["返回标签"]
    end

    subgraph DB["数据库操作"]
        AV["Meme表"] --> AW["存储文件信息"]
        AX["Tag表"] --> AY["存储标签名"]
        AZ["MemeTag表"] --> BA["关联Meme和Tag"]
        BB["清除旧标签"] --> BC["创建新关联"]
    end

    subgraph FE["前端显示"]
        BD["进度条实时更新"] --> BE["显示当前文件"]
        BE --> BF["显示完成数量"]
        BF --> BG2["任务完成后刷新列表"]
        BH["图片卡片展示"] --> BI["显示标签"]
        BI --> BJ["预览大图"]
        BK["分页导航"] --> BL["状态筛选"]
        BM["搜索框"] --> BN["关键词过滤"]
    end

    C --> I
    E --> N
    F --> V
    G --> Y
    H --> AH2
    I --> M
    N --> R
    Q --> AE
    R --> T
    AJ --> AK
    AU --> BC
    AH --> AW
    BC --> BA
    AU --> BB
    AI --> U
    AR --> U
    W --> BH
    X --> BN
    AD --> BJ
    U --> BD
    BG2 --> BH
