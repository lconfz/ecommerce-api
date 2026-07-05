# 电商 API 测试报告

## 一、测试概述
- 项目：ecommerce-api
- 测试时间：2026-07-05
- 测试工具：Postman
- 测试人员：lconf

## 二、功能覆盖测试
18 个接口全部跑通

## 三、参数验证测试
已测：8 项
通过：6 项
发现 Bug：2 个

## 四、Bug 详情

### Bug #1：商品价格可以为负数
- 接口：POST /api/products/
- 请求：{"name":"测试","price":"-50","stock":10,"category":1}
- 实际：201 Created，商品创建成功
- 预期：400 Bad Request，提示价格不能为负数
- 严重程度：中

### Bug #2：商品库存可以为负数
- 接口：POST /api/products/
- 请求：{"name":"测试","price":"99","stock":-5,"category":1}
- 实际：201 Created，库存为 -5 的商品创建成功
- 预期：400 Bad Request，提示库存不能为负数
- 严重程度：中

### Bug #3：购物车数量可以为 0
- 接口：POST /api/cart/
- 请求：{"product":1, "quantity":0}
- 实际：201 Created，购物车成功添加了数量为 0 的商品
- 预期：400 Bad Request，提示数量必须大于 0
- 严重程度：低

### Bug #4：购物车数量为负数导致 500 崩溃
- 接口：POST /api/cart/
- 请求：{"product":1, "quantity":-2}
- 实际：500 Internal Server Error
- 预期：400 Bad Request，提示数量不能为负数
- 严重程度：高（服务器崩溃）


## 五、数据一致性验证

### 5.1 创建商品后验证
- 在 Postman 创建商品（name="一致性测试商品1", price=88.88, stock=30, category_id=1）
- 数据库查询确认：

```sql
SELECT id, name, price, stock, category_id FROM api_product WHERE name='一致性测试商品1'; => 19|一致性测试商品1|88.88|30|1
```

- **结论** ✅ Postman 返回与数据库写入一致

### 5.2 下单后验证

- 购物车加购：product_id=19, quantity=3
- 下单成功后查询：

```sql
SELECT id, user_id, total, status FROM api_order WHERE id=4; => 4|1|266.64|paid

SELECT id, order_id, product_id, quantity, price FROM api_orderitem WHERE order_id=4; => 4|4|19|3|88.88
```

- 金额验算：88.88 × 3 = 266.64 ✅
- 订单明细记录了下单时的单价 88.88 ✅

### 5.3 下单后购物车清空验证

```sql
SELECT * FROM api_cart; => (空)
```

- **结论** ✅ 下单后购物车自动清空

---

## 六、测试总结

| 测试层次 | 场景数 | 通过 | 发现 Bug |
|---------|-------|------|---------|
| 第一层：功能覆盖 | 18 | 18 ✅ | 0 |
| 第二层：参数验证 | 15 | 11 ✅ | **4** |
| 第三层：数据一致性 | 3 | 3 ✅ | 0 |
| **总计** | **36** | **32** | **4** |

### Bug 严重程度分布

| 严重程度 | 数量 | Bug 编号 |
|---------|------|---------|
| 低 | 1 | #3 |
| 中 | 2 | #1, #2 |
| 高 | 1 | #4 |

### 项目整体评价

- **功能实现完善**，18 个核心接口全部可用
- **基础防护到位**，认证校验、必填字段校验、关联 ID 校验均正常工作
- **字段级校验缺失**，价格、库存、数量等数字字段缺少下限校验，尤其是数量为负数时直接导致 500 崩溃，属于较严重的缺陷
- **数据一致性优秀**，下单后金额计算、清空购物车等关键业务流程数据完全正确
