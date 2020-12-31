# C#调用C++

1. C++ char* to C# string

   C++ :  `foo(char* tmp)` 

   C# : `public static extern foo([MarshalAs(UnmanagedType.LPStr)] StringBuilder tmp)`

   