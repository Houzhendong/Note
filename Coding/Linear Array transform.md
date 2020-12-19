# 一维数组表示的二维数组中可能遇到的操作

1. 转置

   ```c#
   for(x = 0; x < height; x++)
   {
       for(y = 0; y < width; y++)
           dstArr[y * height + x] = srcArr[x * width + y];
   }
   ```

   

2. 旋转

   2.1 以右上角为轴逆时针旋转90°

   ```c#
   for(x = 0; x < height; x++)
   {
       for(y = width; y > 0; y--)
       {
           dstArray[(width - j) * height + x] = srcArray[x * width + y-1];
       }
   }
   ```

   