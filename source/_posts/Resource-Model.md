---
title: Resource Model
date: 2024-02-27 10:21:15
tags: [Resource, Model]
---
Magento的Resource Model封装了数据库的增删改查操作，有两种：db resource和collection resource。二者都定义了load方法，分别用于加载单个model和多个model；二者都定义了save方法，分别用于保存单个model和多个model。

### 增删改
对于增删改，依赖`\Magento\Framework\DB\Adapter\AdapterInterface`，两者都定义getConnection方法用于获取这个对象。（一般增删改操作都封装在db resource中，即只对单个model做封装。因为增删改操作大多会一起封装其他业务逻辑，批量操作不容易封装）。
```php
class AbstractDb extends AbstractResource {
  /**
   * Get connection
   *
   * @return \Magento\Framework\DB\Adapter\AdapterInterface|false
   */
  public function getConnection()
  {
    $fullResourceName = ($this->connectionName ? $this->connectionName : ResourceConnection::DEFAULT_CONNECTION);
    return $this->_resources->getConnection($fullResourceName);
  }
}

class AbstractCollection extends AbstractDb {
  /**
   * Retrieve connection object
   *
   * @return AdapterInterface
   */
  public function getConnection()
  {
    return $this->_conn;
  }
}
```
### 查
对于查，依赖`\Magento\Framework\DB\Select`，二者分别定义了_getLoadSelect和getSelect用于获取这个对象：
```php
class AbstractDb extends AbstractResource {
  protected function _getLoadSelect($field, $value, $object)
    {
        $field = $this->getConnection()->quoteIdentifier(sprintf('%s.%s', $this->getMainTable(), $field));
        $select = $this->getConnection()->select()->from($this->getMainTable())->where($field . '=?', $value);
        return $select;
    }
}

class AbstractCollection extends AbstractDb {
  protected $_select;
  public function getSelect()
  {
    if ($this->_select && $this->_fieldsToSelectChanged) {
      $this->_fieldsToSelectChanged = false;
      $this->_initSelectFields();
    }
    return parent::getSelect();
  }
}
```
db resource和Select是使用关系，定义的_getLoadSelect是Select的工厂方法，其本身也没有任何变量对Select进行关联。因此db resource和查询的结果也是无关联的，完全解耦。如此，才能使其作为share的对象，放置在di容器中。

collection resource和select是关联关系，通过_select变量关联Select对象。因此这个对象和查询结果也是关联的，紧密耦合。当有这个对象的依赖时，需要依赖其factory生成新的collection。

Select可以使用自身的query，获得一个PDO_Statement对象，然后基于这个对象再来获取数据，但是`\Magento\Framework\DB\Adapter\AdapterInterface`封装了很多更加方便的方法直接获取数据。
```php
//Fetches the first column of the first row of the SQL result.
public function fetchOne($sql, $bind = []);

//Fetches the first column of all SQL result rows as an array.
//The first column in each row is used as the array key.
public function fetchCol($sql, $bind = []);

//Fetches all SQL result rows as an array of key-value pairs.
//The first column is the key, the second column is the value.
public function fetchPairs($sql, $bind = []);

//Fetches the first row of the SQL result.
//Uses the current fetchMode for the adapter.
public function fetchRow($sql, $bind = [], $fetchMode = null);

//Fetches all SQL result rows as a sequential array.
//Uses the current fetchMode for the adapter.
public function fetchAll($sql, $bind = [], $fetchMode = null);

//Fetches all SQL result rows as an associative array.
//The first column is the key, the entire row array is the value.
public function fetchAssoc($sql, $bind = []);
```