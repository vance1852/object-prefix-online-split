# 对象命名空间的在线分片与续页

存储联调使用合成设备、时间轴和原始字节。设备记录由各客户端及介质的输入组成，每份记录可独立使用。

## 业务协议

对象键按 UTF-8 字节次序比较，范围用 `start` 含端点、`end` 不含端点，可用空值表示无界。客户上传及删除的 `request` 标识在发送方内唯一，写成功后可按它追查决定。初始 `entries` 的 `file` 指真实对象字节。`routing` 给初始拥有者和路由代次。
本地存储节点用 `POST /nodes/{node}/copies/{transfer}` 接受搬运回报；在资料里 `copy` 是该节点确实收到的动作，可能晚于来源真正发生提交的时刻。 `origin_pos` 是来源流中的持久位置。`checkpoint` 中的 `through` 是目标能证明已连续落盘的来源位置。删除动作独立占一个位置。分页时间使用本地时轴，`until` 表示末端不包含。

## 本地设备资料

运行环境为 Python 3.11.2，使用标准库。设备资料端默认仅在 127.0.0.1:8090 运行，使用 --port 可另设端口。

~~~sh
python3 device_fixture.py fixtures/listing-crosses-a-live-split
~~~

GET /fixture/events 返回设备参数、字节目录和按到达顺序记录的输入。同一时刻按文件先后排列。GET /fixture/objects/文件名 取原始字节。GET /fixture/journal 查看资料端已收到的设备请求，每个请求记录实际正文长度和摘要。

events.json 中的 packets 是可重放的设备响应。资料端按实际 HTTP 方法、路径和完整原始请求正文匹配第一条尚未使用的记录。没有匹配时返回 404。lag_ms 是回包延迟，drop 代表设备收到请求后直接断开连接，接收方需要以其他报告确认结果。没有 HTTP 交互的设备和客户端动作直接取 events。重新启动资料端会回到记录开头。

在项目根目录运行 python3 -m unittest -v 检查事件、对应字节、设备响应及接收日志。
