# NSCoding 与 Codable 的比较和迁移

翻译原文：[Migrating to Codable from NSCoding](https://medium.com/if-let-swift-programming/migrating-to-codable-from-nscoding-ddc2585f28a4)

---

> 好消息！伴随着 Swift 4 发布的还有 Codable 协议，允许数据在原生对象（native model）和外部表示（external representation）之间互相转化（后者包括 JSON，property list，XML等）；代替 NSCoding 进行 archive / unarchive 操作。

我们不禁要问：`Codable` 很厉害吗？放着好好的 `NSCoding` 不用，为什么要学习一个新协议？稍安勿躁，下面我们通过一个例子，对二者做出全面比较。



假设有 `Product` 类型，包含标题，价格，数量三个属性：

```
struct Product {
  var title:String
  var price:Double
  var quantity:Int
}
```



#### 支持 NSCoding

首先，我们让 `Product` 支持 NSCoding：

```
class Product: NSObject, NSCoding {
  var title:String
  var price:Double
  var quantity:Int
  enum Key:String {
    case title = "title"
    case price = "price"
    case quantity = "quantity"
  }
  init(title:String,price:Double, quantity:Int) {
    self.title = title
    self.price = price
    self.quantity = quantity
  }
  func encode(with aCoder: NSCoder) {
    aCoder.encode(title, forKey: Key.title.rawValue)
    aCoder.encode(price, forKey: Key.price.rawValue)
    aCoder.encode(quantity, forKey: Key.quantity.rawValue)
  }
  convenience required init?(coder aDecoder: NSCoder) {
    let price = aDecoder.decodeDouble(forKey: Key.price.rawValue)
    let quantity = aDecoder.decodeInteger(forKey: Key.quantity.rawValue)
    guard let title = aDecoder.decodeObject(forKey: Key.title.rawValue) as? String else { return nil }
    self.init(title:title,price:price,quantity:quantity)
  }
}
```

注意：

- 必须继承自 `NSObject`；
- 枚举 `Key` （结构体也可以）持有序列化时所需的 key（与属性名相同）；
- 遵守 `NSCoding` 并实现它的两个方法；
- 在 `encode` 方法中，使用 `NSCoder` 对属性进行序列化。



#### 支持 Codable

再看看 `Codable` 的实现：

```
struct Product: Codable {
  var title:String
  var price:Double
  var quantity:Int
  enum CodingKeys: String, CodingKey {
    case title
    case price
    case quantity
  }
  init(title:String,price:Double, quantity:Int) {
    self.title = title
    self.price = price
    self.quantity = quantity
  }
  func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy: CodingKeys.self)
    try container.encode(title, forKey: .title)
    try container.encode(price, forKey: .price)
    try container.encode(quantity, forKey: .quantity)
  }
  init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    title = try container.decode(String.self, forKey: .title)
    price = try container.decode(Double.self, forKey: .price)
    quantity = try container.decode(Int.self, forKey: .quantity)
  }
}
```



看起来差不多：

- 枚举 `CodingKeys` 负责维护序列化 key 与属性名的对应关系；
- 遵守 `Codable` 并实现它的两个方法；
- 在 `init` 方法中，使用 `Decoder` 初始化各项属性；
- 在 `encode` 方法中，使用 `NSCoder` 对属性进行序列化。



#### 比较两种实现


我们通过一张图对上述两种实现进行比较：

![比较两种实现](https://ws3.sinaimg.cn/large/006tNbRwly1fx5ep46i9lj30m80avads.jpg)

#### 为什么选择 Codable

即然它们如此相似，不就重复造轮子了吗？

非也，原因如下：

- `Codable` 不对实现者做任何要求，可以是任意类型（`struct`，`enum`，`class`）； 而 `NSCoding` 只能由 `NSObject` 及其子类实现。前者给予开发者更多的自由，更多可能；
- `Codable` 的本质在于统一所有序列化操作，包括 archive / unarchive，将原生对象转换为外部表示等操作；
- 解码、 编码互相独立。例如，如果只解析 API 返回的数据，则仅实现 `Decodable` 即可；
- 默认，如果条件允许，编译器将自动实现 `Codable`。这意味着上述代码可以省略。



![为什么选择 Codable](https://ws4.sinaimg.cn/large/006tNbRwly1fx5fhy2apkj30m80c8juu.jpg)



#### 手动实现 Codable

自动实现很酷，显示很残酷。有时我们必须亲自动手，比如：

- 某些属性不支持 Codable；
- 序列化后结构发生改变；
- 只序列化某些属性；
- 属性名与序列化 key 不一致；

####  archive / unarchive 操作对二者的使用

archive / unarchive 的本质是对序列化后的数据进行持久化处理。


首先是 `Product` + `NSCoding`：

```
func storeProducts() {
  let success = NSKeyedArchiver.archiveRootObject(products, toFile: productsFile.path)
  print(success ? "Successful save" : "Save Failed")
}
func retrieveProducts() -> [Product]? {
  return NSKeyedUnarchiver.unarchiveObject(withFile: productsFile.path) as? [Product]
}
```

然后是 `Product` + `NSCodable`：

```
func storeProducts() {
  do {
    let data = try PropertyListEncoder().encode(products)
    let success = NSKeyedArchiver.archiveRootObject(data, toFile: productsFile.path)
    print(success ? "Successful save" : "Save Failed")
  } catch {
    print("Save Failed")
  }
}
func retrieveProducts() -> [Product]? {
  guard let data = NSKeyedUnarchiver.unarchiveObject(withFile: productsFile.path) as? Data else { return nil }
  do {
    let products = try PropertyListDecoder().decode([Product].self, from: data)
    return products
  } catch {
    print("Retrieve Failed")
    return nil
  }
}
```

不难看出，`Codable` 需要我们书写更多代码。

![archive / unarchive 操作对二者的使用](https://ws3.sinaimg.cn/large/006tNbRwgy1fx6esrbh09j318g0fitdx.jpg)

有两点原因：

- `Codable` 允许序列化过程中抛出异常。这样的容错机制允许我们提前发现问题，避免程序直接崩溃；
- Encoder 是独立的，所以正确的顺序是：1. 序列化，2. archive。unarchive 同理。



#### 迁移至 Codable

好吧，`Codable` 看起来的确不错。那么问题来了，需要升级仍在使用 `NSCoding` 的老项目吗？

其实没必要， `Apple` 并未宣布废弃 `NSCoding`，所以继续使用也没问题。

但如果真的想迁移，又要怎么做呢？

如果仅修改 `NSCoding` 改为 `Codable`，程序可以运行，但是 unarchive 时会由于 `NSInvalidUnarchiveOperationException` 崩溃。

建议同时实现两个协议。老用户第一次使用新版本，通过 `NScoding` 读取数据；通过 `Codable` 保存数据。开发者可以在未来移除对 `NSCoding` 的支持。

实施起来也很简单。只需在实现 `NSCoding` 的基础上遵守 `Codable`，后者的实现由编译器自动生成（假设无需手动编写）。

```
class Product: NSObject, Codable, NSCoding {
```

这样，读取数据不会崩溃，因两个协议都支持了。

更新 `retrieveProducts` 方法，兼容任意编码方式：

```
func retrieveProducts() -> [Product]? {
  let unarchivedData = NSKeyedUnarchiver.unarchiveObject(withFile: productsFile.path)
  //Work with Codable
  if let data = unarchivedData as? Data {
    do {
      let decoder = PropertyListDecoder()
      let products = try decoder.decode([Product].self, from: data)
      return products
    } catch {
      print("Retrieve Failed")
      return nil
    }
  } 
  // Work with NSCoding
  else if let products = unarchivedData as? [Product] {
    return products
  } else {
    return nil
  }
}
```

后续使用 `Codable` 保存数据，之前的方法无需修改。

完成！现在我们已经成功迁移至 `Codable`，同时兼容 `NSCoding`。只需记得随后使用 `Codable` 保存数据即可。

