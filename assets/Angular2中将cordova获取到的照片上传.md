最近在用Angular2+ionic2写一个饭否的第三方客户端，需要将拍照或者手机相册里得到的照片，通过饭否的Api上传，折腾了一个星期后，终于找到了解决方案。

### 需要传何种格式的照片数据

最常见的做法是通过前台一个<input type="file">向后台传一个formData，但是cordova调取手机相册或者相机得到的数据是base64或者一个文件路径，饭否的Api是没办法接收base64格式的photo数据的，另外如果照片数据过大，显然也不适合使用base64。虽然我也可以通过添加一个file类型的input去获取照片，但是这样的话，手机上点击后出现的选择菜单我没办法控制。所以，如何把一个图片路径或者base64转成formData才是关键。

### 寻找解决方案

首先我尝试了能否把图片路径进行转化，我找到了stackoverflow上一年前的一个问题：[Append image file to form data](http://stackoverflow.com/questions/28843883/append-image-file-to-form-data-cordova-angular) ，里面提到用 [cordova-plugin-file-transfer](https://github.com/apache/cordova-plugin-file-transfer)这个插件。

```
var fd = new FormData();
     window.resolveLocalFileSystemURL(attachment.img, function(fileEntry) {
         fileEntry.file(function(file) {
             var reader = new FileReader();
                 reader.onloadend = function(e) {
                      var imgBlob = new Blob([ this.result ], { type: "image/jpeg" } );
                      fd.append('attachment', imgBlob);
                      fd.append('uuid', attachment.uuid);
                      fd.append('userRoleId', 12345);
                      console.log(fd);
                      //post form call here
                 };
                 reader.readAsArrayBuffer(file);

         }, function(e){$scope.errorHandler(e)});
    }, function(e){$scope.errorHandler(e)});
```



我做了一下尝试，但是不知道是什么原因始终触发不了reader.onloadend这个函数，于是我又在[cordova-plugin-file-transfer](https://github.com/apache/cordova-plugin-file-transfer)找到了另一个办法，但这个方法传到后台的数据，后台解析不到formData,所以这个办法也宣告失败。

```
var win = function (r) {
    console.log("Code = " + r.responseCode);
    console.log("Response = " + r.response);
    console.log("Sent = " + r.bytesSent);
}

var fail = function (error) {
    alert("An error has occurred: Code = " + error.code);
    console.log("upload error source " + error.source);
    console.log("upload error target " + error.target);
}

var options = new FileUploadOptions();
options.fileKey = "file";
options.fileName = fileURL.substr(fileURL.lastIndexOf('/') + 1);
options.mimeType = "text/plain";

var params = {};
params.value1 = "test";
params.value2 = "param";

options.params = params;

var ft = new FileTransfer();
ft.upload(fileURL, encodeURI("http://some.server.com/upload.php"), win, fail, options);
```



### base64转formData

前两种方法都失败了，只能寄希望于这个办法可以成功，Google了一下，果然有解决方案:[Convert Data URI to File then append to FormData](http://stackoverflow.com/questions/4998908/convert-data-uri-to-file-then-append-to-formdata) ,而且已经是5年前的答案了，惭愧。这个方法通过将base64转成Blob，然后formData将转换过来的Blob append进去，这个Blob就是饭否api所需要接收的photo参数。不过这个方法依然不支持图片路径转换。

```
function dataURItoBlob(dataURI) {
    // convert base64/URLEncoded data component to raw binary data held in a string
    var byteString;
    if (dataURI.split(',')[0].indexOf('base64') >= 0)
        byteString = atob(dataURI.split(',')[1]);
    else
        byteString = unescape(dataURI.split(',')[1]);

    // separate out the mime component
    var mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0];

    // write the bytes of the string to a typed array
    var ia = new Uint8Array(byteString.length);
    for (var i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
    }

    return new Blob([ia], {type:mimeString});
}
```

改好之后重新打包测试，上传照片成功！

由于饭否对上传图片的大小有限制，所以控制在2M左右的大小比较合适。通过测试，在进行Camera插件相关选项设置时，quality:80并且destinationType:'jpeg',比较合适。如果要生成png图片的话虽然照片质量要好一点，但是照片大小很容易超出限制。

ps，如果想选择相册照片进行上传时，无需安装Image Picker这个插件，只需要在Camera的选项配置中修改sourceType:PHOTOLIBRARY就可以了。

pps：后台接收formData的方法可参考这里：[Posting images to twitter in Node.js using Oauth](http://stackoverflow.com/questions/12921371/posting-images-to-twitter-in-node-js-using-oauth) 

