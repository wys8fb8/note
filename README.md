# 1. 看现状
python scripts/detect_schema_state.py --db-prefix inkframe_dev_

# 2. 假设诊断说 V001-V088 已存在 → 标记 applied（不跑 SQL）
python migrate_manual_mysql.py --db-prefix inkframe_dev_ \
    --mark-applied V001-V088 --dry-run     # 先 dry-run 确认
python migrate_manual_mysql.py --db-prefix inkframe_dev_ \
    --mark-applied V001-V088               # 实际写

# 3. 之后增量自动安全
python migrate_manual_mysql.py --db-prefix inkframe_dev_
