1、门户、管理后台、艺术家后台、app
2、作品（三类鉴权）
2.1 艺术家上传免费的
2.2 B类确权需要上武汉链，定价非0
2.3 文交所+武汉链

3、 画廊：
3.1 管理员上传的作品,公共免费
3.2 艺术家上传免费和非免费

4、我的图库
4.1 Ai生图
4.2 我的购买
4.3 我的收藏
4.4 我的上传
 数据库表：table1管理员上传的公共版本画, 艺术家的免费和非免费的都放在一起table2,table1和table2都需要添加适配屏字段，价格艺术家和管理员价格都要存。
 我的图库table3(数据源 ai生图，公版，艺术家，我的上传）id,三图，...画的信息，作品类型免费|收费。width,height, 推荐屏，收费的（绑定的设备）
 
5、艺术家入驻
5.1 真实姓名
5.2 昵称
5.3 身份证号码
5.4 手机号码（注册也有了）
5.5 邮箱（注册邮箱有了）
5.6 简介一句话
注册取消上传艺术品

6、 管理后台 
6.1 批量上传 （默认审核通过，但是不上架，有批量操作上架）
6.2 艺术家审核
6.3 艺术品管理（字段分开审核状态和上架状态分开）
6.5 硬件管理
6.6 用户管理
6.7 订单管理
6.8 配置管理


我的图库，除了购买的都可以删除（管理员先不管）



 我现在有个设计需要改动
 1、艺术画廊的数据来源有两种，一种是艺术家上传的作品（免费和收费的）、一种是管理员上传的公版画，这两个数据源头我想分开表来管理，而不是都混在一张表里。管理员上传的作品没有艺术家id的。
 2、目前审核状态，待审，已审核，已拒绝，上架下架都在一个字段管理，我需要拆成两个字段来实现。审核是审核，上下架是上下架。
 3、我的图库数据来源有，AI生成，我的上传，我的收藏（免费的和公版画），我购买的艺术品（收费的有版次的概念）这里我需要这个分类。我想把我的图库中的字段整全包含缩略图，水印图原图，还有数据来源不依赖其他表也能独立有全量数据。
 4、艺术家入驻申请接口，现在艺术家有个需要上传艺术品3-10是可选项目但是我想去掉，等成为艺术家后再上传，更清晰的管理艺术品。身份证正反面也非必须上传，身份证号码需要填写帮添加一个字段。


 UPDATE device_display_queue dq
JOIN (
    SELECT id,
           ROW_NUMBER() OVER (
               PARTITION BY device_id
               ORDER BY added_at ASC, id ASC
           ) AS new_sort_order
    FROM device_display_queue
) t ON dq.id = t.id
SET dq.sort_order = t.new_sort_order;
按 device_id 分组、(added_at, id) 升序重排为 1..N，幂等可重复。建议执行前先备份或加 LIMIT 验证：


-- 预览（不落库）
SELECT id, device_id, artwork_type, sort_order,
       ROW_NUMBER() OVER (PARTITION BY device_id ORDER BY added_at, id) AS new_sort_order
FROM device_display_queue
ORDER BY device_id, new_sort_order;


reorder 接口用法
路径：POST /api/v1/device/display-queue/reorder
认证：用户 Bearer Token（设备所有者）

请求

{
  "device_id": 1,
  "ordered_ids": [3, 1, 2, 5]
}
ordered_ids 必须严格等于该设备当前队列全部 queue_item_id 的集合（即 docs/api/06-设备管理.md:823 display-queue/list 返回的 items[].queue_item_id 全集），不能缺、不能多
数组顺序即为新的 sort_order：第一项 → sort_order=1，第二项 → 2，依此类推
不区分 paid/free（slice-DV-1b 合并队列）
典型用法
先调 GET /api/v1/device/display-queue/list?device_id=1 拿到当前 items[].queue_item_id 列表
前端把列表打乱/拖动后得到新顺序
用新顺序的 queue_item_id 数组调 reorder

curl -X POST "http://localhost:8080/api/v1/device/display-queue/reorder" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"device_id": 1, "ordered_ids": [3, 1, 2, 5]}'
成功响应

{ "success": true, "code": 200, "data": { "reordered": true } }
