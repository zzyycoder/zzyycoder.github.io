---
title: Pricing
date: 2023-12-05 16:51:21
tags: Product Price
---
### 哪些元素包含价格

1. 商品，包括商品本身以及设置的定制项
2. 购物车项/心愿单项
3. 订单项
4. 配送

其中订单项和购物车/心愿单项的不同之处在于订单项价格是确定的，购物车/心愿单项的价格则是不确定的，需要随着后台修改变动。

### 影响商品价格的因素

1. 产品设置的`price`、`special_price`和`tier_price`
2. 数量和`custom options`对价格的影响
3. 汇率换算
4. catalog促销
5. 税费等价格调整

### Price Info

{% asset_img Pricing.png %}

`Product`有`PriceInfo`的引用，`PriceInfo`通过`Collection`也有`Product`的引用。二者相互引用，类似**状态模式**。

### Price

`Collection`通过`di`配置价格类型：

```xml
<virtualType name="Magento\Catalog\Pricing\Price\Collection" type="Magento\Framework\Pricing\Price\Collection">
    <arguments>
        <argument name="pool" xsi:type="object">Magento\Catalog\Pricing\Price\Pool</argument>
    </arguments>
</virtualType>
<virtualType name="Magento\Catalog\Pricing\Price\Pool" type="Magento\Framework\Pricing\Price\Pool">
    <arguments>
        <argument name="prices" xsi:type="array">
            <item name="regular_price" xsi:type="string">Magento\Catalog\Pricing\Price\RegularPrice</item>
            <item name="final_price" xsi:type="string">Magento\Catalog\Pricing\Price\FinalPrice</item>
            <item name="tier_price" xsi:type="string">Magento\Catalog\Pricing\Price\TierPrice</item>
            <item name="special_price" xsi:type="string">Magento\Catalog\Pricing\Price\SpecialPrice</item>
            <item name="base_price" xsi:type="string">Magento\Catalog\Pricing\Price\BasePrice</item>
            <item name="custom_option_price" xsi:type="string">Magento\Catalog\Pricing\Price\CustomOptionPrice</item>
            <item name="configured_price" xsi:type="string">Magento\Catalog\Pricing\Price\ConfiguredPrice</item>
            <item name="configured_regular_price" xsi:type="string">Magento\Catalog\Pricing\Price\ConfiguredRegularPrice</item>
        </argument>
    </arguments>
</virtualType>
```

1. regular price/tier price/special price
2. base price，根据上面四个价格计算，取最低的那个，主要用于辅助final price/custom option price的计算
3. final price，当时simple product时，等于base price；当是configurable product时，取选中simple product的final price
4. custom option price，根据base price计算商品custom options的price
5. configured (final) price，继承自final price，计算了custom options对价格影响
6. configured regular price，继承自regular price，计算了custom options对价格的影响

注：

1. storefront常展示regular price和final price
2. 汇率换算在计算regular price/tier price/special price时应用，间接影响了base price
3. catalog促销价影响了base price
4. 税费等调整(adjustments)在所有的价格类型计算都涉及
5. `configured price`和`configured regular price`应用于`wishlist item`和`quote item`

### Price Model

除了`Price Info`可获取商品价格，`Product`也封装了一些方法获取价格：

```php
// Magento\Catalog\Model
function getPrice();
function getSpecialPrice();
function getSpecialPrice();
function getTierPrice($qty = null);
function getFinalPrice($qty = null);
```

这些方法的价格算法和`Price Info`基本类似，不同点在于：

1. 未做汇率换算
2. final price未考虑catalog促销、税费等价格调整

即最开始列的5条只做了1和2，3～5未处理。
