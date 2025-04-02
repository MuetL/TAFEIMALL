# demo

[toc]

## 系统概述

B2C电商平台，包含买家、卖家双端，支持商品浏览、购物车、订单创建、支付流程、订单管理等完整电商功能。

## 功能模块

### 买家端功能

用户管理
    注册/登录/找回密码

    个人信息管理

    收货地址管理（支持默认地址设置）

    账户安全设置

    支付方式管理（支持默认支付方式设置）

商品浏览
    多级分类浏览（三级分类结构）

    商品搜索（关键词、分类、价格区间）

    商品详情页（多图展示、规格、评价、销量显示）

    商品收藏

    浏览历史记录

购物流程
    购物车管理（增删改查、选择性结算）

    订单创建（默认地址自动填充、可选择修改）

    订单支付（默认支付方式自动选择、可修改）

    订单状态跟踪（待支付、待发货、运输中、待收货、已完成）

    订单评价

其他功能
    退换货申请

    订单投诉

    客服咨询

### 卖家端功能

店铺管理
    店铺信息设置

    店铺装修（简单模板）

商品管理
    商品发布/编辑/上下架

    多图上传管理（支持主图设置）

    商品分类管理

    商品库存管理

    销售数据统计

订单管理
    订单处理（发货、退款）

    订单状态更新

    销售数据分析

客服系统
    买家消息回复

    售后处理

## 数据库设计(MySQL)

### 核心表结构

#### 用户相关

sql

CREATE TABLE `user` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `username` VARCHAR(50) NOT NULL,
  `password` VARCHAR(100) NOT NULL,
  `phone` VARCHAR(20),
  `email` VARCHAR(50),
  `status` TINYINT NOT NULL DEFAULT 1,
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE `user_address` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `user_id` BIGINT NOT NULL,
  `receiver` VARCHAR(50) NOT NULL,
  `phone` VARCHAR(20) NOT NULL,
  `province` VARCHAR(50) NOT NULL,
  `city` VARCHAR(50) NOT NULL,
  `district` VARCHAR(50),
  `detail` VARCHAR(200) NOT NULL,
  `is_default` TINYINT(1) NOT NULL DEFAULT 0,
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE `user_payment_preference` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `user_id` BIGINT NOT NULL,
  `payment_type` VARCHAR(20) NOT NULL,
  `is_default` TINYINT(1) NOT NULL DEFAULT 0,
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

#### 商品相关

sql

CREATE TABLE `product` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `name` VARCHAR(100) NOT NULL,
  `category_id` BIGINT NOT NULL,
  `price` DECIMAL(10,2) NOT NULL,
  `stock` INT NOT NULL,
  `sales` INT NOT NULL DEFAULT 0,
  `status` TINYINT NOT NULL DEFAULT 1,
  `description` TEXT,
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE `product_image` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `product_id` BIGINT NOT NULL,
  `image_url` VARCHAR(255) NOT NULL,
  `sort` INT NOT NULL DEFAULT 0,
  `is_main` TINYINT(1) NOT NULL DEFAULT 0,
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE `category` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `name` VARCHAR(50) NOT NULL,
  `parent_id` BIGINT,
  `level` TINYINT NOT NULL,
  `sort` INT NOT NULL DEFAULT 0,
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

#### 订单相关

sql

CREATE TABLE `order` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `order_no` VARCHAR(50) NOT NULL,
  `user_id` BIGINT NOT NULL,
  `total_amount` DECIMAL(10,2) NOT NULL,
  `payment_amount` DECIMAL(10,2) NOT NULL,
  `status` VARCHAR(20) NOT NULL DEFAULT '待支付',
  `pay_time` DATETIME,
  `ship_time` DATETIME,
  `receive_time` DATETIME,
  `complete_time` DATETIME,
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE `order_item` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `order_id` BIGINT NOT NULL,
  `product_id` BIGINT NOT NULL,
  `quantity` INT NOT NULL,
  `price` DECIMAL(10,2) NOT NULL
);

#### 其他表

sql

CREATE TABLE `cart` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `user_id` BIGINT NOT NULL,
  `product_id` BIGINT NOT NULL,
  `quantity` INT NOT NULL,
  `selected` TINYINT(1) NOT NULL DEFAULT 1,
  `create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE `browse_history` (
  `id` BIGINT PRIMARY KEY AUTO_INCREMENT,
  `user_id` BIGINT NOT NULL,
  `product_id` BIGINT NOT NULL,
  `browse_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

## 接口设计(JSON/HTTP)

### 用户相关接口

用户注册
    POST /api/user/register

    请求参数：{username, password, phone, email}

    返回：{code, message, data: {userId}}

设置默认地址
    POST /api/address/setDefault

    参数：{addressId}

    返回：{code, message}

### 商品相关接口

获取分类树
    GET /api/category/tree

    返回：{code, message, data: [{id, name, children: [...]}]}

商品图片上传
    POST /api/seller/product/image/upload

    参数：multipart/form-data (productId, files[])

    返回：{code, message, data: {imageUrls}}

### 订单相关接口

创建选中商品订单
    POST /api/cart/checkout

    参数：{cartItemIds: [], addressId}

    返回：{code, message, data: {orderId}}

获取订单状态
    GET /api/order/{orderId}/status

    返回：{code, message, data: {status, timeline: [{status, time}]}}

### 其他接口

获取浏览历史
    GET /api/user/history

    参数：{page, size}

    返回：{code, message, data: {list, total}}

## 技术架构

前端技术栈
    Vue3 + TypeScript

    Vue Router

    Pinia状态管理

    Element Plus组件库

    Axios HTTP客户端

后端技术栈
    Spring Boot 2.7

    Spring Security

    MyBatis Plus

    JWT认证

    Redis缓存

    MySQL 8.0

部署架构
    Nginx反向代理

    Docker容器化

    Jenkins持续集成

## 关键业务逻辑

订单状态流转
java

// 订单状态机示例
public class OrderStateMachine {
    private static final Map<String, List<String>> transitions = Map.of(
        "待支付", List.of("待发货", "已取消"),
        "待发货", List.of("运输中"),
        "运输中", List.of("待收货"),
        "待收货", List.of("已完成"),
        "已完成", Collections.emptyList()
    );

    public static boolean canTransition(String current, String next) {
        return transitions.getOrDefault(current, Collections.emptyList())
                         .contains(next);
    }
}

购物车结算
javascript

// 前端购物车计算逻辑
const calculateSelected = () => {
  const selectedItems = cartItems.value.filter(item => item.selected);
  total.value = selectedItems.reduce((sum, item) => {
    return sum + (item.price * item.quantity);
  }, 0);
  selectedCount.value = selectedItems.length;
};

默认地址管理
java

// 设置默认地址服务
public void setDefaultAddress(Long userId, Long addressId) {
    // 1. 验证地址属于该用户
    UserAddress address = addressMapper.selectByIdAndUser(addressId, userId);
    if (address == null) throw new BusinessException("地址不存在");   

    // 2. 取消原有默认地址
    addressMapper.cancelAllDefault(userId);    
    
    // 3. 设置新默认地址
    addressMapper.setDefault(addressId);
}

## 扩展设计

性能优化
    商品分类缓存到Redis

    销量统计使用异步更新

    图片使用CDN加速

安全措施
    支付接口防重放攻击

    敏感操作日志记录

    定期安全审计

监控方案
    Prometheus + Grafana监控

    ELK日志系统

    业务指标监控（订单成功率等）

## 项目里程碑

基础框架搭建（2周）

    用户系统

    商品分类

    基础订单流程

核心功能开发（4周）

    购物车系统

    支付流程

    订单状态管理

    卖家后台

高级功能实现（2周）

    浏览历史

    商品评价

    数据统计

测试与优化（2周）

    性能测试

    安全测试

    UI优化

部署上线（1周）

    生产环境配置

    监控系统部署

    上线检查
