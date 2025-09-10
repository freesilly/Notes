在C#中，实现回调机制主要依赖于**委托（Delegate）**。委托本质上是一个类型安全的函数指针，它可以“持有”一个或多个方法的引用。

我们从最基础到最常用的方式来逐一讲解。

#### 1. 委托 (Delegate) - 基础构件

委托是实现回调的基石。

**第一步：定义一个委托类型**  
这个定义规定了回调方法必须拥有的“签名”（即返回类型和参数列表）。
```
// 定义一个委托，它指向的方法必须没有返回值(void)，且接受一个字符串参数。
public delegate void NotifyCallback(string message);
```
**第二步：创建一个需要回调的方法（被调用方）**  
这个方法接受一个我们刚才定义的委托作为参数。
```
public class Processor
{
    // 这个方法模拟一个长时间运行的任务
    // 它接受一个回调委托，以便在完成时通知调用方
    public void ProcessData(NotifyCallback callback)
    {
        Console.WriteLine("开始处理数据...");
        // 模拟耗时操作
        System.Threading.Thread.Sleep(3000); 
        Console.WriteLine("数据处理完成！");

        // 关键：在这里执行回调！
        // 调用方传进来的方法会在这里被执行。
        callback("任务成功结束。");
    }
}
```
**第三步：创建回调方法并注册（调用方）**  
调用方需要提供一个符合委托签名的方法，并在调用 ProcessData 时将其传入。
```
public class Program
{
    // 这是一个符合 NotifyCallback 委托签名的方法
    public static void OnProcessComplete(string message)
    {
        Console.WriteLine($"[回调] 收到来自 Processor 的通知: {message}");
    }

    public static void Main()
    {
        Processor processor = new Processor();

        // 1. 创建委托实例，并让它指向我们的回调方法
        NotifyCallback myCallback = new NotifyCallback(OnProcessComplete);

        // 2. 将委托实例传递给 ProcessData 方法。这个过程就是“注册回调”。
        Console.WriteLine("即将调用 ProcessData 方法...");
        processor.ProcessData(myCallback);
        // 也可以直接传入方法名，C#会自动创建委托实例
        // processor.ProcessData(OnProcessComplete); 

        Console.WriteLine("Main 方法继续执行其他任务...");
    }
}
```
**执行流程分析：**

1. Main 方法调用 processor.ProcessData，并将 OnProcessComplete 方法作为回调“注册”进去。
    
2. ProcessData 开始执行，Main 方法理论上可以继续做别的事（虽然在这个同步例子里它会等待）。
    
3. ProcessData 内部任务完成后，通过 callback(...) 调用了 OnProcessComplete。
    
4. 控制权暂时回到了 Program 类的 OnProcessComplete 方法，打印出完成信息。
    
#### 2. Action<> 和 Func<> - 简化的委托

每次都 delegate void ... 太麻烦了。.NET 提供了内置的泛型委托 Action<> 和 Func<>，让代码更简洁。

- Action<>：用于指向没有返回值 (void) 的方法。
    
- Func<>：用于指向有返回值的方法。
    
用 Action 改进上面的例子：

```
public class Processor
{
    // 直接使用 Action<string>，无需再手动定义委托
    public void ProcessData(Action<string> callback)
    {
        Console.WriteLine("开始处理数据 (使用 Action)...");
        System.Threading.Thread.Sleep(3000);
        Console.WriteLine("数据处理完成！");
        callback("任务成功结束。");
    }
}

// 调用方的代码几乎不变
public class Program
{
    public static void OnProcessComplete(string message)
    {
        Console.WriteLine($"[回调] 收到来自 Processor 的通知: {message}");
    }

    public static void Main()
    {
        Processor processor = new Processor();
        // 直接传入方法，非常简洁
        processor.ProcessData(OnProcessComplete);
        
        // 甚至可以直接使用 Lambda 表达式定义一个匿名的回调方法
        processor.ProcessData(message => {
            Console.WriteLine($"[Lambda 回调] 通知: {message}");
        });
    }
}
```
这是现代C#中最常见的简单回调实现方式。

#### 3. 事件 (Event) - 最正式、最安全的模式

当一个类需要向**多个**外部订阅者通知某件事发生时，使用事件是最佳选择。事件是基于委托的、更高级的封装，提供了更强的安全性和封装性。

它引入了**发布者-订阅者 (Publisher-Subscriber)** 模式。

- **发布者**：拥有事件的类，它负责在特定时机“触发”事件。
    
- **订阅者**：对事件感兴趣的类，它将自己的方法（事件处理器）“注册”到事件上。
    

**第一步：在发布者中定义事件**  
事件通常基于 Action<> 或 EventHandler<> 委托。
```
public class Downloader
{
    // 1. 定义一个委托来描述事件处理器的签名
    //    (通常使用 Action 或 EventHandler)
    // 2. 定义一个基于该委托的 public event
    public event Action<int> OnProgressChanged; // 报告下载进度
    public event Action<string> OnDownloadComplete; // 下载完成通知

    public void StartDownload(string url)
    {
        Console.WriteLine($"开始下载: {url}");
        for (int i = 0; i <= 100; i += 10)
        {
            System.Threading.Thread.Sleep(200); // 模拟下载耗时
            
            // 3. 触发事件，通知所有订阅者
            //    使用 ?. 来安全调用，防止没有订阅者时抛出空引用异常
            OnProgressChanged?.Invoke(i);
        }

        string result = "下载内容...";
        // 4. 任务完成，触发完成事件
        OnDownloadComplete?.Invoke(result);
    }
}
```
**第二步：在订阅者中注册事件处理器**  
订阅者使用 += 操作符来“注册”或“订阅”事件，使用 -= 来“注销”或“取消订阅”。
```
public class UI
{
    public void ShowDownloadProgress(int progress)
    {
        Console.WriteLine($"[UI] 下载进度: {progress}%");
    }

    public void HandleDownloadComplete(string content)
    {
        Console.WriteLine($"[UI] 下载完成！内容长度: {content.Length}");
    }
}

public class Logger
{
    public void LogDownloadComplete(string content)
    {
        Console.WriteLine($"[日志] 文件下载成功。");
    }
}

public class Program
{
    public static void Main()
    {
        Downloader downloader = new Downloader();
        UI ui = new UI();
        Logger logger = new Logger();

        // --- 这就是注册回调的过程 ---
        // UI 订阅了两个事件
        downloader.OnProgressChanged += ui.ShowDownloadProgress;
        downloader.OnDownloadComplete += ui.HandleDownloadComplete;

        // Logger 只关心下载完成事件
        downloader.OnDownloadComplete += logger.LogDownloadComplete;

        // 开始下载，之后 Downloader 会在合适的时机回调所有已注册的方法
        downloader.StartDownload("http://example.com/file.zip");
        
        // 如果不再需要通知，可以取消订阅
        downloader.OnDownloadComplete -= logger.LogDownloadComplete;
    }
}
```

**事件的优势：**

- **多播 (Multicast)**：一个事件可以有多个订阅者，触发时会依次调用所有订阅的方法。
    
- **封装性**：在类的外部，你只能对事件进行 += (注册) 和 -= (注销) 操作，不能直接 = 赋值或触发事件，这保证了只有类的内部才能决定何时触发事件。