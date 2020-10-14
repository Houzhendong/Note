# 方法1

```c#
private static BitmapImage LoadImage(byte[] imageData)
    {
        if (imageData == null || imageData.Length == 0) return null;
        var image = new BitmapImage();
        using (var mem = new MemoryStream(imageData))
        {
            mem.Position = 0;
            image.BeginInit();
            image.CreateOptions = BitmapCreateOptions.PreservePixelFormat;
            image.CacheOption = BitmapCacheOption.OnLoad;
            image.UriSource = null;
            image.StreamSource = mem;
            image.EndInit();
        }
        image.Freeze();
        return image;
    }
```

# 方法2

```c#
BitmapSource bitmapSource = BitmapSource.Create(2, 2, 300, 300,PixelFormats.Indexed8,BitmapPalettes.Gray256, 		byteArrayIn, 2);
Image.Source = bitmapSource;
```

```xaml
<Image RenderOptions.BitmapScalingMode="NearestNeighbor" RenderOptions.EdgeMode="Aliased" x:Name="Image"></Image>  
```

