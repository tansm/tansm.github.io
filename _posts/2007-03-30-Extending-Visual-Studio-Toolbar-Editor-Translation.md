# 使用C# 扩展VS的带工具栏的编辑器（翻译）

> 原文：https://www.cnblogs.com/tansm/archive/2007/03/30/693637.html

#### 介绍

这个例子展示了如何使用一个包(Package）来扩展Visual Studio，使之支持一个特定文件类型的编辑器并附带工具箱支持，在这个例子中我们将实现一个 .tbx文件的编辑器以及一个可用于此文档的工具箱项目。

- 提供一个编辑器工厂类；
- 文档的序列化和反序列化；
- 工具箱支持，即从工具箱中拖动一个文本到文档中；
- 支持源代码控制和只读文件支持。

![](/images/2007-03-30-Extending-Visual-Studio-Toolbar-Editor-Translation-1.jpg)

#### 入门

这个例子实现了.tbx文件的编辑器功能，其内部实际上只是简单的使用了RickTextBox控件来编辑文档。这个例子主要还是用来展示编辑器如何与Vistual Studio工具箱的交互。

这个编辑器使用了SVsToolbox服务并实现了IVsToolboxUser接口，以支持于工具箱交互，实现操控工具箱和支持拖动。

在这个编辑器中，支持LOGVIEWID_Designer逻辑视图，需要在注册表中登记入口信息以描述这个包支持此后缀。

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\8.0\Editors\ {93fa4dc3-61ec-47af-b0ba-50cad3caf049}] "DisplayName"="#106" "Package"="{68a4ede6-8f63-44f2-803e-65f770e709e1}"

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\8.0\Editors\ {93fa4dc3-61ec-47af-b0ba-50cad3caf049}\Extensions] "addin"=dword:00000032

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\8.0\Editors\ {93fa4dc3-61ec-47af-b0ba-50cad3caf049}\LogicalViews] "{7651a702-06e5-11d1-8ebd-00a0c90f26ea}"=""

其中

{93fa4dc3-61ec-47af-b0ba-50cad3caf049}是指EditorFactory类型的COM GUID；

{68a4ede6-8f63-44f2-803e-65f770e709e1}指EditorPackage的COM GUID；

{7651a702-06e5-11d1-8ebd-00a0c90f26ea}指的是 LOGVIEWID_Designer的值。

这个Example.EditorWithToolbox的例子包含一个IntergrationTests的目录，这个目录中的测试用例要求你的机器必须安装VsIdeTestHost.msi ，这个安装包将创建 VisualStudioTeamSystemIntegration\Test Tool Extensibility\VsIdeHostAdapter文件夹。

项目文件

**AssemblyInfo.cs** | 包含组装件的信息
**ClassDiagram.cd** | 工程的类描述图
**EditorFactory.cs** | 实现了IVsEditorFactory接口以创建编辑器的视图对象
**EditorPane.cs** | 实现了EditorPane类, 用来容纳编辑器 (RichTextBox控件) 并响应编辑器的Command命令
**EditorControl.cs** | 派生自RichTextBox.的控件，用来编辑文本
**GuidList.cs** | 包含了所有的GUID定义, 包括package的GUID和所有Command的GUID.
**Resources.resx** | 项目的资源文件. 这些定义将被SampleDocViewEditor.vsdir使用.
**EditorPackage.cs** | 包含了包的定义，其关于编辑器的attributes定义能够自动注册到注册表中，他也实现了创建一个EditorFactory实例，并通知IDE (调用 IVsRegisterEditor::RegisterEditor).
**Templates\tbx.tbx** | EditorWithToolbox.vsdir 将使用这个 "tbx"作为例子文件.
**Templates\EditorWithToolbox.vsdir** | 在Visual Studio的新建对话框中，提供模板。

© Microsoft Corporation. All rights reserved.
