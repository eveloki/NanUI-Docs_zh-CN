文件下载功能只需要实现CEF自带的接口即可

请注意，这是初步文件，可能会被更改

## ChromiumWebBrowser.DownloadHandler 

直接按照示例代码实现即可

```html
   DownloadHandler.OnBeforeDownload += (sender, args) =>
            {
                string filename = args.SuggestedName;
                SaveFileDialog fileDialog = new SaveFileDialog()
                {
                    AddExtension = true,
                    Filter = "所有文件(*.*)|*.*",
                    OverwritePrompt = true,
                    RestoreDirectory = true,
                    FileName = filename

                };
                args.Callback.Continue(fileDialog.FileName, true);
                var js = $"alert( '下载成功' )";
                ExecuteJavascript(js);

            };
```

需要注意的是,本接口为chromiumfx自带接口，有任何BUG请自行至CEF官网查找方法

