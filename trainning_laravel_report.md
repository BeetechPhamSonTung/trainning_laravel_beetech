# Upload files trong Laravel
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

# Session trong Laravel

## I.Giới thiệu (Introduction)
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
### 1. Cấu hình(Configuration)

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
### 2.Điều kiện tiên quyết của driver (Driver prerequisite)

#### Database

Khi sử dụng driver `database` cho session, bạn cần phải tạo một table để lưu trữ các session đó. Laravel cung cấp cho chúng ta chuỗi lệnh Artisan để có thể khởi tạo nhanh table `sessions`.
> php artisan session:table
>
> php artisan migrate

Trước khi thực hiện hai lệnh trên, ta cần phải config database cho ứng dụng. Các bạn mở file `.env` là thiết lập các thông số database.

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=root
DB_PASSWORD=
```
Nếu như bạn không thích tên table `sessions` mặc định, bạn có thể cấu hình nó tại file `config/session.php` trước khi chạy 2 dòng lệnh trên.
```
'table' => 'sessions',
```
## II. Sử dụng session (Using the session)

Có hai hướng chính để làm việc với session trong Laravel framework:

1.Thông qua lớp khởi tạo `Illuminate\Http\Request`.

2.Sử dụng global helper `session`.

### 1. Lưu trữ dữ liệu (Storing data)

Với method `put` từ object `Illuminate\Http\Request`
```php
$request->session()->put('key','value');
```
Với global helper `session`
```php
session(['key'=>'value']);
```
Bạn có thể push 1 item session vào mảng của nó bằng cách sử dụng dấu `.` để tham chiếu trong method `push`:
```php
$request->session()->push('user.username','Phạm Sơn Tùng');
```
hoặc thông qua global helper `session`:
```php
session(['user.username'=>'Phạm Sơn Tùng']);
```
Bây giờ chúng ta hãy thử kiểm tra xem session có thực sự được lưu trữ hay không?

Hãy thử đăng ký một route ở `routes/web.php` để thực hiện store session như bên dưới:
```php
Route::get('/session', function() {
    session(['name' => 'Lê Chí Huy']);
});
```
hoặc 
```php
use Illuminate\Http\Request;

Route::get('/session', function (Request $requets) {
    $request->session()->put('name', 'Lê Chí Huy');
});
```
Nếu bạn đang sử dụng driver `file`, bạn hãy mở các file trong thư mục `storage/framework/sessions`, bạn sẽ tìm ra được session mà bạn vừa lưu trữ.
```
a:4:{s:6:"_token";s:40:"C56DUX9Tg07nbBOcPrwDqqf0JjEK8yBJk7PSeAIC";s:9:"_previous";
a:1:{s:3:"url";s:29:"http://127.0.0.1:8000/session";}s:6:"_flash";a:2:{s:3:"old";
a:0:{}s:3:"new";a:0:{}}s:4:"name";s:17:"Phạm Sơn Tùng";}
```
Nếu bạn sử dụng driver `cookie`, các session sẽ được lưu trữ dưới dạng cookie, được mã hóa để người dùng không thể xem hoặc chỉnh sửa.

![alt](https://images.viblo.asia/full/ceeaa1b4-991c-40f4-bfdd-46acb060d83b.JPG)

> **Lưu ý:** Bạn có thể tắt chế độ mã hóa session tương tự như một cookie.

Nếu bạn đang sử dụng driver `database`, bạn có thể vào database của mình sau đó truy vấn đến table `sessions`.

![alt](https://images.viblo.asia/af8b4454-4a6a-4137-aaf0-e961913c2303.JPG)

Như bạn thấy, driver `database` cũng mã hóa các session để tăng tính bảo mật.

Còn nếu bạn sử dụng driver `array` thì như đã nói ở trên, nó không lưu trữ ở bất kỳ đâu cả, các session sẽ biến mất ở request kế tiếp.

### 2.Lấy dữ liệu (Retrieving data)

Cũng như lưu trữ session, ở phần lấy dữ liệu ta cũng có hai cách.
* Thông qua `Illuminate\Http\Request`
```php
$request->session()->get('key');
```
* Thông qua global helper `session`
```php 
session('key');
```
Nếu session không tồn tại, bạn có thể thiết lập giá trị mặc định cho nó. Bạn có thể truyền giá trị mặc định hoặc một Closure xử lý logic, sau đó trả về giá trị mặc định cần thiết lập.

* Thông qua `Illuminate\Http\Request`
```php 
$request->session()->get('key', 'default');

$request->session()->get('key', function() {
    return 'default';
});
```
* Thông qua global helper `session`
```php 
session('key', 'default');

session('key', function() {
    return 'default';
});
```

#### a.Lấy tất cả dữ liệu session (Retrieving all session data)

Với lớp `Illuminate\Http\Request` cho phép chúng ta lấy toàn bộ session được lưu trữ thông qua method `all`.
```php
$request->session()->all();
```
#### b.Kiểm tra một session có tồn tại (Checking if a session exist)

Để kiểm tra sự tồn tại của một session, ta có thể sử dụng method `has` có trong lớp khởi tạo `Illuminate\Http\Request`. Method `has` sẽ trả về `true` nếu session tồn tại và có giá trị không phải là `null`.

Chẳng hạn mình đăng ký route sau:
```php
Route::get('/session', function(Request $request) {
    dd($request->session()->has('foo'));
});
```
Do chúng ta chưa lưu trữ session nào có key là `foo` cả, nên kết quả sẽ là `false`.

Giờ các bạn thử lưu trữ session `foo` này nhưng với giá trị là `null` xem sao.
```php
Route::get('/session', function(Request $request) {
    $request->session()->put('foo', null);
    
    dd($request->session()->has('foo'));
});
```
Vâng, kết quả thu được vẫn sẽ là `false`.

Nếu bạn chỉ muốn kiểm tra xem session có tồn tại hay không, dù có giá trị `null` vẫn chấp nhận thì bạn có thể sử dụng method `exists` thay cho `has`.

```php
Route::get('/session', function(Request $request) {
    $request->session()->put('foo', null);
    
    dd($request->session()->exists('foo'));
});
```
Lúc này kết quả ta nhận được sẽ là `true`.

#### c.Lấy và xóa một mục (Retrieving and deleting an item)

Một số session sau khi lấy xong thì không cần dùng đến nữa, có một method thực hiện chuỗi công việc này. Method `pull` sau khi trả về giá trị của session thì sẽ xóa nó đi.
```php
$value = $request->session()->pull('key', 'default');
```

### 3.Flash data

Chắc các bạn đã nghe về flash session data ở trong các tập trước rồi, nên mình sẽ không nói lại nữa. Nếu bạn muốn lưu trữ một số session chỉ trong request kế tiếp, bạn có thể sử dụng method `flash`.
```php
$request->session()->flash('status', 'message');
```

Nếu bạn muốn flash data một lần nữa, bạn có thể sử dụng method `reflash` để kéo dài "tuổi thọ" cho các flash data.
```php
$request->session()->reflash();
```
Nếu muốn chỉ đích danh flash data cần giữ lại, bạn có thể sử dụng method `keep` và truyền tham số là key flash session mà bạn muốn flash một lần nữa.
```php
$request->session()->keep('key');

$request->session()->keep(['foo', 'bar']);
```

### 4.Xóa dữ liệu (Deleteing data)

Method `forget` sẽ giúp chúng ta dễ dàng xóa bỏ một session.

```php
$request->session()->forget('key');

$request->session()->forget(['foo', 'bar']);
```
Nếu bạn muốn xóa tất cả các session thì có thể sử dụng method `flush`.
```php
$request->session()->flush();
```

### 5.Khởi tạo lại session ID (Regenerating the session ID)

Việc tạo lại session ID sẽ ngăn chặn người dùng tấn công ứng dụng với session fixation. Đây là một hình thức tấn công đơn giản nhưng lại vô cùng hiệu quả. Nó dựa vào việc server không thay đổi session ID sau khi đăng nhập vào hệ thống. Tin tặc sẽ lợi dụng điều này để thông qua session ID người dùng đánh cắp thông tin.

Dễ hiểu như thế này, một website http://unsafe.com chấp nhận các session ID từ request. Hacker sẽ gửi đường dẫn http://unsafe.com?SID=1, với `SID=1` chính là session ID mà server đã cung cấp cho trình duyệt của hắn. Sau khi người dùng bị lừa và đăng nhập vào hệ thống thông qua link http://unsafe.com?SID=1 thì hacker có thể đăng nhập gián tiếp tài khoản người dùng, từ đó thực hiện khai thác thông tin, dữ liệu.

Chính vì thế, Laralve sẽ tự động tạo lại session ID nếu ứng dụng của bạn sử dụng `LoginController` mặc định. Nếu như bạn cần tạo lại session ID thủ công, bạn có thể sự dụng method `regenerate`.

```php
$request->session()->regenerate();
```

# Cookie trong Laravel

Cookie là những tập tin nhỏ mà trình duyệt tạo ra nhằm lưu trữ thông tin trong quá trình duyệt web, cookie có thể sử dụng để duy trì trạng thái người dùng khi vào các trang khác nhau trên một website. Cookie có thời gian hết hạn, do đó trong khoảng thời gian chưa hết hạn, nó lưu trữ thông tin người dùng cho những lần truy cập tiếp theo trên chính website đó.
> Tất cả cookie được tạo bởi framework đã mã hóa và đăng ký bằng các mã chứng thực, chính vì vậy nó sẽ được coi là không hợp lệ nếu người dùng cố tình thay đổi.

## I.Lấy cookie từ request (Retrieving cookie from request)

Để nhận một giá trị cookie từ request, bạn có thể sử dụng method `cookie` trong lớp khởi tạo `Illuminate\Http\Request`.
```php
$value = $request->cookie('name');
```
Ngoài ra, bạn có thể sử dụng Cookie `facade` để thay thế.
```php
use Illuminate\Support\Facades\Cookie;

$value = Cookie::get('name');
```
## II.Đính kèm cookie đến response (Attaching cookie to response)

Bạn có thể đính kèm mộ cookie đến object `Illuminate\Http\Response` bằng cách sử dụng method `cookie`. Bạn nên truyền đầy đủ tham số tên, giá trị cookie và thời gian tồn tại của cookie (tính theo phút). Nếu bạn bỏ qua tham số thời gian, Laravel sẽ coi cookie tồn tại như một session.
```php
return response('Hi')->cookie('name', 'Phạm Sơn Tùng', $minutes);
```

Method `cookie` hoạt động tương tự hàm `setcookie` mặc định của PHP. Chính vì vậy nó cũng có thể nhận một số tham số tùy chọn khác.

```php
return response('Hi')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```
Ngoài ra bạn có thể sử dụng method `queue` trong `Cookie` facade. Nói một chút về "queue", nó có nghĩ đen là "xếp hàng". Hiểu một cách đơn giản, thì set cookie "queue" sẽ thực hiện cuối cùng khi đã hoàn tất cả xử lý khác trong request hiện tại

## III.Generating cookie instances

Đại khái nếu bạn muốn set cookie chỉ khi cookie đó trả về cùng với response thì đăng ký cookie với method `cookie`. Cách này rất giống với cách đăng ký ở trên nhưng chúng ta có thể chèn một tham số object thay vì các tham số riêng lẻ như trên.
```php
$cookie = cookie('name', 'value', $minutes);

return back()->cookie($cookie);
```

# Model trong Laravel

## Defining Models (định nghĩa Model)
**ORM** (Object Relational Mapping) là tên gọi chỉ việc ánh xạ các record dữ liệu trong hệ quản trị cơ sở dữ liệu sang dạng đối tượng mà mã nguồn đang định dạng trong class, nôm na có thể hiểu là một dạng kỹ thuật giúp lập trình viên thao tác dễ dàng hơn với database. Trong Laravel chúng ta sử dụng **Eloquent ORM**.

Thông thường các `model` nằm trong thư mục `app` nhưng bạn có thể thoải mái đặt nó ở bất kì đâu bạn muốn miễn là tự động load được tới file `composer.json`. Tất cả các `model` đều extend `Illuminate\Database\Eloquent\Model`.

Cách dễ nhất để tạo 1 `model` là dùng lệnh `make:model` trong **Artisan command**

```
php artisan make:model Flight
```

Nếu bạn muốn tạo luôn `database migration` thì bạn có thể dùng `--migration` hoặc `-m`
```
php artisan make:model Flight --m

php artisan make:model Flight --migration
```

### Tên bảng

Mặc định nếu như bạn đặt tên model là `Post` thì laravel sẽ mapping với bảng tên `posts`.
 Những nếu bạn muốn đặt tên bảng khác đi nhưng vẫn phải theo đúng quy chuẩn 
 convention đặt tên trong Laravel nhé(snake case). Ví dụ mình không đặt tên `posts` 
 nữa mà mình đặt tên là `my_posts` chẳng hạn thì biến `$table` trong 
 `Illuminate\Database\Eloquent\Model` sẽ mapping đúng model với tên bảng chúng ta 
 khai báo.
```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    //
}
```

### Retrieving Models (lấy dữ liệu từ model)

`Eloquent` cung cấp cho chúng ta rất nhiều câu truy vấn query được định nghĩa sẵn, từ đó giúp việc lấy dữ liệu từ database trở nên hết sức nhanh gọn và dễ dàng. Ví dụ :
```php
<?php
use  App\Post;

$posts = Post::all();

foreach ($posts as $post) {
    echo $post->title;
}
```
Method all sẽ trả về tất cả dữ liệu có trong bảng. Hoặc ta có thể thêm điều kiện, sau đó sử dụng `get`.
```php
<?php 
$posts = Post::where('active', 1)
               ->orderBy('id', 'desc')
               ->take(10)
               ->get();
```

# Migration

## 1.Giới thiệu chung

Migration giống như một hệ thống quản lý phiên bản giống như Git nhưng dành cho cơ sở dữ liệu của bạn. Migration cho phép bạn định nghĩa các bảng trong CSDL, định nghĩa nội dung các bảng cũng như cập nhật thay đổi các bảng đó hoàn toàn bằng PHP. Đồng thời các thao tác với CSDL này còn có thể sử dụng trên các loại CSDL khác nhau như MySQL, SQL Server, Postgres, ... mà không cần phải chỉnh sửa lại code theo CSDL sử dụng.
Điều kiện tiên quyết để chạy migration một cách thành công:

* Phải có kết nối với database .
* migrations muốn sử dụng được thì phải nằm trong thư mục App\database\migrations

## 2.Tạo migration

Để Tạo Migrations thì các bạn cũng có 2 cách tạo là dùng tay và dùng lệnh, nhưng mình khuyến khích mọi người dùng lệnh.

Tạo Migrations bằng lệnh thì các bạn mở cmd lên và trỏ tới thư mục chứa project của các bạn(như mọi khi :-) ) và gõ 1 trong các lệnh sau tùy theo mục đích của bạn.

* php artisan make:migration TenMigrate  : Tạo migrations thông thường.
* php artisan make:migration TenMigrate --create=TableName  : Tạo migrations cho bảng.
* php artisan make:migration TenMigrate --table=TableName  : Tạo migrations chỉnh sửa bảng.
>Chú Thích: TenMigrate,TableName là các thông số các bạn có thể tùy chỉnh.

VD: Mình sẽ tạo Migrations create_users_table cho table users.

```
php artisan make:migration create_users_table --create=users
```

Nếu tạo thành công nó sẽ báo dạng như sau: Created migration: xxxxxxxxxxxx: Lúc này bạn có thể kiểm tra lại bằng cách truy cập vào  App\database\migrations nếu thấy  có file tên trùng với phần xxxxxxxx ở trên thì là đã thành công.

Tiếp đó các bạn mở file ra và sẽ thấy nội dung có dạng:

```php 
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateUsersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
           
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        //
    }
}
```
### Hàm up() và hàm down().
* Hàm up trong Migrations có tác dụng thực thi migration
* Hàm down trong Migrations có tác dụng thực thi đoạn lệnh `rollback`(trở về trước đó).

## Thực thi migration

Các lệnh thực thi migrations:

* php artisan migrate =	 chạy migration
* php artisan migrate:resest =	 resest lại migration
* php artisan migrate:refesh =	 chạy lại migration
* php artisan migrate:status =	 xem trạng thái của migration
* php artisan migrate:install =	 cài đặt migration

## Schema

Bây giờ chúng ta sẽ tìm hiểu kỹ hơn `Schema` facade thực thi như nào nhé. Trong file migration để dùng `Schema` thì chúng ta sẽ use `Illuminate\Support\Facades\Schema`.
Nếu muốn tạo một bảng mới trong DB của mình thì chúng ta có thể sử dụng
```php
Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
});
```
Nếu các bạn muốn kiểm tra xem `table` hoặc `column` có tồn tại hay không thì ta dùng
```php
if (Schema::hasTable('users')) {
    //
}

if (Schema::hasColumn('users', 'email')) {
    //
}
```
Nếu muốn đổi tên bảng từ `post` sang `posts` thì ta dùng
```php
Schema::rename('post', 'posts')
```
Khi chúng ta muốn xóa bảng thì có thể sử dụng Schema::drop()
```php
Schema::drop('users');

Schema::dropIfExists('users');
```

### Foreign Key Constraints

Đôi khi chúng ta muốn tạo các rằng buộc cho các bảng, chúng ta có thể sử dụng cú pháp sau để rằng buộc cho 2 bảng:
```php
Schema::table('posts', function ($table) {
    $table->integer('user_id')->unsigned();

    $table->foreign('user_id')->references('id')->on('users');
});
```
Chú ý nếu không migrate mà không chạy được thì các bạn có thể tách ra làm 2 file migration để chạy.
Để drop một foreign ta dùng : `$table->dropForeign('posts_user_id_foreign')`;
Chúng ta nên để ý quy tắc đặt tên foreign `<tên_table>_<tên_khóa_ngoại>_foreign`
Bạn có thể kích hoạt hay bỏ kích hoạt việc sử dụng foreign key constraint trong migration sử dụng hai hàm sau:
```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();    
```    
    