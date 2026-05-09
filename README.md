 python migrate_manual_mysql.py --db-prefix inkframe_dev_ 
[OK] sqlmigrate_mysql/ 下共 97 条迁移，全部已映射。
[OK] inkframe_dev_user: 0 条新迁移执行完成，13 条已应用跳过
[OK] inkframe_dev_artist: 0 条新迁移执行完成，10 条已应用跳过
  [EXEC] inkframe_dev_artwork: V101 ... OK
[OK] inkframe_dev_artwork: 1 条新迁移执行完成，21 条已应用跳过
[OK] inkframe_dev_copyright: 0 条新迁移执行完成，6 条已应用跳过
[OK] inkframe_dev_product: 0 条新迁移执行完成，6 条已应用跳过
[OK] inkframe_dev_order: 0 条新迁移执行完成，10 条已应用跳过
[OK] inkframe_dev_payment: 0 条新迁移执行完成，5 条已应用跳过
  [EXEC] inkframe_dev_device: V077 ... [ERROR] V077__display_queue_add_crop_coords.sql @ inkframe_dev_device: [1060] Duplicate column name 'crop_x'
PS D:\公司项目\电子纸\xiyiartgithub\xiyiart> python migrate_manual_mysql.py --db inkframe_dev_artwork                     
[OK] sqlmigrate_mysql/ 下共 97 条迁移，全部已映射。
[ERROR] 未知数据库: inkframe_dev_artwork（前缀=inkframe_），候选: inkframe_user, inkframe_artist, inkframe_artwork, inkframe_copyright, inkframe_product, inkframe_order, inkframe_payment, inkframe_device, inkframe_notification, inkframe_ai, inkframe_points, inkframe_admin
PS D:\公司项目\电子纸\xiyiartgithub\xiyiart> python migrate_manual_mysql.py --db-prefix inkframe_dev_  --mark-applied V077
[OK] sqlmigrate_mysql/ 下共 97 条迁移，全部已映射。
[MARK-APPLIED] 准备把 1 个版本标记为已应用：V077
[OK] inkframe_dev_user: 标记 0 个版本
[OK] inkframe_dev_artist: 标记 0 个版本
[OK] inkframe_dev_artwork: 标记 0 个版本
[OK] inkframe_dev_copyright: 标记 0 个版本
[OK] inkframe_dev_product: 标记 0 个版本
[OK] inkframe_dev_order: 标记 0 个版本
[OK] inkframe_dev_payment: 标记 0 个版本
  [MARK] inkframe_dev_device: V077
[OK] inkframe_dev_device: 标记 1 个版本
[OK] inkframe_dev_notification: 标记 0 个版本
[OK] inkframe_dev_ai: 标记 0 个版本
[OK] inkframe_dev_points: 标记 0 个版本
[OK] inkframe_dev_admin: 标记 0 个版本

[DONE] 共 12 个库，已标记 1 条版本。
PS D:\公司项目\电子纸\xiyiartgithub\xiyiart> python migrate_manual_mysql.py --db-prefix inkframe_dev_                     
[OK] sqlmigrate_mysql/ 下共 97 条迁移，全部已映射。
[OK] inkframe_dev_user: 0 条新迁移执行完成，13 条已应用跳过
[OK] inkframe_dev_artist: 0 条新迁移执行完成，10 条已应用跳过
[OK] inkframe_dev_artwork: 0 条新迁移执行完成，22 条已应用跳过
[OK] inkframe_dev_copyright: 0 条新迁移执行完成，6 条已应用跳过
[OK] inkframe_dev_product: 0 条新迁移执行完成，6 条已应用跳过
[OK] inkframe_dev_order: 0 条新迁移执行完成，10 条已应用跳过
[OK] inkframe_dev_payment: 0 条新迁移执行完成，5 条已应用跳过
  [EXEC] inkframe_dev_device: V085 ... [ERROR] V085__add_display_queue_image_dimensions.sql @ inkframe_dev_device: [1060] Duplicate column name 'image_width'
PS D:\公司项目\电子纸\xiyiartgithub\xiyiart> python migrate_manual_mysql.py --db inkframe_dev_artwork                     
[OK] sqlmigrate_mysql/ 下共 97 条迁移，全部已映射。
[ERROR] 未知数据库: inkframe_dev_artwork（前缀=inkframe_），候选: inkframe_user, inkframe_artist, inkframe_artwork, inkframe_copyright, inkframe_product, inkframe_order, inkframe_payment, inkframe_device, inkframe_notification, inkframe_ai, inkframe_points, inkframe_admin
