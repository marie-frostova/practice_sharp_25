# Предисловие.

Для сдачи заданий этого блока заведите в проекте App директорию Practice3. Важно, чтобы каждое задание было в отдельной ветке (pull request) на гите. Задания 1, 2, 3 необходимо выполнять последовательное. Задание 4 лучше выполнять в последнюю очередь.
# Задание 1.

В модели маркетплейса пользователи играют ключевую роль.
В проекте App создайте класс `User` с полями `id`, `login`, `password`, `name`, `surname`, `inn`, `phone`, `registerDate`.
Для `id` используйте тип `Guid`, для `registerDate` - `DateTime`.

Сделайте публичный конструктор `public User(string login, string password, string name, string surname, string inn, string phone)`. Как инициализировать поля, которые не передали в параметрах?

Сделайте публичный метод `string GetUserFullName()`, который возвращает строку с именем и фамилией пользователя.

Сделайте публичный метод `public bool TryUpdatePhone(string phone)`, который позволяет менять телефон пользователя и возвращает `true` - если поменять телефон удалось (прошла валидация), `false` - иначе. 
Для валидации телефона создайте приватный статический метод `private static bool IsPhoneValid(string phone)` в классе `User`. Можете переиспользовать предыдущие домашки.

Сделайте все поля приватными.

Сделайте поля, которые устанавливаются один раз в момент создания пользователя `readonly`. Какие это поля?

Теперь в `Main` попробуйте создать пару пользователей через конструктор.

У первого пользователя поменяйте телефон, а у второго выведите в консоль полное имя.

# Задание 2.

Продолжайте в классе `User`.

Для каждого поля сделайте публичное свойство. Для тех полей, у которых был модификатор `readonly` сделайте свойство `init`.

Добавьте логику по валидации в `setter` для свойства `Phone`. Теперь можете сделать метод `TryUpdatePhone` приватным.

Сделайте в `User` конструктор, который не принимает параметров, но внутри себя устанавливает `Id` и `RegisterDate`.

В `Main` попробуйте создать юзера с помощью инициализатора.

Теперь поменяйте свойство `Phone`.

Получится ли поменять свойство `Id`? Если нет, почему?

# Задание 3.

Создайте статический класс `UserCreator`, сделайте в нем публичный метод `CreateUser`, который по переданным пользователем реквизитам создает экземпляр `User`.

В классе `User` свойство `Password` заменяем на `PasswordHash`. 

В классе `User` теперь нужно изменить конструктор, чтобы принимал хеш вместо пароля. 

Cделайте внутри класса `UserCreator` приватный метод, который по паролю считает md5 hash. Используйте [метод](https://learn.microsoft.com/ru-ru/dotnet/api/system.security.cryptography.md5?view=net-8.0) из [System.Security.Cryptography](https://learn.microsoft.com/ru-ru/dotnet/api/system.security.cryptography?view=net-8.0).
# Задание 4.

Мы хотим получать статистику по действиям пользователей в маркетплейсе. Все действия пользователей, которые мы логируем, описаны перечислением `ActionTypes`.

Одна запись в логах - `UserActionItem` (когда, какое действие и сколько раз совершил пользоваетель).

Мы должны все эти записи в логах агрегировать по месяцам (или по дням, в зависимости от запроса) и положить в один словарик в классе `UserActionStatItem`.

Изучите пример.

Пример:

```
UserActionItems:
Date           Action                Count
09-12-24       Login                 1
10-12-24       SearchProducts        3
10-12-24       GetProductDetails     12
10-12-24       AddProductToCart      2
01-01-25       PayOrder              1
15-01-25       RecieveOrder          1
```

Если запрос `UserActionStatRequest` такой:
```
StartDate             10-12-24  
EndDate               30-01-25
DateGroupType         Daily
```

То ответ такой:
```
[
	{
		Start: 10-12-24
		End: 10-12-24
		Metrics: {
			"SearchProducts": 3,
			"GetProductDetails": 12,
			"AddProductToCart": 2
		}
	},
	{
		Start: 01-01-25
		End: 01-01-25
		Metrics: {
			"PayOrder ": 1
		}
	},
	{
		Start: 15-01-25
		End: 15-01-25
		Metrics: {
			"RecieveOrder": 1
		}
	},
]
```

Если запрос `UserActionStatRequest` такой:
```
StartDate             09-12-24  
EndDate               14-01-25
DateGroupType         Monthly
```

То ответ такой:
```
[
	{
		Start: 09-12-24
		End: 31-12-24
		Metrics: {
			"Login": 1,
			"SearchProducts": 3,
			"GetProductDetails": 12,
			"AddProductToCart": 2
		}
	},
	{
		Start: 01-01-25
		End: 14-01-25
		Metrics: {
			"PayOrder ": 1
		}
	}
]
```

Вам даны следующие сущности. Положите каждую сущность (класс или перечисление) в отдельный файл в папке Practice3 проекта App. 

Работайте только в классе `UserSatProvider`. Вам необходимо написать метод, который по запросу вернет статистику по действиям пользователя.

Старайтесь логически разбивать всю логику на кусочки и реализовывать их в отдельных приватных методах.

```csharp
public enum ActionTypes  
{  
    Login,  
    Logout,  
    SearchProducts,  
    GetProductDetails,  
    AddProductToCart,  
    RemoveProductFromCart,  
    PayOrder,  
    CancelOrder,  
    RecieveOrder  
}
  
public class UserActionItem  
{  
    public DateTime Date { get; set; }  
    public ActionTypes Action { get; set; }  
    public int Count { get; set; }  
}  
  
public class UserActionStatItem  
{  
    public DateTime StartDate { get; set; }  
    public DateTime EndDate { get; set; }  
    public Dictionary<ActionTypes, int> ActionMetrics { get; set; }  
}  
  
public class UserActionStatRequest  
{  
    public DateTime StartDate { get; set; }  
    public DateTime EndDate { get; set; }  
    public DateGroupTypes DateGroupType { get; set; }  
}

public enum DateGroupTypes  
{  
    Daily,  
    Monthly  
}
  
public class UserActionStatResponse  
{  
    public List<UserActionStatItem> UserActionStat { get; set; }  
}  
  
public class UserSatProvider  
{  
  public UserActionStatResponse GetUserActionStat(UserActionStatRequest request, List<UserActionItem> userActionItems)  
  {
    //TODO  
    throw new NotImplementedException();  
  }
}
```
