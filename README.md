INSERT OR IGNORE INTO system_configs (group_key, config_key, config_value, is_encrypted, description) VALUES
('bailian', 'api_key', '你的百炼API Key', 1, '百炼 API Key'),
('bailian', 'api_url', 'https://dashscope.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation', 0, '百炼 API 地址（北京）'),
('bailian', 'model', 'wan2.7-image', 0, '模型名称'),
('bailian', 'timeout', '180', 0, '请求超时（秒）'),
('bailian', 'size', '2K', 0, '输出分辨率（1K/2K/4K）');

INSERT OR IGNORE INTO system_configs (group_key, config_key, config_value, is_encrypted, description) VALUES
('ai', 'provider', 'bailian', 0, 'AI 生图 provider（bailian / gemini）'),
('ai', 'image_cost_points', '10', 0, '每次 AI 生图消耗点数');
