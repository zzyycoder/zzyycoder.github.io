---
title: Indexer
date: 2023-12-07 17:32:34
tags: indexer
---

### Why Indexer?

实体包含属性，通常属性值的获取直接读取数据库存储即可（比如商品的`name`、`sku`等），但是有些属性值获取却包含一定逻辑（例如商品的`final_price`，最终值要综合商品设置价格、用户组、税费设置等）。

这些逻辑在处理单个实体时没有什么问题，查询到实体的数据再做计算即可；但是当查询多个实体并且需要按照这些（包含计算逻辑的）属性作排序和筛选时，就变得麻烦了许多。生硬地采用联表查询，不仅使得sql语句杂乱难维护，查询效率也会很慢，因为通常要关联不止一张表。

`Magento`的` Indexer`就是为了解决这个问题。通过事先计算好这些属性值并存入一张表中，这样在需要查询时只需关联这一张表。本质上，这属于用空间换取时间的做法。

### Indexer Basic

`Magento` 的`Indexer`工作模式分两种：`Update on Save`和`Update by Schedule`。

`Update on Save`是指更新实体时再即时更新相关的`index`记录，`Update by Schedule`则是先把更新的实体`id`存入数据表，后续通过`cronjob`处理这些实体。

`Update by Schedule`将实体id存入数据表是由`Magento/Framework/Mview`实现，原理是使用`mysql`的`triggers`，当设置`Indexer`的工作模式为`Update by Schedule`时，会创建一张`change log`表，并根据`mview.xml`的配置创建`triggers`。

### Default Indexers

`Mangeto`自带的一些`indexer`介绍：

- Product Price

  Index数据存储在`catalog_product_index_price`中，包含`price`，`final_price`，`min_price`，`max_price`，`tier_price`

  当使用这个index数据时会先将eav attribute price移除，替换为使用index的price，这是为了保证数据一致。

  搜索和类目页面的商品列表支持根据价格排序和筛选，因此会用到index数据。

  ```php
  // Magento\Catalog\Model\Layer在获取product collection后，会prepareProductCollection
  public function getProductCollection()
  {
      if (isset($this->_productCollections[$this->getCurrentCategory()->getId()])) {
          $collection = $this->_productCollections[$this->getCurrentCategory()->getId()];
      } else {
          $collection = $this->collectionProvider->getCollection($this->getCurrentCategory());
          $this->prepareProductCollection($collection);
          $this->_productCollections[$this->getCurrentCategory()->getId()] = $collection;
      }
  
      return $collection;
  }
  
  // Magento\Catalog\Model\Layer\Category\CollectionFilter会addMininalPrice、addFinalPrice
  public function filter(
      $collection,
      \Magento\Catalog\Model\Category $category
  ) {
      $collection
          ->addAttributeToSelect($this->catalogConfig->getProductAttributes())
          ->addMinimalPrice()
          ->addFinalPrice()
          ->addTaxPercents()
          ->addUrlRewrite($category->getId())
          ->setVisibility($this->productVisibility->getVisibleInCatalogIds());
  }
  
  // Magento\Catalog\Model\ResourceModel\Product\Collection addPriceData时关联查询index表
  public function addPriceData($customerGroupId = null, $websiteId = null)
  {
      $this->_productLimitationFilters->setUsePriceIndex(true);
  
      if (!isset($this->_productLimitationFilters['customer_group_id']) && $customerGroupId === null) {
          $customerGroupId = $this->_customerSession->getCustomerGroupId();
      }
      if (!isset($this->_productLimitationFilters['website_id']) && $websiteId === null) {
          $websiteId = $this->_storeManager->getStore($this->getStoreId())->getWebsiteId();
      }
  
      if ($customerGroupId !== null) {
          $this->_productLimitationFilters['customer_group_id'] = $customerGroupId;
      }
      if ($websiteId !== null) {
          $this->_productLimitationFilters['website_id'] = $websiteId;
      }
  
      $this->_applyProductLimitations();
  
      return $this;
  }
  ```

  
