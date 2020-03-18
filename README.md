# Component library with .NET IL Linker configuration

This sample shows how you can embed a .NET IL Linker configuration file into a library that specifies what assemblies, types, and members should be preserved by the .NET IL Linker.

## Run the sample

To build and run the sample locally, install the [.NET Core SDK](https://dotnet.microsoft.com/download) version 3.1.102 or later.

Then run `dotnet run` from the *BlazorApp1* directory.

Try entering various expressions to filter to the list of people:

![Screen shot](https://user-images.githubusercontent.com/1874516/77014109-c56eb380-692e-11ea-853d-8b48f6812807.png)

## Handling reflection with the .NET IL Linker

The .NET IL Linker statically analyzes .NET assemblies to determine what code is being used and can remove any unused code. However, if the code uses .NET reflection there is no way for the linker to know what additional code is being used via reflection. Instead, you need to manually instruct the linker what additional code should be preserved. This can be done using an XML linker config file.

The ComponentLibWithXmlLinkerConfig project is a [Blazor component library](https://docs.microsoft.com/aspnet/core/blazor/class-libraries) that uses [System.Linq.Dynamic.Core](https://github.com/StefH/System.Linq.Dynamic.Core) to run dynamic LINQ queries over a collection. System.Linq.Dyanmic.Core works using reflection. The component library is then used from a Blazor WebAssembly app. The app has been setup to run the linker on every build by setting `<BlazorWebAssemblyEnableLinking>true</BlazorWebAssemblyEnableLinking>`. Without any additional linker config the app fails at runtime with the following exception because methods called by System.Linq.Dynamic.Core via reflection on the `System.Linq.Queryable` type have been removed by the linker:

```
Unhandled exception rendering component:
System.TypeInitializationException: The type initializer for 'System.Linq.Dynamic.Core.DynamicQueryableExtensions' threw an exception. ---> System.Exception: Specific method not found: All ---> System.InvalidOperationException: Sequence contains no matching element
  at System.Linq.Enumerable.Single[TSource] (System.Collections.Generic.IEnumerable`1[T] source, System.Func`2[T,TResult] predicate) <0x2146c98 + 0x000f0> in <filename unknown>:0 
  at System.Linq.Dynamic.Core.DynamicQueryableExtensions.GetMethod (System.String name, System.Int32 parameterCount, System.Func`2[T,TResult] predicate) <0x2146898 + 0x0004c> in <filename unknown>:0 
   --- End of inner exception stack trace ---
  at System.Linq.Dynamic.Core.DynamicQueryableExtensions.GetMethod (System.String name, System.Int32 parameterCount, System.Func`2[T,TResult] predicate) <0x2146898 + 0x00078> in <filename unknown>:0 
  at System.Linq.Dynamic.Core.DynamicQueryableExtensions..cctor () <0x213ca70 + 0x00024> in <filename unknown>:0 
   --- End of inner exception stack trace ---
  at ComponentLibWithXmlLinkerConfig.Component1.OnClick () [0x00001] in C:\users\user\Documents\GitHub\danroth27\ComponentLibWithXmlLinkerConfig\ComponentLibWithXmlLinkerConfig\Component1.razor:33 
  at (wrapper delegate-invoke) <Module>.invoke_void()
  at Microsoft.AspNetCore.Components.EventCallbackWorkItem.InvokeAsync[T] (System.MulticastDelegate delegate, T arg) <0x20d8fd0 + 0x00062> in <filename unknown>:0 
  at Microsoft.AspNetCore.Components.EventCallbackWorkItem.InvokeAsync (System.Object arg) <0x20d8f10 + 0x0000a> in <filename unknown>:0 
  at Microsoft.AspNetCore.Components.ComponentBase.Microsoft.AspNetCore.Components.IHandleEvent.HandleEventAsync (Microsoft.AspNetCore.Components.EventCallbackWorkItem callback, System.Object arg) <0x20d8e78 + 0x0000a> in <filename unknown>:0 
  at Microsoft.AspNetCore.Components.EventCallback.InvokeAsync (System.Object arg) <0x20d8a00 + 0x00040> in <filename unknown>:0 
  at Microsoft.AspNetCore.Components.RenderTree.Renderer.DispatchEventAsync (System.UInt64 eventHandlerId, Microsoft.AspNetCore.Components.RenderTree.EventFieldInfo fieldInfo, System.EventArgs eventArgs) <0x20c9058 + 0x000a8> in <filename unknown>:0 
```

To preserve the `System.Linq.Queryable` type and its members, the following XML linker config file is added to the ComponentLibWithXmlLinkerConfig project:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<!-- IL linker format: https://github.com/mono/linker#link-xml-file-examples -->
<linker>
  <assembly fullname="System.Core">
    <type fullname="System.Linq.Queryable" preserve="all" />
  </assembly>
</linker>
```

This file is then embedded into the library by adding the following to the project file:

```xml
<ItemGroup>
  <EmbeddedResource Include="ComponentLibWithXmlLinkerConfig.xml">
    <LogicalName>$(MSBuildProjectName).xml</LogicalName>
  </EmbeddedResource>
</ItemGroup>
```

> Note that the name of the embedded XML linker config file must match the name of the assembly. By default the file name will be prefixed with the assembly namespace, so the correct name must be explicitly specified as the `LogicalName` for the embedded file.

## Additional resources

- [Configure the Linker for Blazor apps](https://docs.microsoft.com/aspnet/core/host-and-deploy/blazor/configure-linker)
- [XML linker config file format](https://github.com/mono/linker#link-xml-file-examples)
- [Custom Linker Configuration in Xamarin apps](https://docs.microsoft.com/en-us/xamarin/cross-platform/deploy-test/linker)
