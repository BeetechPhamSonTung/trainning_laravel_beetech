#Upload files trong Laravel
## Tạo view
-Đầu tiên mình cần tạo ra một view form có một input file và một input submit để thực hiện upload. (name: **DemoUpload.blade.php**)
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Demo Upload File - Toidicode.com</title>
    <meta name="author" content="ThanhTai">
</head>
<body>
<form action="{{ url('file') }}" enctype="multipart/form-data" method="POST">
    @csrf
    <input type="file" name="file" >
    <br/>
    <input type="submit" value="upload">
</form>
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
</body>
</html>
```
Chúng ta phải thêm `enctype="multipart/form-data"` để trình duyệt hiểu và mã hóa dữ liệu thành nhiều phần và `@csrf` để tránh bị tấn công `CSRF ( Cross Site Request Forgery)`

Ở đây chúng ta dùng biến `$errors` để in ra các lỗi mà dùng validate ở controller

## Tạo controller 
Tiếp theo chúng ta cần tạo một controller có tên: **FileController.php**
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class FileController extends Controller
{
    public function index()
    {
        return view('DemoUpload');
    }

    public function doUpload(Request $request)
    {
        if ($request->hasFile('file')) {
            $file = $request->file;
            $request->validate([
                'file' => [
                    'max:512',
                    'mimes:txt',
                ]
            ]);
            //Lấy Tên files
            echo 'Tên Files: ' . $file->getClientOriginalName();
            echo '<br/>';

            //Lấy Đuôi File
            echo 'Đuôi file: ' . $file->getClientOriginalExtension();
            echo '<br/>';

            //Lấy đường dẫn tạm thời của file
            echo 'Đường dẫn tạm: ' . $file->getRealPath();
            echo '<br/>';

            //Lấy kích cỡ của file đơn vị tính theo bytes
            echo 'Kích cỡ file: ' . $file->getSize();
            echo '<br/>';

            //Lấy kiểu file
            echo 'Kiểu files: ' . $file->getMimeType();

            $file->move('F:\SonTung - Setup\xampp\htdocs\blog\uploads', 'testUploadFile1');
        }
    }
}

```
* Ở đây chúng ta có thể giới hạn dung lượng file người dùng upload lên bằng `validate` như `max:512` là file có dung lượng tối đa là `512kb`, `mimes` là để kiểm soát loại file người dùng up như là `txt`...
* Để upload files với Request thì Laravel có cung cấp cho chúng ta hàm `move()` sử dụng với 2 thông số truyền vào:
```
move($Location,$filesName)
```
Trong đó:

$Location: Là thư mục chứa file upload lên Sever.
$filesName: Là tên mới của file.

#Session trong Laravel
##I.Giới thiệu (Introduction)
Laravel cung cấp cho chúng ta các khuôn mẫu để có thể tương tác với session. Trong Laravel có nhiều driver để lưu trữ session. Các bạn có thể mở file `config/session.php` lên và sẽ thấy các driver mà Laravel cho phép ta sử dụng:
```
/*
    |--------------------------------------------------------------------------
    | Default Session Driver
    |--------------------------------------------------------------------------
    |
    | This option controls the default session "driver" that will be used on
    | requests. By default, we will use the lightweight native driver but
    | you may specify any of the other wonderful drivers provided here.
    |
    | Supported: "file", "cookie", "database", "apc",
    |            "memcached", "redis", "dynamodb", "array"
    |
    */
```
###1. Cấu hình(Configuration)
File cấu hình session trong ứng dụng Laravel là `config/session.php`. Dưới đây là một số driver hay sử dụng trong Laravel:

* `file`: các session sẽ được lưu trữ trong file tại thư mục `storage/framework/sessions`. Bạn có thể thay đổi thư mục lưu trữ ở bên dưới:
```
'files' => storage_path('framework/sessions'),
```
* `cookie`: session sẽ được lưu trữ trong cookie an toàn, được mã hóa.
* `database`: các session sẽ được lưu trữ trong database.
* `memcached`/ `redis`: session sẽ hoạt động tốt hơn nếu lưu vào các driver này, dựa trên cache.
* `array`: các session sẽ được lưu trữ trong mảng PHP và không được duy trì.
> Mặc định thì Laravel sử dụng driver `file`, nhưng trong thực tế với các ứng dụng lớn thường hay sử dụng các package như `redis` hay `memcached` để tối ưu hiệu năng. Còn đối với driver `array` thường được sử dụng trong quá trình test.

Để chọn driver session cho ứng dụng, bạn chỉ cần thay đổi tại file `.env`.
```
SESSION_DRIVER=file
```
Ngoài ra còn có các thiết lập mà bạn cần quan tâm trong session. Chẳng hạn như thiết lập thời gian sống của session thông qua `SESSION_LIFETIME` ở file `.env` (được tính bằng phút).
```
SESSION_LIFETIME=120
```
Bạn có thể thiết lập xóa bỏ các session sau khi trình duyệt kết thúc bằng cách thay đổi giá trị bên dưới thành `true`:
```
'expire_on_close' => false,
```
Nếu bạn muốn tăng tính bảo mật khi lưu trữ các session, đặc biệt với driver `file`, bạn có thể thiết lập giá trị `true` cho cấu hình bên dưới:
```
'encrypt' => false,
```
Với cấu hình này, các session trước khi lưu trữ sẽ được mã hóa.
###2.Điều kiện tiên quyết của driver (Driver prerequisite)
####Database