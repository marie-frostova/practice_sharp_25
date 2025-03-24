# Задание 1.

Для того, чтобы системе маркетплейса справиться с повышенной нагрузкой в черную пятницу, необходимо сократить количество запросов к бутылочному горлышку нашей системы - к базе данных в операциях чтения. Одно из решений такой проблемы - это реализовать кэш, чтобы обращаться в него вместо непосредственно БД. Поскольку писать каждый раз для различных ситуаций свой кэш - довольно-таки затратная история, вы приняли решение решить эту проблему generic-кэшем с вытеснением.

```c#
public interface IDisplacementCache<T>
{
	int Capacity { get; set; }
	TimeSpan Ttl { get; set; }
	ITimeService TimeService { get; }
	
	IReadOnlyDictionary<string, T> GetAllCacheItems();
	T AddOrUpdate(string key, T item);
	T TryGet(string key);
	void ClearExpiredItems();
}
```

```c#
public interface ITimeService
{
	DateTime GetNowTime();
}
```

Параметр `Capacity` задает вместимость кэша. Если пытаемся добавлять элемент сверх вместимости, удаляем элемент, к которому обращались позже всего - предполагаем, что он самый неактуальный. `Ttl` задает время жизни элемента с момента последнего добавления - все устаревшие элементы удаляются в методе `ClearExpiredItems`, а также если пытаемся получить элемент через `TryGet`. `Ttl` обновляется только при добавлении элемента.


Подсказка: обратите внимание, что из-за того, что передаем сервис по работе с временем отдельно (вместо работы с DateTime.Now), можем написать тесты для проверки логики `Ttl` элементов - вместо того, чтобы физически ждать необходимое время, когда элементы "протухли", можем вручную переместиться в будущее.

Подсказка: обратите внимание, что `Capacity` и `Ttl` может меняться во время работы. При изменении  `Capacity`, должны при необходимости вытолкнуть лишние элементы. При изменении `Ttl` должны учесть, что старые элементы могут храниться t1 времени, а новые будут храниться уже t2 времени.

Написать тесты, которые проверяют различную работу с Capacity, Ttl.

# Задание 2.

Для маркетплейса необходимо реализовать сущности Продукта и Корзины и интерфейсы для работы с ними.

Добавьте класс `Product` с такими свойствами:
```
public Guid Id { get; set; }

public string Name { get; set; }

public string Description { get; set; }

public decimal Price { get; set; }

//сколько единиц товара в наличии
public int Amount { get; set; }

//кто продавец
public Guid SellerId { get; set; }
```

Добавьте класс `Cart` с такими свойствами:
```
public Guid Id { get; set; }

//какой продукт и сколько единиц в корзине	
public Dictionary<Guid, int> Products { get; set; }

public Guid CustomerId { get; set; }
```

У `Customer` добавьте Корзину:
```
public Cart Cart { get; set; }
```

У `Seller` добавьте список Продуктов:
```
public List<Product> Products { get; set; }
```

Для работы с Продуктами необходимы следующие методы. Вам нужно реализовать этот интерфейс.
```
public interface IProductsService  
{  
  //отдает всю информацию о продукте по его Id
  public Result<Product> GetProduct(Guid productId);  
  
  //отдает все продукты продавца
  public Result<List<Product>> GetSellerProducts(Guid sellerId);
  
  //отдает продукты, которые удовлетворяют поисковому запросу
  public Result<List<Product>> SearchProducts(ProductsSearchDto dto);  
  
  //добавляет продукт в корзину
  public Result<VoidResult> AddProductToCart(Guid cartId, Guid productId);  
  
  //создает продукт
  public Result<VoidResult> CreateProduct(CreateProductDto createProductDto); 
  
  //изменяет некоторые свойства продукта
  public Result<VoidResult> UpdateProduct(UpdateProductDto updateProductDto); 
  
  //удаляет продукт
  public Result<VoidResult> DeleteProduct(Guid productId);  
}
```

Вспомогательные сущности:

```
public sealed class ProductsSearchDto  
{  
    public string Query { get; get; }  
    public ProductsSortType ProductsSortType { get; get; } 
}

public enum ProductsSortType  
{  
    Popularity,  
    Price  
}
```

```
public sealed class CreateProductDto  
{  
    public string Name { get; get; }  
    public string Description { get; get; }  
    public decimal Price { get; get; }  
    public Guid Seller { get; get; }  
}
```

```
public sealed class UpdateProductDto  
{  
    public string Name { get; get; }  
    public string Description { get; get; }  
    public decimal Price { get; get; }  
    public int Amount { get; get; }  
}
```

Теперь реализуйте интерфейс для работы с Корзиной:

```
public interface ICartsService  
{  
    public Result<VoidResult> DeleteProductFromCartAsync(Guid cartId, Guid productId);  
    public Result<VoidResult> ChangeProductAmountAsync(Guid cart, Guid product, int delta);  
}
```

Подумайте, как организовать хранение данных.
