INSERT OR IGNORE INTO system_configs (group_key, config_key, config_value, is_encrypted, description) VALUES
('bailian', 'api_key', '你的百炼API Key', 1, '百炼 API Key'),
('bailian', 'api_url', 'https://dashscope.aliyuncs.com/api/v1/services/aigc/multimodal-generation/generation', 0, '百炼 API 地址（北京）'),
('bailian', 'model', 'wan2.7-image', 0, '模型名称'),
('bailian', 'timeout', '180', 0, '请求超时（秒）'),
('bailian', 'size', '2K', 0, '输出分辨率（1K/2K/4K）');

INSERT OR IGNORE INTO system_configs (group_key, config_key, config_value, is_encrypted, description) VALUES
('ai', 'provider', 'bailian', 0, 'AI 生图 provider（bailian / gemini）'),
('ai', 'image_cost_points', '10', 0, '每次 AI 生图消耗点数');

INSERT INTO ai_styles (name, code, prompt_prefix, preview_url, description, sort_order, is_active)
VALUES
  ('水墨画',   'ink_wash',     'in traditional Chinese ink wash painting style',  NULL, '中国传统水墨画风格',   1, 1),
  ('油画',     'oil',          'in oil painting style',                           NULL, '经典油画风格',         2, 1),
  ('素描',     'sketch',       'in pencil sketch style',                          NULL, '铅笔素描风格',         3, 1),
  ('极简线条', 'minimal_line', 'in minimal line art style',                       NULL, '极简线条艺术风格',     4, 1),
  ('几何抽象', 'geometric',    'in geometric abstract art style',                 NULL, '几何抽象艺术风格',     5, 1),
  ('浮世绘',   'ukiyo_e',      'in Japanese ukiyo-e woodblock print style',       NULL, '日本浮世绘版画风格',   6, 1),
  ('黑白摄影', 'bw_photo',     'in black and white photography style',            NULL, '黑白摄影风格',         7, 1),
  ('波普艺术', 'pop_art',      'in pop art style',                                NULL, '波普艺术风格',         8, 1);
