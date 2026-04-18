-- =============================================================================
-- InkFrame 测试数据清理脚本
-- 生成日期: 2026-04-18
-- 说明: 清理所有业务数据，保留配置表、类型表和管理员数据
-- 用法: mysql -u root -p < scripts/cleanup_test_data.sql
-- =============================================================================
-- 注意: 外键已提前清理，可直接 DELETE
-- 保留的表（不清理）:
--   inkframe_admin:  admins, admin_operation_logs, system_configs（整库保留）
--   inkframe_user:   membership_plans（配置表）
--   inkframe_artwork: artwork_categories（类型表）
--   inkframe_product: product_categories（类型表）
--   inkframe_device:  device_models（类型表）
--   inkframe_notification: notification_templates（配置表）
--   inkframe_ai:     ai_styles（配置表）
--   inkframe_points: points_recharge_packages（配置表）
-- =============================================================================

SET FOREIGN_KEY_CHECKS = 0;

-- =============================================================================
-- 1. inkframe_user — 用户服务
-- =============================================================================
-- 保留: membership_plans（会员套餐配置）
-- 清理: 用户、地址、会员记录、OAuth、会话、通知偏好、关注
-- =============================================================================
USE inkframe_user;

DELETE FROM user_follows;
DELETE FROM user_notification_prefs;
DELETE FROM user_sessions;
DELETE FROM user_oauth_bindings;
DELETE FROM user_oauth;
DELETE FROM user_memberships;
DELETE FROM user_addresses;
DELETE FROM users;

-- =============================================================================
-- 2. inkframe_artist — 艺术家服务
-- =============================================================================
-- 保留: 无配置表
-- 清理: 全部
-- =============================================================================
USE inkframe_artist;

DELETE FROM artist_withdraw_requests;
DELETE FROM artist_bank_accounts;
DELETE FROM artist_earnings_records;
DELETE FROM artist_earnings_accounts;
DELETE FROM artist_kyc_records;
DELETE FROM artists;

-- =============================================================================
-- 3. inkframe_artwork — 艺术品服务
-- =============================================================================
-- 保留: artwork_categories（作品分类）
-- 清理: 作品、审核、资产定义、数字藏品、点赞、收藏、图库
-- =============================================================================
USE inkframe_artwork;

DELETE FROM user_library;
DELETE FROM wishlists;
DELETE FROM artwork_likes;
DELETE FROM digital_collectibles;
DELETE FROM asset_definitions;
DELETE FROM artwork_review_records;
DELETE FROM artworks;

-- =============================================================================
-- 4. inkframe_copyright — 版权服务
-- =============================================================================
-- 保留: 无配置表
-- 清理: 全部
-- =============================================================================
USE inkframe_copyright;

DELETE FROM exchange_certifications;
DELETE FROM blockchain_records;
DELETE FROM copyright_certifications;

-- =============================================================================
-- 5. inkframe_product — 商品服务
-- =============================================================================
-- 保留: product_categories（商品分类）
-- 清理: 商品、SKU、库存日志
-- =============================================================================
USE inkframe_product;

DELETE FROM inventory_logs;
DELETE FROM product_skus;
DELETE FROM products;

-- =============================================================================
-- 6. inkframe_order — 订单服务
-- =============================================================================
-- 保留: 无配置表
-- 清理: 全部
-- =============================================================================
USE inkframe_order;

DELETE FROM shipping_events;
DELETE FROM shipping_records;
DELETE FROM order_items;
DELETE FROM orders;
DELETE FROM cart_items;
DELETE FROM checkout_sessions;

-- =============================================================================
-- 7. inkframe_payment — 支付服务
-- =============================================================================
-- 保留: 无配置表
-- 清理: 全部
-- =============================================================================
USE inkframe_payment;

DELETE FROM refund_records;
DELETE FROM payment_transactions;

-- =============================================================================
-- 8. inkframe_device — 设备服务
-- =============================================================================
-- 保留: device_models（设备型号配置）
-- 清理: 设备、展示队列、推送历史、NFT 转让、网关绑定、播放日志、展示状态、图片变体
-- =============================================================================
USE inkframe_device;

DELETE FROM device_play_log;
DELETE FROM device_display_state;
DELETE FROM artwork_variants;
DELETE FROM device_gateway_bindings;
DELETE FROM device_nft_transfers;
DELETE FROM device_push_history;
DELETE FROM device_display_queue;
DELETE FROM devices;

-- =============================================================================
-- 9. inkframe_notification — 通知服务
-- =============================================================================
-- 保留: notification_templates（通知模板配置）
-- 清理: 通知记录、邮件日志、短信日志
-- =============================================================================
USE inkframe_notification;

DELETE FROM sms_send_logs;
DELETE FROM email_send_logs;
DELETE FROM notifications;

-- =============================================================================
-- 10. inkframe_ai — AI 服务
-- =============================================================================
-- 保留: ai_styles（AI 风格配置）
-- 清理: AI 生图任务
-- =============================================================================
USE inkframe_ai;

DELETE FROM ai_generation_tasks;

-- =============================================================================
-- 11. inkframe_points — 积分服务
-- =============================================================================
-- 保留: points_recharge_packages（充值包配置）
-- 清理: 积分账户、积分流水
-- =============================================================================
USE inkframe_points;

DELETE FROM points_transactions;
DELETE FROM points_accounts;

-- =============================================================================
-- 12. inkframe_admin — 管理后台（整库保留，不清理）
-- =============================================================================
-- 保留: admins, admin_operation_logs, system_configs
-- 说明: 管理员账号和系统配置不清理
-- =============================================================================

SET FOREIGN_KEY_CHECKS = 1;

-- =============================================================================
-- 完成
-- =============================================================================
SELECT '清理完成' AS status;






-- V088: ai_generation_tasks 增加 artwork_id 列
-- 保存到图库后回写对应的作品 ID，便于前端关联跳转
ALTER TABLE ai_generation_tasks ADD COLUMN artwork_id BIGINT NULL DEFAULT NULL;


已有的生图记录如果通过 save-to-library 或 auto_save 保存过作品，可以通过 result_object_key 关联回去。回填 SQL：


-- 回填已有 AI 生图任务的 artwork_id（通过 original_object_key 关联）
UPDATE ai_generation_tasks t
  JOIN artworks a ON a.original_object_key = t.result_object_key
SET t.artwork_id = a.id
WHERE t.artwork_id IS NULL
  AND t.result_object_key IS NOT NULL
  AND t.result_object_key != '';
执行前可以先 SELECT 验一下匹配数量：


SELECT COUNT(*)
FROM ai_generation_tasks t
  JOIN artworks a ON a.original_object_key = t.result_object_key
WHERE t.artwork_id IS NULL
  AND t.result_object_key IS NOT NULL
  AND t.result_object_key != '';
如果量不大直接跑就行，不影响现有数据。



¥¥¥¥¥¥¥¥¥
对，各服务独立数据库，不能 JOIN。分两步：

第 1 步：在 artwork 库查映射


-- 在 artwork_service 的数据库执行
SELECT id AS artwork_id, original_object_key
FROM artworks
WHERE original_object_key LIKE 'originals/ai/%'
  AND original_object_key != '';
把结果拿到后，第 2 步：在 ai 库逐条更新


-- 在 ai_service 的数据库执行
-- 用第 1 步查出的结果替换 VALUES 里的内容
UPDATE ai_generation_tasks
SET artwork_id = CASE result_object_key
    WHEN 'originals/ai/200/20260415_xxx.jpg' THEN 5001
    WHEN 'originals/ai/100/20260416_yyy.jpg' THEN 5002
    -- ... 第 1 步查出多少条就填多少行
END
WHERE result_object_key IN (
    'originals/ai/200/20260415_xxx.jpg',
    'originals/ai/100/20260416_yyy.jpg'
    -- ... 同上
)
AND artwork_id IS NULL;
如果数据量不大（几十条以内），也可以直接生成单条 UPDATE：


-- 每条一句，简单粗暴
UPDATE ai_generation_tasks SET artwork_id = 5001 WHERE result_object_key = 'originals/ai/200/20260415_xxx.jpg' AND artwork_id IS NULL;
UPDATE ai_generation_tasks SET artwork_id = 5002 WHERE result_object_key = 'origin

