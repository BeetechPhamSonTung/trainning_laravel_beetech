#Upload files trong Laravel
## Tạo view
```
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
```php
class A
{
	public function __construct($text)
	{
	    echo $text;
	}
}

$tweet = new A('Hello World'); // Hello World
```
