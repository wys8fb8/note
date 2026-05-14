python migrate_manual_mysql.py --db-prefix inkframe_dev_ --tolerate-existing
[OK] sqlmigrate_mysql/ 下共 114 条迁移，全部已映射。
[OK] inkframe_dev_user: 0 条新迁移执行完成，15 条已应用跳过
[OK] inkframe_dev_artist: 0 条新迁移执行完成，12 条已应用跳过
  [EXEC] inkframe_dev_artwork: V113 ... OK
  [EXEC] inkframe_dev_artwork: V114 ... OK
  [EXEC] inkframe_dev_artwork: V115 ... OK
  [EXEC] inkframe_dev_artwork: V116 ... OK
  [EXEC] inkframe_dev_artwork: V117 ... OK
[OK] inkframe_dev_artwork: 5 条新迁移执行完成，27 条已应用跳过
[OK] inkframe_dev_copyright: 0 条新迁移执行完成，6 条已应用跳过
[OK] inkframe_dev_product: 0 条新迁移执行完成，6 条已应用跳过
[OK] inkframe_dev_order: 0 条新迁移执行完成，10 条已应用跳过
[OK] inkframe_dev_payment: 0 条新迁移执行完成，5 条已应用跳过
  [EXEC] inkframe_dev_device: V118 ... [ERROR] V118__display_chain_keyed_by_library_item.sql @ inkframe_dev_device: [1091] Can't DROP 'idx_display_queue_artwork'; check that column/key exists
        statement: DROP INDEX idx_display_queue_artwork ON device_display_queue
