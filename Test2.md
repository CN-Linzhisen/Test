# 商品溯源

## 代码注释

> TraceabilityFactory.sol 

```solidity
pragma solidity ^0.4.25;
pragma experimental ABIEncoderV2;

import "./Traceability.sol";

contract TraceabilityFactory{
    // 商品溯源信息
    struct GoodsTrace{
        Traceability trace; // 溯源信息
        bool valid; // 是否存在
    }
    // 商品类别对应的商品
    mapping(bytes32=>GoodsTrace) private _goodsCategory; 
	// 新增可溯源商品组事件
    event newTraceEvent(bytes32 goodsGroup);
	
    //Create traceability commodity category
    /*
        描述：创建可溯源商品类别
        参数:
            goodsGroup: 商品组
        返回值：
            Traceability: 溯源信息
    */
    function createTraceability(bytes32  goodsGroup) public returns(Traceability) {
        // 判断商标（商品组）是否存在
        require(!_goodsCategory[goodsGroup].valid,"The trademark already exists" );
        // 初始化商品组
        Traceability category = new Traceability(goodsGroup);
        _goodsCategory[goodsGroup].valid = true;
        _goodsCategory[goodsGroup].trace = category;
        emit newTraceEvent(goodsGroup);
        return category;
    }

    /*
        描述：获取可溯源商品类别信息
        参数:
            goodsGroup: 商品组
        返回值：
            Traceability: 溯源信息
    */
    function getTraceability(bytes32  goodsGroup) private view returns(Traceability) {
        // 判断商标（商品组）是否存在
        require(_goodsCategory[goodsGroup].valid,"The trademark has not exists" );
        return _goodsCategory[goodsGroup].trace;
    }

    //Create traceability products    
    /*
        描述：创建可溯源商品
        参数:
            goodsGroup: 商品组
            goodsId: 商品Id
        返回值：
            Goods: 商品信息
    */
    function createTraceGoods(bytes32  goodsGroup, uint64 goodsId) public returns(Goods) {
        // 获取可溯源商品类别信息
        Traceability category = getTraceability(goodsGroup);
        return category.createGoods(goodsId);
    }
    
    //Change product status
    /*
        描述：修改商品状态
        参数:
            goodsGroup: 商品组
            goodsId: 商品ID
            goodsStatus: 商品状态
            remark: 商品备注
        返回值：
            Traceability: 溯源信息
    */
    function changeTraceGoods(bytes32  goodsGroup, uint64 goodsId, int16 goodsStatus, string memory remark) public {
        // 获取可溯源商品类别信息
        Traceability category = getTraceability(goodsGroup);
        // 修改商品状态
        category.changeGoodsStatus(goodsId, goodsStatus, remark);
    }
    
    //Query the current status of goods
    /*
        描述：查询商品状态
        参数:
            goodsGroup: 商品组
            goodsId: 商品ID
        返回值：
            int16类型: 商品状态
    */
    function getStatus(bytes32 goodsGroup, uint64 goodsId) public view returns(int16) {
        // 获取可溯源商品类别信息
        Traceability category = getTraceability(goodsGroup);
        return category.getStatus(goodsId);
    }
    
    //The whole process of querying goods
    /*
        描述：查询商品的整个过程
        参数:
            goodsGroup: 商品组
            goodsId: 商品ID
        返回值：
            Goods.TraceData[]: 商品追踪数据
    */
    function getTraceInfo(bytes32 goodsGroup, uint64 goodsId) public view returns(Goods.TraceData[]) {
        // 获取可溯源商品类别信息
        Traceability category = getTraceability(goodsGroup);
        return category.getGoods(goodsId).getTraceInfo();
    }
    
    /*
        描述：根据商品组名生成bytes32
        参数:
            name: 商品组名
        返回值：
            bytes32类型: 商品组名对应的bytes32
    */
    function getGoodsGroup(string memory name) public pure returns (bytes32) {
        return keccak256(abi.encode(name));
    }
}
```

> Traceability.sol

```solidity
pragma solidity ^0.4.25;

import "./Goods.sol";

contract Traceability{
    // 商品信息
    struct GoodsData{
        Goods traceGoods; // 溯源商品信息
        bool valid; // 是否存在
    }
    // 商品类别
    bytes32 _category; 
    // 商品ID对应的商品信息
    mapping(uint64=>GoodsData) private _goods;
    constructor(bytes32  goodsTp) public {
        _category = goodsTp;
    }
    // 新增商品事件
    event newGoodsEvent(uint64 goodsId);
    
    /*
        描述：创建商品
        参数：
            goodsId: 商品ID
        返回值：
            Goods: 商品信息
    */
    function createGoods(uint64 goodsId) public returns(Goods){
        // 判断商品ID是否已存在
        require(!_goods[goodsId].valid, "is really exists");
        _goods[goodsId].valid = true;
        // 初始化商品信息
        Goods traceGoods = new Goods(goodsId);
        emit newGoodsEvent(goodsId);
        _goods[goodsId].traceGoods = traceGoods;
        return traceGoods;
    }
    
    /*
        描述：修改商品状态
        参数：
            goodsId: 商品ID
            goodsStatus: 商品状态
            remark: 商品摘要数据
    */
    function changeGoodsStatus(uint64 goodsId, int16 goodsStatus, string memory remark) public{
        // 判断商品是否存在
        require(_goods[goodsId].valid, "not exists");
        // 修改商品状态
         _goods[goodsId].traceGoods.changeStatus(goodsStatus, remark);
    }
      
    /*
        描述：获取商品状态
        参数：
            goodsId: 商品ID
        返回值：
            int64类型: 商品状态
    */
     function getStatus(uint64 goodsId)public view returns(int16){
         // 判断商品是否存在
         require(_goods[goodsId].valid, "not exists");
         return _goods[goodsId].traceGoods.getStatus();
    }

    /*
        描述：获取商品信息
        参数：
            goodsId: 商品ID
        返回值：
            Goods: 商品信息
    */
     function getGoods(uint64 goodsId)public view returns(Goods){
         // 判断商品是否存在
         require(_goods[goodsId].valid, "not exists");
         return _goods[goodsId].traceGoods;
    }
}
```

> Goods.sol

```solidity
pragma solidity ^0.4.25;
pragma experimental ABIEncoderV2;

contract Goods{
    struct TraceData{
        address addr;     //Operator address 操作者地址
        int16 status;     //goods status 商品状态
        uint timestamp;   //Operator time 操作时间
        string remark;    //Digested Data 摘要数据
    }
    uint64 _goodsId; // 商品ID
    int16 _status;   // 商品状态
    TraceData[] _traceData; // 商品溯源过程信息
    // 新增商品状态事件
    event newStatus( address addr, int16 status, uint timestamp, string remark);
    
    constructor(uint64 goodsId) public{
        _goodsId = goodsId;
        // 添加商品的溯源信息
        _traceData.push(TraceData({addr:tx.origin, status:0, timestamp:now, remark:"create"}));
        emit newStatus(tx.origin, 0, now, "create");
    }

    /*
        描述：修改商品状态
        参数：
            goodsStatus: 商品状态
        返回值：
            remark: 商品摘要数据
    */
    function changeStatus(int16 goodsStatus, string memory remark) public {
        _status = goodsStatus;
        // 添加商品的溯源信息（操作者地址、商品状态、当前时间戳、摘要信息）
         _traceData.push(TraceData({addr:tx.origin, status:goodsStatus, timestamp:now, remark:remark}));
          emit newStatus(tx.origin, goodsStatus, now, remark);
    }
      
     /*
        描述：获取商品状态
        返回值：
            int16类型: 商品状态
    */  
    function getStatus()public view returns(int16){
        return _status;
    }
    
     /*
        描述：获取商品溯源过程信息
        返回值：
            _data: 商品溯源过程信息
    */
    function getTraceInfo()public view returns(TraceData[] memory _data){
        return _traceData;
    }
}
```

## 合约功能解析

### 部署合约

![image](https://github.com/CN-Linzhisen/Test/assets/88913359/78fe33e6-dadb-459d-9adf-7ae995159f18)


### 根据商品组名获取对应的bytes32

> 输入商品组名（商标名），返回对应的bytes32。如输入`水果`，将返回`0x29e589ba4e2084476248da00a58ed9e6ebd7fee0d2dc0501513af2ba8d0157b2`。后续可根据该bytes32创建可溯源商品类别等。

```solidity
function getGoodsGroup(string memory name) public pure returns (bytes32) {
    return keccak256(abi.encode(name));
}
```

![image](https://github.com/CN-Linzhisen/Test/assets/88913359/5307227a-6df5-4f2c-a91f-0d8eb8425a81)


### 创建可溯源商品类别

> 输入商品组名（商标名）对应的bytes32，即由`getGoodsGroup`方法获取到的bytes32，如：`0x29e589ba4e2084476248da00a58ed9e6ebd7fee0d2dc0501513af2ba8d0157b2`。初始化对应的商品组信息，并将对应商品组信息返回。

```solidity
function createTraceability(bytes32  goodsGroup) public returns(Traceability) {
    // 判断商标（商品组）是否存在
    require(!_goodsCategory[goodsGroup].valid,"The trademark already exists" );
    // 初始化商品组
    Traceability category = new Traceability(goodsGroup);
    _goodsCategory[goodsGroup].valid = true;
    _goodsCategory[goodsGroup].trace = category;
    emit newTraceEvent(goodsGroup);
    return category;
}
```

![image](https://github.com/CN-Linzhisen/Test/assets/88913359/981714c7-2d90-464e-9adb-41bdd8ef2b91)


### 创建可溯源商品

> 输入商品组名（商标名）对应的bytes32，商品ID，如：goodsGroup: `0x29e589ba4e2084476248da00a58ed9e6ebd7fee0d2dc0501513af2ba8d0157b2`、goodsId: `1`。返回对应商品信息，如：`"addr": "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4", "status":0, 		"timestamp": "1692071077", "remark": "create"`。

```solidity
function createTraceGoods(bytes32  goodsGroup, uint64 goodsId) public returns(Goods) {
    // 获取可溯源商品类别信息
    Traceability category = getTraceability(goodsGroup);
    return category.createGoods(goodsId);
}
```

![image](https://github.com/CN-Linzhisen/Test/assets/88913359/def33aae-186b-4253-af07-9c6df8797589)


### 修改溯源商品

> 输入商品组名（商标名）、商品ID，商品状态，商品的摘要信息。如修改上述商品组名`水果`中，商品ID为`1`的商品、状态修改为`2`、并添加摘要数据`运输中`则输入：`goodsGroup: 0x29e589ba4e2084476248da00a58ed9e6ebd7fee0d2dc0501513af2ba8d0157b2、goodsId：1、goodsStatus：2、remark：运输中`。

```solidity
function changeTraceGoods(bytes32  goodsGroup, uint64 goodsId, int16 goodsStatus, string memory remark) public {
    // 获取可溯源商品类别信息
    Traceability category = getTraceability(goodsGroup);
    // 修改商品状态
    category.changeGoodsStatus(goodsId, goodsStatus, remark);
}
```

![image](https://github.com/CN-Linzhisen/Test/assets/88913359/2110e334-a6d6-4356-9b6c-a8e4c43c035c)


### 获取商品状态

> 输入商品组名（商标名）、商品ID，如需查看商品组名`水果`中，商品ID为`1`的商品状态，则输入：`goodsGroup: 0x29e589ba4e2084476248da00a58ed9e6ebd7fee0d2dc0501513af2ba8d0157b2、goodsId：1`。返回对应商品的状态，如`int16:2`。

```solidity
function getStatus(bytes32 goodsGroup, uint64 goodsId) public view returns(int16) {
    // 获取可溯源商品类别信息
    Traceability category = getTraceability(goodsGroup);
    return category.getStatus(goodsId);
}
```

![image](https://github.com/CN-Linzhisen/Test/assets/88913359/b7768089-2734-4542-8ed3-4e703f72ad19)


### 查询商品的整个过程

> 输入商品组名（商标名）、商品ID，如需查看商品组名`水果`中，商品ID为`1`的商品状态，则输入：`goodsGroup: 0x29e589ba4e2084476248da00a58ed9e6ebd7fee0d2dc0501513af2ba8d0157b2、goodsId：1`。返回对应商品的整个溯源过程，如：
>
> `0:tuple(address,int16,uint256,string)[]: `
>
> `0x5B38Da6a701c568545dCfcB03FcB875f56beddC4,0,1692071077,create,`
>
> `0x5B38Da6a701c568545dCfcB03FcB875f56beddC4,2,1692073700,运输中`。

```solidity
function getTraceInfo(bytes32 goodsGroup, uint64 goodsId) public view returns(Goods.TraceData[]) {
    // 获取可溯源商品类别信息
    Traceability category = getTraceability(goodsGroup);
    return category.getGoods(goodsId).getTraceInfo();
}
```

![image](https://github.com/CN-Linzhisen/Test/assets/88913359/7318b938-a669-4522-9df9-1209cb05f11b)
