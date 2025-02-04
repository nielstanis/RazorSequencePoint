# Sequence Point 

To flag XSS in compiled Razor view we rely on the `MethodDebugInformation` and sequence points which are part of the PDB created at compile time as well.
Turns out that there is a difference which has been pinpointed down to a revision of .NET8 SDK, occuring between v8.0.20x and v8.0.30x versions.

The XSS can be found here: [Index.cshtml](src/Pages/Index.cshtml#L8) at line number 8.

- The `src` folder contains the source of project, the `global.json` files are used to switch SDK.
- The `compiled` folder contains zip archive with `app.dll` and `app.pdb` for the version `8.0.203` and `8.0.302`, which where ran on MacOSX. 

## 8.0.203

In the generated Razor View for `Index.cshtml` in `8.0.203` part of `app.dll` we find:

```csharp
#line default
#line hidden
#nullable disable
            WriteLiteral("\r\n<div class=\"text-center\">\r\n    <h1 class=\"display-4\">Welcome ");
#nullable restore
#line (8,36)-(8,66) 6 "/Users/nelson/github/RazorSequencePoint/src/Pages/Index.cshtml"
Write(Html.Raw(this.Model.DataFromQ));

#line default
#line hidden
#nullable disable
            WriteLiteral("<!--XSS found here--></h1>\r\n    <p>Learn about <a href=\"https://learn.microsoft.com/aspnet/core\">building Web apps with ASP.NET Core</a>.</p>\r\n</div>\r\n");
        }
```

Inside of `MethodDebugInformation` we can find entry `31000076` that is pointing out to `Index.cshtml` and has `SP(IL_0034 17:8,36-8,67)` the location of the `@Html.Raw`. 

## 8.0.302

In the generated Razor View for `Index.cshtml` in `8.0.302` part of `app.dll` we find:

```csharp
#line default
#line hidden
#nullable disable

            WriteLiteral("\r\n<div class=\"text-center\">\r\n    <h1 class=\"display-4\">Welcome ");
            Write(
#nullable restore
#line (8,36)-(8,66) "/Users/nelson/github/RazorSequencePoint/src/Pages/Index.cshtml"
Html.Raw(this.Model.DataFromQ)

#line default
#line hidden
#nullable disable
            );
            WriteLiteral("<!--XSS found here--></h1>\r\n    <p>Learn about <a href=\"https://learn.microsoft.com/aspnet/core\">building Web apps with ASP.NET Core</a>.</p>\r\n</div>\r\n");
        }
```

No entry inside of `MethodDebugInformation` points out to the above location, also the generated view is a slightly bit different because the `#line` directive does not have an offset, probably because `Write(` is done before the directive is defined.  