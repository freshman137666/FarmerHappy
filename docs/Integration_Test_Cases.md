# 集成测试用例集 (Integration Test Cases)

## 1. 简介
本项目采用分层集成测试策略，覆盖了从路由层、控制器层到业务服务层的逻辑。测试用例主要分为以下三类：
1. **路由集成测试 (Router Integration Tests)**: 验证 HTTP 请求是否能正确路由到对应的控制器方法，并正确解析请求参数。
2. **控制器集成测试 (Controller Integration Tests)**: 验证控制器层的参数校验、业务逻辑调用（Mock Service）及 HTTP 响应格式。
3. **服务层集成测试 (Service Layer Integration Tests)**: 验证业务逻辑与数据库交互（使用 H2 内存数据库模拟），确保数据持久化和业务规则正确。

## 2. 测试用例详细列表

### 2.1 服务层集成测试 (Service Layer)
**测试类**: `service.buyer.OrderServiceIT`
**描述**: 验证买家订单服务的核心业务逻辑，包括创建订单、支付、退款、确认收货等，使用 H2 内存数据库进行真实数据交互。

| ID | 测试方法名 | 测试描述 | 预期结果 |
|:---|:---|:---|:---|
| S-ORD-01 | `create_order_updates_stock_balance_and_sales` | 创建订单 | 库存减少，余额扣除，销量增加，订单状态为 shipped |
| S-ORD-02 | `apply_refund_restores_balance_stock_and_sales` | 申请退款 | 余额退回，库存恢复，销量回滚，订单状态为 refunded |
| S-ORD-03 | `confirm_receipt_moves_to_completed_and_pays_farmer` | 确认收货 | 订单状态变为 completed，农户收到货款 |
| S-ORD-04 | `get_order_detail_returns_expected_fields` | 查询订单详情 | 返回正确的订单信息（ID, 标题, 金额等） |
| S-ORD-05 | `get_order_list_filters_by_status_and_title` | 查询订单列表 | 根据状态和标题正确筛选订单 |
| S-ORD-06 | `refund_only_refund_throws_error_and_no_state_change` | 仅退款模式异常 | 已发货订单不支持仅退款，抛出异常，状态不变 |
| S-ORD-07 | `update_order_changes_buyer_address_and_remark` | 修改订单 | 成功修改买家地址和备注 |
| S-ORD-08 | `get_farmer_order_detail_and_list_for_shipped` | 农户查询订单 | 农户能查询到自己名下的订单详情和列表 |
| S-ORD-09 | `create_order_stock_insufficient_throws_error` | 库存不足下单 | 抛出“库存不足”异常 |
| S-ORD-10 | `create_order_insufficient_balance_throws_error` | 余额不足下单 | 抛出“余额不足”异常 |

**测试类**: `service.farmer.ProductServiceIT`
**描述**: 验证农户商品管理服务，包括商品的增删改查、上下架及批量操作。

| ID | 测试方法名 | 测试描述 | 预期结果 |
|:---|:---|:---|:---|
| S-PROD-01 | `create_product_success` | 创建商品 | 数据库插入商品记录，返回商品ID |
| S-PROD-02 | `update_product_success` | 更新商品 | 商品信息被更新，返回更新后的信息 |
| S-PROD-03 | `on_shelf_product` | 上架商品 | 商品状态变为 on_shelf |
| S-PROD-04 | `off_shelf_product` | 下架商品 | 商品状态变为 off_shelf |
| S-PROD-05 | `delete_product` | 删除商品 | 商品从数据库中标记删除或物理删除 |
| S-PROD-06 | `batch_action_products` | 批量操作 | 多个商品状态同时更新 |

### 2.2 控制器集成测试 (Controller Layer)
**测试类**: `controller.AuthControllerIT`
**描述**: 验证用户认证、注册及余额查询接口。

| ID | 测试方法名 | 测试描述 | 预期结果 |
|:---|:---|:---|:---|
| C-AUTH-01 | `register_success_returns_200` | 用户注册成功 | 返回 200 及用户 ID |
| C-AUTH-02 | `register_validation_error_returns_400_with_errors` | 注册参数校验失败 | 返回 400 及错误字段列表 |
| C-AUTH-03 | `register_duplicate_phone_returns_409` | 手机号重复注册 | 返回 409 冲突状态码 |
| C-AUTH-04 | `login_success_returns_200` | 用户登录成功 | 返回 200 及用户信息 |
| C-AUTH-05 | `login_security_error_returns_401` | 登录密码错误 | 返回 401 未授权 |

**测试类**: `controller.OrderControllerIT`
**描述**: 验证订单相关接口的参数处理和响应映射。

| ID | 测试方法名 | 测试描述 | 预期结果 |
|:---|:---|:---|:---|
| C-ORD-01 | `create_order_success` | 创建订单接口 | 返回 201 Created |
| C-ORD-02 | `create_order_validation_error` | 创建订单参数缺失 | 返回 400 Bad Request |

### 2.3 路由集成测试 (Router Layer)
**测试类**: `config.RouterConfigIT`
**描述**: 验证 HTTP 请求路由分发逻辑。

| ID | 测试方法名 | 测试描述 | 预期结果 |
|:---|:---|:---|:---|
| R-CONF-01 | `login_parsing_and_dispatch` | 登录路由 | 正确解析 JSON Body 并调用 AuthController.login |
| R-CONF-02 | `create_order_parsing_and_dispatch` | 下单路由 | 正确解析参数并调用 OrderController.createOrder |
| R-CONF-03 | `product_list_query_dispatch` | 商品列表路由 | 正确解析 Query 参数并分发 |
| R-CONF-04 | `batch_actions_parsing_and_dispatch` | 批量操作路由 | 正确解析复杂 JSON 对象并分发 |

## 3. 测试环境
- **测试框架**: JUnit 5
- **Mock 框架**: Mockito (用于隔离层间依赖)
- **数据库**: H2 Database (In-Memory, Mode=MySQL)
- **断言库**: AssertJ

## 4. 执行方式
所有集成测试均可通过 Maven Failsafe 插件自动执行：
```bash
mvn verify
```
或者单独运行：
```bash
mvn -Dit.test=*IT verify
```
