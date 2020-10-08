重新指定解决方案路径的过程中，如果.sln文件和.csproj文件更改了路径，需要用VSCode(或者其他编辑器)打开.sln文件，修改 project 对应的路径；然后打开.csproj文件，修改Nuget引用的库所对应的路径，例如：
```xml
<Reference Include="PropertyChanged, Version=3.2.9.0, Culture=neutral, PublicKeyToken=ee3ee20bcf148ddd, processorArchitecture=MSIL">
	<HintPath>..\packages\PropertyChanged.Fody.3.2.9\lib\net40\PropertyChanged.dll</HintPath>
</Reference>
```
修改hintpath，并修改错误检查中的库引用路径，例如
```xml
<Error Condition="!Exists('..\packages\PropertyChanged.Fody.3.2.9\build\PropertyChanged.Fody.props')"Text="$([System.String]::Format('$(ErrorText)', 'packages\PropertyChanged.Fody.3.2.9\build\PropertyChanged.Fody.props'))" />
<Error Condition="!Exists('..\packages\Fody.6.2.6\build\Fody.targets')" Text="$([System.String]::Format('$(ErrorText)','packages\Fody.6.2.6\build\Fody.targets'))" />
```