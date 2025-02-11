在跨平台开发dotnet项目时，可能会写一些特定平台的脚本供项目在编译前/后调用，此时可以在项目中增加系统判断条件，来调用不同的脚本

```xml
<Target Name="CopySQLiteDll_win" AfterTargets="Build"
          Condition=" '$(OS)' == 'Windows_NT' ">
    <ItemGroup>
      <DataReaders Include="$(TargetDir)runtimes\win-x64\native\e_sqlite3.dll"/>
    </ItemGroup>

    <Copy
      SourceFiles="@(DataReaders)"
      DestinationFolder="$(ProjectDir)..\ReaderAssemblies\"
      SkipUnchangedFiles="true" />
  </Target>

  <Target Name="CopySQLiteDll_macos" AfterTargets="Build"
          Condition="'$([System.Runtime.InteropServices.RuntimeInformation]::IsOSPlatform($([System.Runtime.InteropServices.OSPlatform]::OSX)))'">
    <ItemGroup>
      <DataReaders Include="$(TargetDir)runtimes\osx-x64\native\libe_sqlite3.dylib"/>
    </ItemGroup>

    <Copy
      SourceFiles="@(DataReaders)"
      DestinationFolder="$(ProjectDir)..\ReaderAssemblies\"
      SkipUnchangedFiles="true" />
  </Target>

  <Target Name="CopySQLiteDll_linux" AfterTargets="Build"
          Condition="'$([System.Runtime.InteropServices.RuntimeInformation]::IsOSPlatform($([System.Runtime.InteropServices.OSPlatform]::Linux)))'">
    <ItemGroup>
      <DataReaders Include="$(TargetDir)runtimes\linux-x64\native\libe_sqlite3.so"/>
    </ItemGroup>

    <Copy
      SourceFiles="@(DataReaders)"
      DestinationFolder="$(ProjectDir)..\ReaderAssemblies\"
      SkipUnchangedFiles="true" />
  </Target>
```

**其中**，Windows平台默认定义了`$(OS) == 'Windows_NT'` 的宏，如果仅判断windows平台和非windows平台，可以直接使用这个宏来实现