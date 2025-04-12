# Задание 0

Для маркетплейса осталось реализовать сущности счет и заказ.

```
public sealed class Account
{
    public Guid Id { get; set; }

    public decimal Balance { get; set; }
}
```

```
public sealed class Order
{
    public Guid Id { get; set; }

    public Dictionary<Guid, int> Products { get; set; } //продукт - количество

    public Guid TrustAccount { get; set; } //сюда будут переводится деньги до получения заказа

    public Guid CustomerId { get; set; }

    public Guid SellerId { get; set; }

    public DateTime OrderDate { get; set; }

    public OrderStatus Status { get; set; }
}

public enum OrderStatus
{
    PaidByCustomer,
    AcceptedBySeller,
    RejectedBySeller,
    CancelledByCustomer,
    WaitingForCustomer,
    RecievedByCustomer
}
```

А еще сервисы.

```
public interface IAccountsService
{
    public VoidResult CreateAccount();

    public VoidResult CashInAccount(Guid account, decimal amount); //перевести на счет

    public VoidResult CashOutAccount(Guid account); //вывести со счета

    public VoidResult TryBlockAccount(Guid account); //заморозка средств на специальном счете

    public Result<decimal> CheckAccount(Guid account); //проверка баланса

    public VoidResult DeleteTrustAccount(Guid account);
}
```

```
public interface IOrdersService
{
    public Result<VoidResult> AcceptOrder(Guid order);

    public Result<VoidResult> RejectOrder(Guid order);

    public Result<VoidResult> CancellOrder(Guid order);

    public Result<VoidResult> RecieveOrder(Guid order);
}
```

# Задание 1

В этой серии задач вам предстоит создать интерпретатор [Brainfuck](https://ru.wikipedia.org/wiki/Brainfuck) с возможностью его простого расширения новыми операциями. Решите эту задачу с помощью анонимных функций.

Необходимо создать папку Practice8, в нее положить следующие файлы:

Program.cs

```
using System;
using System.Linq;
using System.Text;
using NUnitLite;

namespace func.brainfuck;

public static class Program
{
    private const string SierpinskiTriangleBrainfuckProgram = @"
                                >
                               + +
                              +   -
                             [ < + +
                            +       +
                           + +     + +
                          >   -   ]   >
                         + + + + + + + +
                        [               >
                       + +             + +
                      <   -           ]   >
                     > + + >         > > + >
                    >       >       +       <
                   < <     < <     < <     < <
                  <   [   -   [   -   >   +   <
                 ] > [ - < + > > > . < < ] > > >
                [                               [
               - >                             + +
              +   +                           +   +
             + + [ >                         + + + +
            <       -                       ]       >
           . <     < [                     - >     + <
          ]   +   >   [                   -   >   +   +
         + + + + + + + +                 < < + > ] > . [
        -               ]               >               ]
       ] +             < <             < [             - [
      -   >           +   <           ]   +           >   [
     - < + >         > > - [         - > + <         ] + + >
    [       -       <       -       >       ]       <       <
   < ]     < <     < <     ] +     + +     + +     + +     + +
  +   .   +   +   +   .   [   -   ]   <   ]   +   +   +   +   +
";

    public static void Main(string[] args)
    {
        if (args.Contains("test"))
        {
            new AutoRun().Execute(Array.Empty<string>());
        }
        else
        {
            Brainfuck.Run(SierpinskiTriangleBrainfuckProgram, Console.Read, Console.Write);
            Console.WriteLine("Это была демонстрация Brainfuck на примере построения треугольника Серпинского");
        }

        Console.ReadLine();
    }
}

public static class Brainfuck
{
    public static string Run(string program, string input = "")
    {
        var inputIndex = 0;
        var output = new StringBuilder();
        Run(program,
            () => inputIndex >= input.Length ? -1 : input[inputIndex++],
            c => output.Append(c));
        return output.ToString();
    }

    public static void Run(string program, Func<int> read, Action<char> write, int memorySize = 30000)
    {
        var vm = new VirtualMachine(program, memorySize);
        BrainfuckBasicCommands.RegisterTo(vm, read, write);
        BrainfuckLoopCommands.RegisterTo(vm);
        vm.Run();
    }
}
```

VirtualMachine.cs

```
using System;
using NUnit.Framework;

namespace func.brainfuck;

public interface IVirtualMachine
{
    void RegisterCommand(char symbol, Action<IVirtualMachine> execute);
    string Instructions { get; }
    int InstructionPointer { get; set; }
    byte[] Memory { get; }
    int MemoryPointer { get; set; }
    void Run();
}
	
public class VirtualMachine : IVirtualMachine
{
    public string Instructions { get; }
    public int InstructionPointer { get; set; }
    public byte[] Memory { get; }
    public int MemoryPointer { get; set; }

    public VirtualMachine(string program, int memorySize)
    {
    }

    public void RegisterCommand(char symbol, Action<IVirtualMachine> execute)
    {
        throw new NotImplementedException();
    }

    public void Run()
    {
        throw new NotImplementedException();
    }
}

[TestFixture]
public class VirtualMachineTests
{
	[Test]
	public void ImplementIVirtualMachine()
	{
		Assert.IsTrue(new VirtualMachine("", 1) is IVirtualMachine);
	}
	
	[Test]
	public void Initialize()
	{
		var vm = new VirtualMachine("xxx", 12345);
		vm.RegisterCommand('x', b => { });
		Assert.AreEqual(12345, vm.Memory.Length);
		Assert.AreEqual(0, vm.MemoryPointer);
		Assert.AreEqual("xxx", vm.Instructions);
		Assert.AreEqual(0, vm.InstructionPointer);
	}

	[Test]
	public void SetMemorySize()
	{
		var vm = new VirtualMachine("", 42);
		Assert.AreEqual(42, vm.Memory.Length);
	}

	[Test]
	public void IncrementInstructionPointer()
	{
		var res = "";
		var vm = new VirtualMachine("xy", 10);
		vm.RegisterCommand('x', b => { res += "x->" + b.InstructionPointer + ", "; });
		vm.RegisterCommand('y', b => { res += "y->" + b.InstructionPointer; });
		vm.Run();
		Assert.AreEqual("x->0, y->1", res);
	}

	[Test]
	public void MoveInstructionPointerForward()
	{
		var res = "";
		var vm = new VirtualMachine("xyz", 10);
		vm.RegisterCommand('x', b => { b.InstructionPointer++; });
		vm.RegisterCommand('y', b => { Assert.Fail(); });
		vm.RegisterCommand('z', b => { res += "z"; });
		vm.Run();
		Assert.AreEqual("z", res);
	}
	    
	[Test]
	public void InstructionPointerCanMovedOutside()
	{
		var res = "";
		var vm = new VirtualMachine("yxz", 10);
		vm.RegisterCommand('x', b => { res += "x"; });
		vm.RegisterCommand('y', b => { Assert.Fail(); });
		vm.RegisterCommand('z', b => { res += "z"; });
		vm.InstructionPointer++;
		vm.Run();
		Assert.AreEqual("xz", res);
	}

	[Test]
	public void MoveInstructionPointerBackward()
	{
		var res = "";
		var vm = new VirtualMachine(">><", 10);
		vm.RegisterCommand('>', b =>
		{
			b.InstructionPointer++;
			res += ">";
		});
		vm.RegisterCommand('<', b =>
		{
			b.InstructionPointer -= 2;
			res += "<";
		});
		vm.Run();
		Assert.AreEqual("><>", res);
	}
	
	[Test]
	public void ChangeMemoryPointer()
	{
		var vm = new VirtualMachine("xy", 10);
		vm.RegisterCommand('x', b => { b.MemoryPointer = b.MemoryPointer + 42; });
		vm.RegisterCommand('y', b => { Assert.AreEqual(42, b.MemoryPointer); });
		vm.Run();
	}

	[Test]
	public void SkipUnknownCommands()
	{
		new VirtualMachine("xyz", 10).Run();
	}

	[Test]
	public void ReadWriteMemory()
	{
		var vm = new VirtualMachine("wr", 10);
		vm.RegisterCommand('w', b => { b.Memory[3] = 42; });
		vm.RegisterCommand('r', b => { Assert.AreEqual(42, b.Memory[3]); });
		vm.Run();
	}

	[Test]
	public void RunInstructionsInRightOrder()
	{
		var vm = new VirtualMachine("abbaaa", 10);
		var res = "";
		vm.RegisterCommand('a', b => { res += "a"; });
		vm.RegisterCommand('b', b => { res += "b"; });
		if (res != "")
			Assert.Fail("Instructions should not be executed before Run() call");
		vm.Run();
		if (res != "")
			Assert.AreEqual("abbaaa", res, "Execution order of program 'abbaaa'");
	}

	[Test]
	public void RunManyInstructions()
	{
		var vm = new VirtualMachine(new string('a', 10000), 10);
		var count = 0;
		vm.RegisterCommand('a', b => { count++; });
		vm.Run();
		Assert.AreEqual(10000, count, "Number of executed instructions");
	}
}
```

BrainfuckBasicCommands.cs

```
using System;
using NUnit.Framework;

namespace func.brainfuck;

public static class BrainfuckBasicCommands
{
	public static void RegisterTo(IVirtualMachine vm, Func<int> read, Action<char> write)
	{
		vm.RegisterCommand('.', b => { });
		vm.RegisterCommand('+', b => {});
		vm.RegisterCommand('-', b => {});
		//...
	}
}

[TestFixture]
public class BrainfuckBasicCommandsTests
{
	[Test]
	public void Print()
	{
		Assert.AreEqual("\0", Run("."));
	}

	[Test]
	public void Inc()
	{
		Assert.AreEqual("\x1", Run("+."));
		Assert.AreEqual("\x5", Run("+++++."));
		Assert.AreEqual("A", Run(new string('+', 'A') + "."));
		Assert.AreEqual("Z", Run(new string('+', 'Z') + "."));
		Assert.AreEqual("\xFF", Run(new string('+', 255) + "."));
	}

	[Test]
	public void Dec()
	{
		Assert.AreEqual("\0", Run("+-."));
		Assert.AreEqual("\x1", Run("+++--."));
		Assert.AreEqual("A", Run(new string('+', 'C') + "--."));
	}

	[Test]
	public void IncOverflow()
	{
		Assert.AreEqual("\x1", Run(new string('+', 257) + "."));
	}

	[Test]
	public void DecOverflow()
	{
		Assert.AreEqual("\xFF", Run("-."));
		Assert.AreEqual("\x1", Run(new string('-', 255) + "."));
	}

	[Test]
	public void Shift()
	{
		Assert.AreEqual(3, Vm(">>>").MemoryPointer);
		Assert.AreEqual(2, Vm(">>><").MemoryPointer);
		Assert.AreEqual(1, Vm(">>><<").MemoryPointer);
		Assert.AreEqual(0, Vm("><").MemoryPointer);
		Assert.AreEqual("\x2", Run("+>++."));
		Assert.AreEqual("\x1", Run("+>++<."));
		Assert.AreEqual("\x1", Run("+>++>+++<<."));
	}

	[Test]
	public void ShiftOverflow()
	{
		Assert.AreEqual(2, Vm("<", 3).MemoryPointer);
		Assert.AreEqual(0, Vm("<>", 3).MemoryPointer);
		Assert.AreEqual(0, Vm(">>>", 3).MemoryPointer);
		Assert.AreEqual(1, Vm(">>>>", 3).MemoryPointer);
		Assert.AreEqual("\x1", Run("<+."));
		Assert.AreEqual("\x2", Run("++<>."));
		Assert.AreEqual("\x1", Run("+<++<+++>>."));
		Assert.AreEqual("\x3", Run("+++" + new string('>', 30000) + "."));
		Assert.AreEqual("\x3", Run("+++" + new string('<', 30000) + "."));
	}

	[Test]
	public void Read()
	{
		Assert.AreEqual("A", Run(",.", "A"));
		Assert.AreEqual("ABC", Run(",.,.,.", "ABC"));
	}

	[Test]
	public void HelloWorld()
	{
		Assert.AreEqual("Hello World!\n", Run(@"
 +++++++++++++++++++++++++++++++++++++++++++++
 +++++++++++++++++++++++++++.+++++++++++++++++
 ++++++++++++.+++++++..+++.-------------------
 ---------------------------------------------
 ---------------.+++++++++++++++++++++++++++++
 ++++++++++++++++++++++++++.++++++++++++++++++
 ++++++.+++.------.--------.------------------
 ---------------------------------------------
 ----.-----------------------.
"));
	}

	[Test]
	public void Constants()
	{
		Assert.AreEqual("QWERTYUIOPASDFGHJKLZXCVBNMqwertyuiopasdfghjklzxcvbnm1234567890",
			Run(
				"Q.W.E.R.T.Y.U.I.O.P.A.S.D.F.G.H.J.K.L.Z.X.C.V.B.N.M." +
				"q.w.e.r.t.y.u.i.o.p.a.s.d.f.g.h.j.k.l.z.x.c.v.b.n.m." +
				"1.2.3.4.5.6.7.8.9.0."));
	}

	[Test]
	public void IgnoreOtehrSymbols()
	{
		Assert.AreEqual("Q", 
			Run("Q!@#$%&*()."));
	}

	[Test]
	public void HelloWorldWithConstants()
	{
		Assert.AreEqual("Hello world",
			Run("H>e>l>l>o>++++++++++++++++++++++++++++++++>w>o>r>l>d<<<<<<<<<<.>.>.>.>.>.>.>.>.>.>."));
	}

	private string Run(string program, string input = "")
	{
		return Brainfuck.Run(program, input);
	}

	private IVirtualMachine Vm(string program, int memorySize = 10)
	{
		var vm = new VirtualMachine(program, memorySize);
		BrainfuckBasicCommands.RegisterTo(vm, () => -1, c => {});
		BrainfuckLoopCommands.RegisterTo(vm);
		vm.Run();
		return vm;
	}
}
```

BrainfuckLoopCommands.cs

```
using System.Text;
using NUnit.Framework;

namespace func.brainfuck;

public static class BrainfuckLoopCommands
{
	public static void RegisterTo(IVirtualMachine vm)
	{
		vm.RegisterCommand('[', b => { });
		vm.RegisterCommand(']', b => { });
	}
}

[TestFixture]
public class BrainfuckLoopCommandsTests
{
	[Test]
	public void Loops()
	{
		Assert.AreEqual("\x0", Run("[+.]."));
		Assert.AreEqual("\x1\x0", Run("+[.-]."));
		Assert.AreEqual("\x3\x2\x1\x0", Run("+++[.-]."));
	}

	[Test]
	public void NestedLoops()
	{
		Assert.AreEqual("\x4", Run("++[>++[>+<-]<-]>>."));
	}

	[Test]
	public void HelloWorldWithLoops()
	{
		Assert.AreEqual("Hello World!\n", Run(@"
++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++
.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.
------.--------.>+.>."));
	}

	[Test]
	public void BottlesOfBeer()
	{
		var text = Run(@"
>+++++++++[<+++++++++++>-]<[>[-]>[-]<<[>+>+<<-]>>[<<+>>-]>>>
[-]<<<+++++++++<[>>>+<<[>+>[-]<<-]>[<+>-]>[<<++++++++++>>>+<
-]<<-<-]+++++++++>[<->-]>>+>[<[-]<<+>>>-]>[-]+<<[>+>-<<-]<<<
[>>+>+<<<-]>>>[<<<+>>>-]>[<+>-]<<-[>[-]<[-]]>>+<[>[-]<-]<+++
+++++[<++++++<++++++>>-]>>>[>+>+<<-]>>[<<+>>-]<[<<<<<.>>>>>-
]<<<<<<.>>[-]>[-]++++[<++++++++>-]<.>++++[<++++++++>-]<++.>+
++++[<+++++++++>-]<.><+++++..--------.-------.>>[>>+>+<<<-]>
>>[<<<+>>>-]<[<<<<++++++++++++++.>>>>-]<<<<[-]>++++[<+++++++
+>-]<.>+++++++++[<+++++++++>-]<--.---------.>+++++++[<------
---->-]<.>++++++[<+++++++++++>-]<.+++..+++++++++++++.>++++++
++[<---------->-]<--.>+++++++++[<+++++++++>-]<--.-.>++++++++
[<---------->-]<++.>++++++++[<++++++++++>-]<++++.-----------
-.---.>+++++++[<---------->-]<+.>++++++++[<+++++++++++>-]<-.
>++[<----------->-]<.+++++++++++..>+++++++++[<---------->-]<
-----.---.>>>[>+>+<<-]>>[<<+>>-]<[<<<<<.>>>>>-]<<<<<<.>>>+++
+[<++++++>-]<--.>++++[<++++++++>-]<++.>+++++[<+++++++++>-]<.
><+++++..--------.-------.>>[>>+>+<<<-]>>>[<<<+>>>-]<[<<<<++
++++++++++++.>>>>-]<<<<[-]>++++[<++++++++>-]<.>+++++++++[<++
+++++++>-]<--.---------.>+++++++[<---------->-]<.>++++++[<++
+++++++++>-]<.+++..+++++++++++++.>++++++++++[<---------->-]<
-.---.>+++++++[<++++++++++>-]<++++.+++++++++++++.++++++++++.
------.>+++++++[<---------->-]<+.>++++++++[<++++++++++>-]<-.
-.---------.>+++++++[<---------->-]<+.>+++++++[<++++++++++>-
]<--.+++++++++++.++++++++.---------.>++++++++[<---------->-]
<++.>+++++[<+++++++++++++>-]<.+++++++++++++.----------.>++++
+++[<---------->-]<++.>++++++++[<++++++++++>-]<.>+++[<----->
-]<.>+++[<++++++>-]<..>+++++++++[<--------->-]<--.>+++++++[<
++++++++++>-]<+++.+++++++++++.>++++++++[<----------->-]<++++
.>+++++[<+++++++++++++>-]<.>+++[<++++++>-]<-.---.++++++.----
---.----------.>++++++++[<----------->-]<+.---.[-]<<<->[-]>[
-]<<[>+>+<<-]>>[<<+>>-]>>>[-]<<<+++++++++<[>>>+<<[>+>[-]<<-]
>[<+>-]>[<<++++++++++>>>+<-]<<-<-]+++++++++>[<->-]>>+>[<[-]<
<+>>>-]>[-]+<<[>+>-<<-]<<<[>>+>+<<<-]>>>[<<<+>>>-]<>>[<+>-]<
<-[>[-]<[-]]>>+<[>[-]<-]<++++++++[<++++++<++++++>>-]>>>[>+>+
<<-]>>[<<+>>-]<[<<<<<.>>>>>-]<<<<<<.>>[-]>[-]++++[<++++++++>
-]<.>++++[<++++++++>-]<++.>+++++[<+++++++++>-]<.><+++++..---
-----.-------.>>[>>+>+<<<-]>>>[<<<+>>>-]<[<<<<++++++++++++++
.>>>>-]<<<<[-]>++++[<++++++++>-]<.>+++++++++[<+++++++++>-]<-
-.---------.>+++++++[<---------->-]<.>++++++[<+++++++++++>-]
<.+++..+++++++++++++.>++++++++[<---------->-]<--.>+++++++++[
<+++++++++>-]<--.-.>++++++++[<---------->-]<++.>++++++++[<++
++++++++>-]<++++.------------.---.>+++++++[<---------->-]<+.
>++++++++[<+++++++++++>-]<-.>++[<----------->-]<.+++++++++++
..>+++++++++[<---------->-]<-----.---.+++.---.[-]<<<]");

		Assert.IsTrue(text.Contains("99 Bottles of beer on the wall"));
		Assert.IsTrue(text.Contains("Take one down and pass it around"));
		Assert.IsTrue(text.Contains("43 Bottles of beer on the wall"));
		Assert.IsTrue(text.Contains("1 Bottle of beer on the wall"));
		Assert.IsTrue(text.Contains("0 Bottles of beer on the wall"));
	}

	[Test]
	public void PairBracetsDictionaryIsNotStatic()
	{
		Run(@"
++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++
.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.
------.--------.>+.>.");
			
		Assert.AreEqual("Hello World!\n", Run(@"
++++++++++[>++++<>+++>++++++++++>+++>+<<<<-]>++
.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.
------.--------.>+.>."));
	}

	[Test]
	[MaxTime(1000)]
	public void LoopsPerformance()
	{
		var s = new string('.', 10000);
		Run("0[->0[->0[->[" + s + "]<]<]<]");
	}

	[Test]
	[MaxTime(1000)]
	public void NestedLoopsPerformance()
	{
		var s = new StringBuilder();
		var size = 50000;
		for (int i = 0; i < size; i++) s.Append("+[->.");
		s.Append("+[-]<");
		for (int i = 0; i < size; i++) s.Append("]<");
		var program = s.ToString();
		Run(program);
	}

	[Test]
	[MaxTime(2000)]
	public void NestedLoopsPerformance2()
	{
		var s = new StringBuilder();
		var size = 30000;
		for (int i = 0; i < size; i++) s.Append("++>");
		for (int i = 0; i < size; i++) s.Append("<[-");
		for (int i = 0; i < size; i++) s.Append("]>");
		var program = s.ToString();
		Run(program);
	}

	private string Run(string program, string input = "")
	{
		return Brainfuck.Run(program, input);
	}
}
```

## Часть 1

Виртуальная машина хранит следующее:

- Массив памяти, каждая ячейка которого хранит 1 байт. По умолчанию размер памяти — 30000 ячеек.
- Указатель на текущую ячейку памяти. Изначально, указатель указывает на нулевую ячейку.
- Выполняемую программу. Она состоит из инструкций, каждая обозначается одним символом. Программа начинает выполняться с первого символа последовательно.
- Номер выполняемой в данный момент инструкции. После выполнения любой инструкции номер увеличивается на единицу. Как только номер инструкции выходит за пределы программы, выполнение заканчивается.
- Конкретные операции на языке Brainfuck могут читать или менять эти данные.

В этой части вам нужно реализовать виртуальную машину в классе VirtualMachine так, чтобы проходили все тесты из VirtualMachineTests.

## Часть 2

Изучите класс Brainfuck, в частности то, как он использует реализованный ранее класс VirtualMachine.

В классе BrainfuckBasicCommands реализуйте метод, регистрирующий следующие простые команды в виртуальную машину:

```
.	              Вывести байт памяти, на который указывает указатель, преобразовав в символ согласно ASCII
+	              Увеличить байт памяти, на который указывает указатель
-	              Уменьшить байт памяти, на который указывает указатель
,	              Ввести символ и сохранить его ASCII-код в байт памяти, на который указывает указатель
>	              Сдвинуть указатель памяти вправо на 1 байт
<	              Сдвинуть указатель памяти влево на 1 байт
A-Z, a-z, 0-9         Cохранить ASCII-код этого символа в байт памяти, на который указывает указатель
```

Например, программа `++>+++.<.` выводит два символа с ASCII кодами 2 и 3, а память после выполнения команды будет выглядеть так `[2, 3, 0, 0, ... 0]`.

Для ввода и вывода используйте переданные в метод Run функции Func<int> read и Action<char> write.

Тут read по аналогии с Console.Read возвращает либо код введенного символа, либо -1, если ввод закончился. Считайте, что на вход будут подаваться только символы с кодами 0..255 — они точно помещаются в один байт.

Детали реализации инструкций восстановите по тестам в классе BrainfuckBasicCommandsTests. Сделайте так, чтобы все тесты в этом файле проходили.

## Часть 3

В классе BrainfuckLoopCommands реализуйте метод, регистрирующий следующие команды в виртуальную машину:

`[`	(Начало цикла) Перескочить по программе вправо на соответствующий (с учетом вложенности) символ ], если текущий байт памяти равен нулю. Продолжать исполнение с этого символа.

`]`	(Конец цикла) Перескочить по списку инструкций влево на соответствующий (с учетом вложенности) символ [, если текущий байт памяти НЕ равен нулю. Продолжать исполнение с этого символа.

Например, программа `++++++++[>++++++++<-]>+`. выводит букву A (ASCII-код 65 получается увеличением 8 раз второй ячейки на 8, а потом добавлением ещё единицы).
Детали реализации инструкций восстановите по тестам в классе BrainfuckLoopCommandsTests. Сделайте так, чтобы все тесты в этом файле проходили.
