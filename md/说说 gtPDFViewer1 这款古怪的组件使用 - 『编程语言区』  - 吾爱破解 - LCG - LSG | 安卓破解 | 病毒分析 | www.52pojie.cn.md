> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/forum.php?mod=viewthread&tid=1453275&extra=page%3D1%26filter%3Dauthor%26orderby%3Ddateline)  ![](https://avatar.52pojie.cn/data/avatar/000/08/22/58_avatar_middle.jpg) 冥界 3 大法王 下面在使用过程中发现了一个问题，官方给的实例是通过 action 调用打开 PDF 文件的  
但是呢？我要求在使用文件浏览器时，双击某文件打开 (传递一个路径过去）就用不了啦~~  
于是查看自带的 CHM 竟然没有  
发现最有可能的是：gtPDFDocument1.LoadFromFile('你的文件路径');  
使用后发现不报错，就是 pdf 没有显示出来。。。  
组件使用时，必须使用 gtPDFViewer1 和 gtPDFDocument1  
案例里也没有找到答案，最后无奈的看了下原官方英文的 FAQ  
通过几轮测试发现出问题的代码在下面的注释代码中。  
[Delphi] _纯文本查看_ _复制代码_

```
procedure TMain_Form1.FileListBox1DblClick(Sender: TObject);
var
  i: Integer;
  TempString: string;
begin
  for i := 0 to FileListBox1.items.count - 1 do
  begin
    if FileListBox1.Selected[i] = True then
    begin
//      ShowMessage(FileListBox1.items[i]);                                 //得到文件名（含扩展名）
      ShowMessage(FileListBox1.Directory + (FileListBox1.items[i]));       //得到文件完整路径
      TempString := FileListBox1.Directory + (FileListBox1.items[i]);
 
      gtPDFViewer1.PDFDocument := gtPDFDocument1;       //这句不加上不显示文档内容
      gtPDFDocument1.LoadFromFile(TempString);
      gtPDFViewer1.Active := True;                                   //这句不加上同样也不显示文档内容
//      ShellExecute(Handle, 'Open', PChar('wordpad.exe'), PChar(TempString), nil, SW_SHOWNORMAL);
    end;
  end;
end;
```

![](https://attach.52pojie.cn/forum/202106/04/115424auua3ozsiaxya15y.png)

**image.png** _(895.9 KB, 下载次数: 0)_

[下载附件](forum.php?mod=attachment&aid=MjI5OTI3NnxlOWZmZmIwMnwxNjIyODIwNzk5fDEwNzQxNTd8MTQ1MzI3NQ%3D%3D&nothumb=yes)

2021-6-4 11:54 上传

![](https://www.52pojie.cn/uc_server/images/noavatar_middle.gif)alongzhenggang  多借   分享  这个 没有见过