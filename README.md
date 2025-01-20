# 豆包商城 API 文档

## 基本信息

- 基础URL：`http://localhost:8888`
- 所有POST请求的Content-Type应该设置为：`application/json`
- 需要认证的接口应在请求头中添加：`Authorization: Bearer {token}`

## 配置说明

### 环境要求
- Go 1.16+
- MySQL 5.7+
- Redis (可选，用于缓存)

### 配置项
项目运行需要以下配置：

1. 数据库配置
```env
DB_HOST=localhost
DB_PORT=3306
DB_NAME=qaqmall
DB_USER=root
DB_PASSWORD=123456
```

2. 服务器配置
```env
SERVER_PORT=8888
JWT_SECRET=your-secret-key
```

3. OpenAI配置（用于AI助手功能）
```env
OPENAI_API_KEY=sk-xxx
OPENAI_API_URL=https://api.openai.com/v1/chat/completions
```

### 快速开始

1. 克隆项目
```bash
git clone [项目地址]
cd qaqmall
```

2. 初始化数据库
```bash
mysql -u root -p < scripts/init_database.sql
```

3. 配置环境变量
```bash
cp .env.example .env
# 编辑 .env 文件，填入实际的配置值
```

4. 运行项目
```bash
go run main.go
```

## API 接口

### 1. 用户管理

#### 1.1 用户注册

- 请求方式：`POST /register`
- 请求参数：
```json
{
    "username": "test_user_123",
    "password": "test123456",
    "email": "test@example.com",
    "phone": "13800138000"
}
```
- 响应示例：
```json
{
    "code": 200,
    "data": {
        "role": "user",
        "user_id": 8,
        "username": "test_user_123"
    },
    "message": "注册成功"
}
```

### 1.2 用户登录

- 请求方式：`POST /login`
- 请求参数：
```json
{
    "username": "test_user_123",
    "password": "test123456"
}
```
- 响应示例：
```json
{
    "code": 200,
    "data": {
        "role": "user",
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "user_id": 8,
        "username": "test_user_123"
    },
    "message": "登录成功"
}
```

### 1.3 用户登出

- 请求方式：`POST /logout`
- 请求头：需要用户token
- 响应示例：
```json
{
    "code": 200,
    "message": "登出成功"
}
```

### 1.4 获取用户信息

- 请求方式：`GET /user/info`
- 请求头：需要用户token
- 响应示例：
```json
{
    "id": 8,
    "username": "test_user_123",
    "role": "user",
    "email": "test@example.com",
    "phone": "13800138000"
}
```

### 1.5 更新用户信息

- 请求方式：`PUT /user/info`
- 请求头：需要用户token
- 请求参数：
```json
{
    "email": "new_email@example.com",
    "phone": "13800138001"
}
```
- 响应示例：
```json
{
    "message": "更新成功",
    "user": {
        "id": 8,
        "username": "test_user_123",
        "role": "user",
        "email": "new_email@example.com",
        "phone": "13800138001"
    }
}
```

### 1.6 删除用户账号

- 请求方式：`DELETE /user`
- 请求头：需要用户token
- 响应示例：
```json
{
    "code": 200,
    "message": "用户已删除"
}
```

## 2. 商品管理

### 2.1 创建商品（需要管理员权限）

- 请求方式：`POST /admin/products`
- 请求头：需要管理员token
- 请求参数：
```json
{
    "name": "iPhone 15",
    "description": "最新款苹果手机",
    "price": 5999.99,
    "stock": 100,
    "image_url": "http://example.com/iphone15.jpg",
    "is_on_sale": true,
    "categories": [1, 2]  // 商品分类ID列表
}
```
- 响应示例：
```json
{
    "id": 25,
    "created_at": "2025-01-18T11:48:51.15+08:00",
    "updated_at": "2025-01-18T11:48:51.15+08:00",
    "name": "iPhone 15",
    "description": "最新款苹果手机",
    "price": 5999.99,
    "stock": 100,
    "image_url": "http://example.com/iphone15.jpg",
    "is_on_sale": true,
    "categories": [
        {
            "id": 1,
            "name": "手机"
        },
        {
            "id": 2,
            "name": "数码产品"
        }
    ]
}
```

### 2.2 修改商品（需要管理员权限）

- 请求方式：`PUT /admin/products/{id}`
- 请求头：需要管理员token
- 请求参数：与创建商品相同
- 响应示例：与创建商品响应格式相同

### 2.3 删除商品（需要管理员权限）

- 请求方式：`DELETE /admin/products/{id}`
- 请求头：需要管理员token
- 响应示例：
```json
{
    "message": "商品已删除"
}
```

### 2.4 获取商品列表

- 请求方式：`GET /products`
- 查询参数：
  - page: 页码（从1开始）
  - pageSize: 每页数量（默认10）
- 响应示例：
```json
{
    "total": 25,
    "items": [
        {
            "id": 1,
            "created_at": "2025-01-18T11:10:19.011+08:00",
            "updated_at": "2025-01-18T11:10:19.011+08:00",
            "name": "测试手机1",
            "description": "这是一款测试手机",
            "price": 1999.99,
            "stock": 100,
            "image_url": "http://example.com/phone1.jpg",
            "is_on_sale": true
        }
    ]
}
```

## 3. 购物车管理

### 3.1 添加商品到购物车

- 请求方式：`POST /cart/items`
- 请求头：需要用户token
- 请求参数：
```json
{
    "product_id": 1,
    "quantity": 1
}
```
- 响应示例：
```json
{
    "id": 1,
    "user_id": 8,
    "product_id": 1,
    "quantity": 1,
    "price": 1999.99,
    "product_name": "测试手机1",
    "product_image": "http://example.com/phone1.jpg",
    "selected": true
}
```

### 3.2 更新购物车商品数量

- 请求方式：`PUT /cart/items/{id}`
- 请求头：需要用户token
- 请求参数：
```json
{
    "quantity": 2,
    "selected": true
}
```
- 响应示例：与添加商品到购物车响应格式相同

### 3.3 删除购物车商品

- 请求方式：`DELETE /cart/items/{id}`
- 请求头：需要用户token
- 响应示例：
```json
{
    "code": 200,
    "message": "商品已从购物车移除"
}
```

### 3.4 清空购物车

- 请求方式：`DELETE /cart/items`
- 请求头：需要用户token
- 响应示例：
```json
{
    "code": 200,
    "message": "购物车已清空"
}
```

### 3.5 获取购物车列表

- 请求方式：`GET /cart/items`
- 请求头：需要用户token
- 响应示例：
```json
{
    "items": [
        {
            "id": 1,
            "user_id": 8,
            "product_id": 1,
            "quantity": 2,
            "price": 1999.99,
            "product_name": "测试手机1",
            "product_image": "http://example.com/phone1.jpg",
            "selected": true
        }
    ]
}
```

## 4. 地址管理

### 4.1 添加收货地址

- 请求方式：`POST /addresses`
- 请求头：需要用户token
- 请求参数：
```json
{
    "name": "张三",
    "phone": "13800138001",
    "province": "广东省",
    "city": "深圳市",
    "district": "南山区",
    "street": "科技园路",
    "detail": "1号楼101室",
    "postal_code": "518000",
    "tag": "家",
    "is_default": true
}
```
- 响应示例：
```json
{
    "id": 3,
    "user_id": 8,
    "name": "张三",
    "phone": "13800138001",
    "province": "广东省",
    "city": "深圳市",
    "district": "南山区",
    "street": "科技园路",
    "detail": "1号楼101室",
    "postal_code": "518000",
    "tag": "家",
    "is_default": true
}
```

### 4.2 修改收货地址

- 请求方式：`PUT /addresses/{id}`
- 请求头：需要用户token
- 请求参数：与添加地址相同
- 响应示例：与添加地址响应格式相同

### 4.3 删除收货地址

- 请求方式：`DELETE /addresses/{id}`
- 请求头：需要用户token
- 响应示例：
```json
{
    "code": 200,
    "message": "地址已删除"
}
```

### 4.4 获取地址列表

- 请求方式：`GET /addresses`
- 请求头：需要用户token
- 响应示例：
```json
{
    "items": [
        {
            "id": 3,
            "user_id": 8,
            "name": "张三",
            "phone": "13800138001",
            "province": "广东省",
            "city": "深圳市",
            "district": "南山区",
            "street": "科技园路",
            "detail": "1号楼101室",
            "postal_code": "518000",
            "tag": "家",
            "is_default": true
        }
    ]
}
```

## 5. 订单管理

### 5.1 创建订单

- 请求方式：`POST /orders`
- 请求头：需要用户token
- 请求参数：
```json
{
    "address_id": 3,
    "items": [
        {
            "product_id": 1,
            "quantity": 2
        }
    ],
    "remark": "测试订单"
}
```
- 响应示例：
```json
{
    "code": 200,
    "message": "创建订单成功",
    "data": {
        "order_id": 1,
        "order_number": "202501181858525",
        "total_amount": 3999.98,
        "expired_at": "2025-01-18T19:28:52+08:00"
    }
}
```

### 5.2 取消订单

- 请求方式：`POST /orders/{id}/cancel`
- 请求头：需要用户token
- 响应示例：
```json
{
    "code": 200,
    "message": "取消订单成功"
}
```

### 5.3 获取订单详情

- 请求方式：`GET /orders/{id}`
- 请求头：需要用户token
- 响应示例：
```json
{
    "id": 1,
    "order_number": "202501181858525",
    "user_id": 8,
    "status": "pending",
    "total_amount": 3999.98,
    "address_id": 3,
    "remark": "测试订单",
    "expired_at": "2025-01-18T19:28:52+08:00",
    "created_at": "2025-01-18T18:58:52+08:00",
    "updated_at": "2025-01-18T18:58:52+08:00",
    "address": {
        "id": 3,
        "user_id": 8,
        "name": "张三",
        "phone": "13800138001",
        "province": "广东省",
        "city": "深圳市",
        "district": "南山区",
        "street": "科技园路",
        "detail": "1号楼101室",
        "postal_code": "518000",
        "tag": "家",
        "is_default": true
    },
    "items": [
        {
            "id": 1,
            "order_id": 1,
            "product_id": 1,
            "product_name": "测试手机1",
            "product_image": "http://example.com/phone1.jpg",
            "price": 1999.99,
            "quantity": 2,
            "product": {
                "id": 1,
                "name": "测试手机1",
                "description": "这是一款测试手机",
                "price": 1999.99,
                "stock": 98,
                "image_url": "http://example.com/phone1.jpg",
                "is_on_sale": true
            }
        }
    ]
}
```

### 5.4 获取订单列表

- 请求方式：`GET /orders`
- 请求头：需要用户token
- 响应示例：
```json
{
    "items": [
        {
            // 订单详情，格式同上
        }
    ]
}
```

### 5.5 修改订单

- 请求方式：`PUT /orders/{id}`
- 请求头：需要用户token
- 请求参数：
```json
{
    "address_id": 4,
    "remark": "新的备注"
}
```
- 响应示例：
```json
{
    "code": 200,
    "message": "更新订单成功",
    "data": {
        // 订单详情，格式同上
    }
}
```

## 6. 支付管理

### 6.1 创建支付

- 请求方式：`POST /payments`
- 请求头：需要用户token
- 请求参数：
```json
{
    "order_id": 1,
    "payment_method": "alipay"
}
```
- 响应示例：
```json
{
    "code": 200,
    "message": "创建支付记录成功",
    "data": {
        "payment_id": 1,
        "payment_number": "PAY202501181858525",
        "amount": 3999.98,
        "expired_at": "2025-01-18T19:28:52+08:00"
    }
}
```

### 6.2 获取支付详情

- 请求方式：`GET /payments/{id}`
- 请求头：需要用户token
- 响应示例：
```json
{
    "id": 1,
    "payment_number": "PAY202501181858525",
    "order_id": 1,
    "user_id": 8,
    "amount": 3999.98,
    "payment_method": "alipay",
    "status": "pending",
    "created_at": "2025-01-18T18:58:52+08:00",
    "updated_at": "2025-01-18T18:58:52+08:00"
}
```

### 6.3 支付回调接口

- 请求方式：`POST /payments/callback`
- 请求参数：根据支付渠道的回调格式
- 响应示例：
```json
{
    "code": 200,
    "message": "支付回调处理成功"
}
```

## 6. AI 智能查询

### 6.1 统一查询接口

- 请求方式：`POST /ai/query`
- 请求头：需要用户token
- 请求参数：
```json
{
    "query": "我的购物车里有什么？"
}
```
- 响应示例：
```json
{
    "answer": "您的购物车中有：iPhone 15（数量：1，单价：5999.99元）和 MacBook Pro（数量：1，单价：14999.99元）。"
}
```

支持的查询类型：
1. 购物车查询：例如"我的购物车里有什么"、"购物车总价是多少"
2. 商品查询：例如"有什么热销商品"、"最近上架了什么新品"
3. 订单查询：例如"我的最近订单状态"、"我有什么待付款的订单"
4. 综合查询：例如"帮我推荐一些商品"、"有什么优惠活动"

注意事项：
1. 查询结果会根据用户的实际数据动态生成
2. AI会根据上下文提供个性化的回答
3. 如果查询的信息不在系统范围内，AI会告知用户

## 注意事项

1. 所有需要认证的接口必须在请求头中携带有效的token
2. 管理员相关接口需要使用管理员账号获取的token
3. 商品管理相关接口中，部分功能仅管理员可用
4. 地址管理和购物车接口仅对已登录用户开放
5. 订单创建后30分钟内未支付将自动取消
6. 订单取消后会自动恢复商品库存 
5. 分页接口默认每页显示10条数据 