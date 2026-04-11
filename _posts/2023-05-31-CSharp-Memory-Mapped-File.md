# C# 使用内存映射像处理内存一样去快速处理文件

在 .NET Core 中，`System.IO.MemoryMappedFiles.MemoryMappedFile` 类提供了对内存映射文件的支持。通过将文件映射到内存，你可以在应用程序中直接访问文件的内容，而不需要显式地进行文件的读取和写入操作。

内存映射文件允许你将文件的特定区域映射到内存中的一个或多个 `MemoryMappedViewAccessor` 对象。同时，你可以使用 `MemoryMappedViewAccessor.SafeMemoryMappedViewHandle` 进行不安全操作，并将其映射到一个数组，例如 `int[]` 或者你自定义的 `Point[]`。

通过这种方式，你可以直接访问内存映射的数据，并对其进行更改，这将自动反映在映射文件中。

## 示例：将内存映射文件映射到自定义结构体数组

```csharp
using System;
using System.IO;
using System.IO.MemoryMappedFiles;
using System.Runtime.InteropServices;

class Program
{
    struct Point
    {
        private int x;
        private int y;

        public Point(int x, int y)
        {
            this.x = x;
            this.y = y;
        }

        public int X => x;
        public int Y => y;
    }

    static void Main()
    {
        int arrayLength = 10;
        int arraySize;

        unsafe
        {
            arraySize = sizeof(Point) * arrayLength;
        }

        using (var mmf = MemoryMappedFile.CreateFromFile("data.bin", FileMode.OpenOrCreate, "myData", arraySize))
        {
            using (var accessor = mmf.CreateViewAccessor())
            {
                unsafe
                {
                    byte* ptr = null;

                    try
                    {
                        accessor.SafeMemoryMappedViewHandle.AcquirePointer(ref ptr);

                        // 将内存映射的指针转换为 Point 数组
                        Point* pointArrayPtr = (Point*)ptr;

                        // 在 Point 数组中进行写操作
                        for (int i = 0; i < arrayLength; i++)
                        {
                            pointArrayPtr[i] = new Point(i, i);
                        }

                        // 从 Point 数组中读取数据
                        for (int i = 0; i < arrayLength; i++)
                        {
                            Point value = pointArrayPtr[i];
                            Console.WriteLine($"x={value.X}, y={value.Y}");
                        }
                    }
                    finally
                    {
                        if (ptr != null)
                        {
                            accessor.SafeMemoryMappedViewHandle.ReleasePointer();
                        }
                    }
                }
            }
        }
    }
}
```

## 说明

在这个示例中：

1. 首先创建一个内存映射文件，并指定了与 `Point[]` 数组相同大小的映射区域。
2. 使用 `CreateViewAccessor` 方法创建一个 `MemoryMappedViewAccessor` 对象。
3. 在 `unsafe` 上下文中，通过调用 `SafeMemoryMappedViewHandle.AcquirePointer` 方法来获取内存映射的指针，然后将指针转换为 `Point*` 类型，从而将其视为 `Point[]` 数组。
4. 通过直接使用指针进行读写操作，你可以直接修改内存映射的数据，这将自动反映在映射文件中。
5. 最后，使用 `SafeMemoryMappedViewHandle.ReleasePointer` 方法释放指针。

使用不安全代码需要谨慎，并且需要遵循安全性和正确性的最佳实践。确保你正确管理内存和指针，并遵循 C# 中的不安全代码规范。
