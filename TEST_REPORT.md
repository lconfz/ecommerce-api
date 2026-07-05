# 电商 API 测试报告

## 一、测试概述
- 项目：ecommerce-api
- 测试时间：2026-07-05
- 测试工具：Postman、SQLite3
- 测试人员：lconf

## 二、功能覆盖测试
18 个接口全部跑通 ✅

## 三、参数验证测试
- 初始测试：6 项通过，4 个 Bug
- 修复后重新验证：全部 15 项通过 ✅

## 四、Bug 详情

### Bug #1：商品价格可以为负数
- 接口：POST /api/products/
- 请求：{"name":"测试","price":"-50","stock":10,"category":1}
- 实际：201 Created，商品创建成功
- 预期：400 Bad Request，提示价格不能为负数
- 严重程度：中
- **状态：✅ 已修复**（在 price 字段添加 MinValueValidator(0)）

### Bug #2：商品库存可以为负数
- 接口：POST /api/products/
- 请求：{"name":"测试","price":"99","stock":-5,"category":1}
- 实际：201 Created，库存为 -5 的商品创建成功
- 预期：400 Bad Request，提示库存不能为负数
- 严重程度：中
- **状态：✅ 已修复**（在 stock 字段添加 MinValueValidator(0)）

### Bug #3：购物车数量可以为 0
- 接口：POST /api/cart/
- 请求：{"product":1, "quantity":0}
- 实际：201 Created，购物车成功添加了数量为 0 的商品
- 预期：400 Bad Request，提示数量必须大于 0
- 严重程度：低
- **状态：✅ 已修复**（在 quantity 字段添加 MinValueValidator(1)）

### Bug #4：购物车数量为负数导致 500 崩溃
- 接口：POST /api/cart/
- 请求：{"product":1, "quantity":-2}
- 实际：500 Internal Server Error
- 预期：400 Bad Request，提示数量不能为负数
- 严重程度：高（服务器崩溃）
- **状态：✅ 已修复**（在 quantity 字段添加 MinValueValidator(1)，防止负数传入数据库）

## 五、数据一致性验证

### 5.1 创建商品后验证
- 在 Postman 创建商品（name="一致性测试商品1", price=88.88, stock=30, category_id=1）
- 数据库查询确认：
  SELECT id, name, price, stock, category_id FROM api_product WHERE name='一致性测试商品1'; => 19|一致性测试商品1|88.88|30|1
- **结论** ✅ Postman 返回与数据库写入一致

### 5.2 下单后验证
- 购物车加购：product_id=19, quantity=3
- 下单成功后查询：
  SELECT id, user_id, total, status FROM api_order WHERE id=4; => 4|1|266.64|paid
  SELECT id, order_id, product_id, quantity, price FROM api_orderitem WHERE order_id=4; => 4|4|19|3|88.88
- 金额验算：88.88 × 3 = 266.64 ✅
- 订单明细记录了下单时的单价 88.88 ✅

### 5.3 下单后购物车清空验证
  SELECT * FROM api_cart; => (空)
- **结论** ✅ 下单后购物车自动清空

## 六、测试总结

| 测试层次 | 场景数 | 通过 | 发现 Bug |
|---------|-------|------|---------|
| 第一层：功能覆盖 | 18 | 18 ✅ | 0 |
| 第二层：参数验证 | 15 | 15 ✅ | 4（全部已修复）|
| 第三层：数据一致性 | 3 | 3 ✅ | 0 |
| **总计** | **36** | **36** | **4（全部已修复）** |

## 七、Bug 修复验证

全部 4 个 Bug 已通过修改 `api/models.py` 修复，修复后重新执行了异常场景测试：

| Bug | 修复方式 | 修复后验证 |
|-----|---------|-----------|
| #1 价格负数 | price 加 MinValueValidator(0) | POST price=-50 → 400 ✅ |
| #2 库存负数 | stock 加 MinValueValidator(0) | POST stock=-5 → 400 ✅ |
| #3 数量为 0 | quantity 加 MinValueValidator(1) | POST quantity=0 → 400 ✅ |
| #4 数量负数导致 500 | quantity 加 MinValueValidator(1) | POST quantity=-2 → 400 ✅ |

**结论：所有 4 个 Bug 已关闭，全部 15 项参数验证测试通过 ✅**
