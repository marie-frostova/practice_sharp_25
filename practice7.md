# Задание 1

В нашем маркетплейсе есть отдел аналитики, который анализирует статистику по дашбордам в режиме реального времени. Но для полноценного анализа им нехватает некоторых функций для временных рядов.
Для этого они просят вас реализовать методы, которые позволят:
- делать [экспоненциальное сглаживание](https://en.wikipedia.org/wiki/Exponential_smoothing#The_exponential_moving_average)
- искать среднее в окне (подсказка: используйте очередь)
- искать максимум в окне.

Реализуйте методы в директории `Practice7`, файла `TimeSeriesStat.cs`.

```
public static class ExpSmoothingTask
{
  public static IEnumerable<DataPoint> SmoothExponentialy(this IEnumerable<DataPoint> data, double alpha)
  {
    //Fix me!
    return data;
  }
}

public static class MovingAverageTask
{
  public static IEnumerable<DataPoint> MovingAverage(this IEnumerable<DataPoint> data, int windowWidth)
  {
    //Fix me!
    return data;
  }
}

public static class MovingMaxTask
{
  public static IEnumerable<DataPoint> MovingMax(this IEnumerable<DataPoint> data, int windowWidth)
  {
    //Fix me!
    return data;
  }
}
```

Вспомогательный класс можете положить в соседнем файле:
```
public class DataPoint
{
	public readonly double X;
	public readonly double OriginalY;
	public double ExpSmoothedY { get; private set; }
	public double AvgSmoothedY { get; private set; }
	public double MaxY { get; private set; }

	public DataPoint(double x, double y)
	{
		X = x;
		OriginalY = y;
	}

	public DataPoint(DataPoint point)
	{
		X = point.X;
		OriginalY = point.OriginalY;
		ExpSmoothedY = point.ExpSmoothedY;
		AvgSmoothedY = point.AvgSmoothedY;
		MaxY = point.MaxY;
	}

	public DataPoint WithExpSmoothedY(double expSmoothedY)
	{
		return new DataPoint(this) { ExpSmoothedY = expSmoothedY };
	}

	public DataPoint WithAvgSmoothedY(double avgSmoothedY)
	{
		return new DataPoint(this) { AvgSmoothedY = avgSmoothedY };
	}

	public DataPoint WithMaxY(double maxY)
	{
		return new DataPoint(this) { MaxY = maxY };
	}
}
```

# Задание 2

Ваш продакт решил, что было бы неплохо показывать покупателю медиану по ценам на одинаковые товары от разных продавцов, чтобы пользователь мог ориентироваться, насколько цена соответствует рынку.
А еще было бы релевантно показывать покупателю похожие товары в том же ценовом диапазоне.
Для этого нужно реализовать функцию поиска медианы и функцию, которая умеет возвращать последовательность, состоящую из пар соседних по ценам товаров.

Реализуйте методы в директории `Practice7`, файла `ExtensionsTask.cs`.

```
public static class ExtensionsTask
{
  /// <summary>
  /// Медиана списка из нечетного количества элементов — это серединный элемент списка после сортировки.
  /// Медиана списка из четного количества элементов — это среднее арифметическое 
  /// двух серединных элементов списка после сортировки.
  /// </summary>
  /// <exception cref="InvalidOperationException">Если последовательность не содержит элементов</exception>
  public static double Median(this IEnumerable<double> items)
  {
    throw new NotImplementedException();
  }
  
  /// <returns>
  /// Возвращает последовательность, состоящую из пар соседних элементов.
  /// Например, по последовательности {1,2,3} метод должен вернуть две пары: (1,2) и (2,3).
  /// </returns>
  public static IEnumerable<(T First, T Second)> Bigrams<T>(this IEnumerable<T> items)
  {
    throw new NotImplementedException();
  }
}
```

# Задание 3

Пришло время немного порефакторить код нашего маркетплейса. А именно, давайте вернемся к классу `UsersService` из задания 4, и внедрим туда подход с использованием `Result<>` для всех методов интерфейса.

Кроме этого, давайте сделаем более низкоуровневый интерфейс для хранения сущностей `IRepository<TEntity>`. Код интерфейса репозитория следующий:

```c#
public interface IRepository<TEntity>
	where TEntity : IIdentifiable
{
	IReadOnlyList<TEntity> GetAll();
	Result<TEntity> GetById(Guid id);
	Result<VoidResult> Delete(Guid id);
	Result<VoidResult> AddNew(TEntity entity);
	Result<VoidResult> Update(Guid id, TEntity entity);
}
```

```c#
public interface IIdentifiable
{
	public Guid Id { get; }
}
```

Используйте соответствующие репозитории в классах `ProductsService` (задание 6) и `UsersService` (задание 4).

# Задание 4

Коллега по цеху прибежал и слезно попросил взять его задачу, так как он уходит в отпуск. Задача чисто техническая: необходимо реализовать функцию ZipSum с использованием yield return, которая принимает на вход две последовательности целых чисел и возвращает последовательность, состоящую из попарных сумм их элементов. 

Подсказка: можно считать, что входные последовательности одинаковой длины.

```c#
public static IEnumerable<int> ZipSum(IEnumerable<int> first, IEnumerable<int> second)
{
    var e1 = first.GetEnumerator();
    var e2 = second.GetEnumerator();
// ...
}
```
