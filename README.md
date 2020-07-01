# pdfjs

iOS加载pdf,显示电子签章
删减版是删除了图片,和一下无用资源
完整版的, 可根据需求自行修改

具体使用也可见简书
https://www.jianshu.com/p/b8d28d0d8408

项目中需要显示订单合同PDF文件。正常情况下，加载pdf文件直接通过UIWebView或者WKWebView就可以。不过实际情况中，PDF文件WKWebview可以在iOS12之后可以正常展示电子签章,但是在12之前无法正常展示.
方案: 使用pdf.js加载

###pdf.js是火狐浏览器的开源项目，github地址为：https://github.com/mozilla/pdf.js 。

1.首先把pdf.js clone到本地：
```
  git clone git://github.com/mozilla/pdf.js.git
```
2.pdf.js 构建基于nodejs,因此需要安装nodejs：

npm install -g gulp-cli
npm install

3.修改工具的源码,在 目录src/core/annotation.js文件中

    // Hide unsupported Widget signatures. 注释掉if中的内容即可显示电子签章
    if (data.fieldType === 'Sig') {
      warn('unimplemented annotation type: Widget signature');
      this.setFlags(AnnotationFlag.HIDDEN);//就是将这句话注释带就可以了
    }

4.进入pdf.js所在的文件，输入gulp generic或$ gulp minified进行构建：

cd pdf.js
#generic和minified取其中一种即可。
gulp generic
gulp minified

构建完成后,会在build文件夹下生成相关文件,
将build\generic或build\ minified下的文件导入到xcode中。我在项目中用的是minified。
图片.png

4.将minified文件使用xcode加入到工程中, 接入代码
文件链接可执行下载(完整版和删减版)直接使用, 或者修改
https://github.com/kbshare/pdfjs/tree/master

 NSData *data = [[NSData alloc]initWithBase64EncodedString:byteStr options:NSDataBase64DecodingIgnoreUnknownCharacters];//byteStr服务端返回的pdf字符串
                NSString *filePath = [NSString docCompFolderWithFileName:@"policy.pdf"];
                
                //移除文件
                [[NSFileManager defaultManager] removeItemAtURL:[NSURL URLWithString:filePath] error:nil];
                
                BOOL isSuccess = [data writeToFile:filePath atomically:YES];
                if (isSuccess) {
                    ayh_dispatch_after(1, ^{
                        [self loadPDFFile:filePath];
                        NSLog(@"成功")
                    });
                }else{
                    NSLog(@"不成功");
                }

- (void)loadPDFFile:(NSString*)filePath {
    NSString *viwerPath = [[NSBundle mainBundle] pathForResource:@"viewer" ofType:@"html" inDirectory:@"minified/web"];
    NSString *urlStr = [NSString stringWithFormat:@"%@?file=%@#page=1",viwerPath,filePath];
    urlStr = [urlStr stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:urlStr]];
    [self.staticWeb loadRequest:request];
}

即可加载成功
问题

1.生成的minified文件夹太大, 导入到项目中十几兆, 影响安装包大小
优化

可以将不必要的文件删除, 都是一些无用文件, 和本地化的文件,及图片资源

图片.png

2.pdf.js加载文件,容器的样式太复杂, 一般都用不到,可以对viewer.html, 和viewer.css进行修改, 我这里只剩下了显示页数, 和放大缩小功能, 你可以根据具体需求进行修改, viewer.css写的很好,很方便修改样式
