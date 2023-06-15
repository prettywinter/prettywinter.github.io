---
layout: wiki
wiki: Java
title: Java解析MySQL的binlog日志
order: 160
---

1. 引入 binlog 的解析库

```xml pom.xml
<dependency>
    <groupId>com.zendesk</groupId>
    <artifactId>mysql-binlog-connector-java</artifactId>
    <version>0.28.0</version>
</dependency>
```

> 网络上很多使用 https://github.com/shyiko/mysql-binlog-connector-java 的，作者在 github 上已经表明此库不在维护。
> 现在维护的是：https://github.com/osheroff/mysql-binlog-connector-java 库，浪子使用的也是该库的依赖。

2. 编码

```yml
mysql:
  binlog:
    hostname: localhost
    port: 3306
    schema: tb_exp
    username: root
    password: root
    table-list: exp_user
```

```java
@Data
@Component
@ConfigurationProperties(prefix = "mysql.binlog")
public class BinlogProperty {

    private String hostname;
    private String port;
    private String username;
    private String password;
    /**
     * 数据库
     */
    private String schema;
    /**
     * 监听的表集合
     */
    private List<String> tableList;
}
```

```java
/**
 * binlog
 *
 * @author jhlz
 * @since 2023/5/29 9:57
 */
@Component
public class BinlogUtil1 {
    private static final Logger log = LoggerFactory.getLogger(BinlogUtil.class);
    @Autowired
    private BinlogProperty binlogProperty;
    /**
     * 数据库名称 map
     */
    private static final Map<Long, String> TABLE_MAP = new HashMap<>();

    /**
     * 异步线程处理 binlog，防止阻塞主线程
     */
    @PostConstruct
    public void handle() {
        ThreadUtil.execAsync(() -> {
            binlogListening();

            log.info("正在监听 <{}:{}:{}> binlog……",
                    binlogProperty.getHostname(),
                    binlogProperty.getSchema(),
                    binlogProperty.getTableList());
        });
    }

    /**
     * binlog 监听
     */
    public void binlogListening() {
        BinaryLogClient client = new BinaryLogClient(binlogProperty.getHostname(),
                Integer.valueOf(binlogProperty.getPort()),
                binlogProperty.getUsername(),
                binlogProperty.getPassword());
        client.setServerId(1);
        // 注册监听
        client.registerEventListener(event -> {
            EventData data = event.getData();
            EventType eventType = event.getHeader().getEventType();

            if (Objects.equals(EventType.TABLE_MAP, eventType)) {
                TableMapEventData tableMapEventData = (TableMapEventData) data;
                String database = tableMapEventData.getDatabase();
                String tableName = tableMapEventData.getTable();
                // 目标明确！缓存指定数据库以及指定的表名
                if (Objects.equals(binlogProperty.getSchema(), database)
                        && binlogProperty.getTableList().contains(tableName)) {
                    TABLE_MAP.put(tableMapEventData.getTableId(), tableName);
                }
            }
            // 新增的 binlog 数据
            if (EventType.isWrite(eventType)) {
                WriteRowsEventData wr = (WriteRowsEventData) data;
                // 获取表名称
                String tableName = TABLE_MAP.get(wr.getTableId());
                if (Objects.nonNull(tableName)) {
                    log.info("{} =============== INSERT", tableName);
                    System.out.println(wr.toString());
                }
            }
            // 更新的 binlog 数据
            if (EventType.isUpdate(eventType)) {
                UpdateRowsEventData ur = (UpdateRowsEventData) data;
                String tableName = TABLE_MAP.get(ur.getTableId());
                if (Objects.nonNull(tableName)) {
                    log.info("{} =============== UPDATE", tableName);
                    System.out.println(ur.toString());
                }
            }
        });
        // 连接服务
        connectServer(client);
    }

    /**
     * 获取指定索引位置的最新值，如果是批量，只拿第一条记录的值
     *
     * @param data  binlog 数据
     * @param index 数据库目标字段的索引位置，起始索引 0
     * @return 目标索引位置的值或者 null
     */
    public static String getValueByIndex(EventData data, int index) {
        // 新增事件
        if (data instanceof WriteRowsEventData) {
            WriteRowsEventData wr = (WriteRowsEventData) data;
            BitSet includedColumns = wr.getIncludedColumns();
            // 如果该位置有值为 true，否则是 false
            if (includedColumns.get(index)) {
                List<Serializable[]> rows = wr.getRows();
                // 只拿第一条记录的值
                Serializable[] arr = rows.get(0);
                // 需要判空，如果一条数据中某列的值为 null 则 NPE
                if (Objects.nonNull(arr[index])) {
                    return arr[index].toString();
                }
            }
        }
        // 更新事件
        if (data instanceof UpdateRowsEventData) {
            UpdateRowsEventData ur = (UpdateRowsEventData) data;
            BitSet includedColumns = ur.getIncludedColumns();
            if (includedColumns.get(index)) {
                // 只获取第一条记录
                Map.Entry<Serializable[], Serializable[]> entry = ur.getRows().get(0);
                // entry 的 key 为旧值，value 为更新值
                Serializable[] newValue = entry.getValue();
                if (newValue[index] != null) {
                    return newValue[index].toString();
                }
            }
        }
        return null;
    }

    /**
     * 获取事件数据指定索引位置的值
     *
     * @param data  binlog 数据
     * @param index 数据库目标字段的索引位置
     * @param key   数据库目标字段的名称
     * @return map，键为传入的 key 值，值为 binlog 读取的新值
     */
    private static List<Map<String, Object>> getValueByIndex(EventData data, int index, String key) {
        List<Map<String, Object>> res = new ArrayList<>();
        if (data instanceof WriteRowsEventData) {
            WriteRowsEventData wr = (WriteRowsEventData) data;
            BitSet includedColumns = wr.getIncludedColumns();
            if (includedColumns.get(index)) {
                // 获取每一行的值，Serializable[] 一条记录的所有值
                List<Serializable[]> rows = wr.getRows();
                // TODO 批量插入不清楚什么格式，先循环吧，使用 MAP 装载结果，后面出问题再调试
                rows.forEach(r -> {
                    // 拿到索引位的值
                    Serializable value = r[index];
                    // 若 value 为 null，则抛出 NPE
                    if (Objects.nonNull(value)) {
                        Map<String, Object> map = new HashMap<>();
                        map.put(key, value);
                        res.add(map);
                    }
                });
            }
        }
        if (data instanceof UpdateRowsEventData) {
            UpdateRowsEventData ur = (UpdateRowsEventData) data;
            BitSet includedColumns = ur.getIncludedColumns();
            if (includedColumns.get(index)) {
                ur.getRows().forEach(u -> {
                    Serializable[] newValue = u.getValue();
                    Serializable obj = newValue[index];
                    if (Objects.nonNull(obj)) {
                        Map<String, Object> map = new HashMap<>();
                        map.put(key, obj);
                        res.add(map);
                    }
                });
            }
        }
        return res;
    }

    /**
     * 连接服务器
     *
     * @param client 客户端
     */
    private static void connectServer(BinaryLogClient client) {
        try {
            client.connect();
        } catch (IOException e) {
            log.error("连接失败：{}", e.getMessage());
            throw new RuntimeException(e);
        }
    }
}
```
