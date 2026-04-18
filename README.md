UPDATE product_skus SET available_stock = stock - locked_stock WHERE available_stock = 0 AND stock > 0;
-- 1. 修复 artworks 表价格为空的记录
UPDATE artworks a JOIN asset_definitions ad ON a.id = ad.artwork_id
SET a.price = ad.price_per_edition, a.total_supply = ad.edition_total
WHERE a.price IS NULL OR a.total_supply = 0;

-- 2. 修复 SKU 的 available_stock（上一个 bug）
UPDATE product_skus SET available_stock = stock - locked_stock
WHERE available_stock = 0 AND stock > 0;
