UPDATE product_skus SET available_stock = stock - locked_stock WHERE available_stock = 0 AND stock > 0;
