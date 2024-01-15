# Laravel
Laravel 8.x
![image](https://hackmd.io/_uploads/ByH4U9wdp.png)

## 安裝 Composer
![image](https://hackmd.io/_uploads/Syj6kSS_p.png)
[Composer](https://getcomposer.org/)是PHP的套件管理程式，使用Laravel之前須先安裝該程式



安裝指令：`$ brew install composer # 安裝composer`

---

## 安裝 Laravel 與建立專案
### 安裝 Laravel installer：
```bash!
$ composer global require laravel/installer
# Make sure to place Composer's system-wide vendor bin directory in your $PATH
# macOS: $HOME/.composer/vendor/bin
# 編輯.zshrc 檔案，將PATH加上去
export PATH="$PATH:$HOME/.rvm/bin:$HOME/.composer/vendor/bin"
```
### 如何創建新專案：
```bash!
### 方法1 使用laravel installer (需要事先安裝)
$ laravel new new_project # 建立新專案

$ cd new_project # 進入專案目錄
$ php artisan key:generate --ansi # 產生新的APP_KEY
$ php artisan serve # 開啟local server
$ php artisan -V # 檢查version

### 方法2 使用composer
$ composer create-project --prefer-dist laravel/laravel YourProjectName
$ composer create-project --prefer-dist laravel/laravel YourProjectName "8.*"
... # 以下相同
```
---

## 初探路由與首頁
大致目錄結構：
- app：應用程式的核心程式碼
- resource：畫面資源
- routes：路徑定義

首頁是在`public/index.php`

主要透過`routes/web.php`設定網頁路徑。
```php!
# 基礎範例
Route::get('/', function () {
    return view('welcome');
});
```
該路徑對應到`resources/view/welcome.blade.php`，`blade`是畫面的樣板。

## 模板系統
Laravel預設使用**Blade**作為模板系統。
在`resources/view/layouts`中建立模板，例如`app.blade.php`:
```php!
# resources/view/layouts/app.blade.php
@yield('title')

@section('sidebar')
    This is master sidebar.
@show

<div class="container">
    @yield('content', 'Default content')
</div>
```
要使用該layout模板，在任意模板中：
```php!
# resources/view/child.blade.php
@extends('layouts.app')

@section('title', 'Page title')

@section('sidebar')
    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

### including subview
`@include(shared.errors)`

### View中撰寫PHP
Blade允許撰寫php程式碼，可以使用PHP`<?php ?>`標記，或是使用Blade的`{{}}`語法顯示變數的值。
```php!
{{-- Blade 註釋 --}}
<?php
    // 這裡是 PHP 代碼
    $variable = "Hello, Blade!";
?>

<!DOCTYPE html>
<html>
<head>
    <title>Home Page</title>
</head>
<body>
    <h1>{{ $variable }}</h1>

    {{-- Blade 指令 --}}
    @if($condition)
        <p>The condition is true.</p>
    @else
        <p>The condition is false.</p>
    @endif
</body>
</html>
```
### 後端傳遞參數至View
在Route或Controller中return參數至View中：
`return view('home', ['name' => "Linda"]);`

---
## CSS放哪裡
最終是使用`public/css`中的`.css`檔案。
ex: `<link rel="stylesheet" type="text/css" href="css/app.css">`
### Webpack
![image](https://hackmd.io/_uploads/rk8CCY8_T.png)
Laravel 8 是使用[webpack](https://webpack.js.org/)管理與打包前端資源，並且透過[Laravel Mix](https://laravel-mix.com/)作為Laravel與webpack的中間層程式。
經常用到的指令：
- npm install：根據package.json將專案的依賴與配置下載安裝到node_module資料夾中
- npm run watch：Laravel Mix提供的指令，監聽前端資源並在內容變化時即時編譯
### 使用Sass
先在本機安裝Sass：`npm install sass sass-loader`
`webpack.mix.js`:
`mix.sass('resources/sass/app.scss', 'public/css');`
參考：[Laravel Sass](https://laravel.com/docs/8.x/mix#sass)

### 背景圖片
Laravel Mix會將你的圖片也複製打包，因此`resource/images`中放置的圖片會被複製到`public`資料夾中。
```css!
.example {
    background: url('../images/example.png');
}
```
或者在`webpack.js`中：
```javascript!
mix.copyDirectory('resources/images', 'public/images');
```
## 使用JavaScript
webpacker 會打包`resources/js/app.js`：
`mix.js('resources/js/app.js', 'public/js')`

接著在blade中使用`<script>`引入打包後的`public/app.js`。
ex: `<script src="./js/app.js"></script>`

或著也可以在blade中的`<script>`區塊撰寫JavaScript程式碼。

:::warning
:warning: webpack會將function封裝並打包至`public/js/app.js`，因此在`resouces/js/app.js`以外的區域是無法使用內部的function，除非將function賦予至window全域變數。
ex: `window.myFunction = myFunction;`
:::

## 版本控制 and env參數
### 版本控制與去除快取影響
[官網解說](https://laravel.com/docs/8.x/mix#versioning-and-cache-busting)
在`webpack.mix`中使用：
```js!
mix.js('resources/js/app.js', 'public/js')
   .version();
```
在其他地方必須使用`mix`這個function來實現版本控制，這樣可以看到載入JS或CSS等資源時後方有加上id，讓程式能夠自動識別版本變化：
```php!
<script src="{{ mix('/js/app.js') }}"></script>
```
![image](https://hackmd.io/_uploads/SkbeQAv_T.png)

### env 環境參數

[環境參數](https://laravel.com/docs/8.x/mix#environment-variables)寫在`.env`檔案中，並且內部的變數會被加載到`$_ENV `這個超級全域變數中，可以透過`env`輔助函式來獲取。
```
MIX_SENTRY_DSN_PUBLIC=http://example.com
```
- 在php程式中：`env('MIX_SENTRY_DSN_PUBLIC', 'default value')`
- 在js程式中：`process.env.MIX_SENTRY_DSN_PUBLIC`

例如版本控制中可以環境變數：
```js
// webpack.mix.js
app_ver = process.env.APP_VERSION
...
mix.js('resources/js/app.js', `public/js/${app_ver}/app.js`)
```
```php!
// app.blade.php
<?php $app_version = env('APP_VERSION') ?>
<script src="{{mix("/js/{$app_version}/app.js")}}"></script>
```

## Controller & Routes
### 創建基本Controller
指令創建Controller：
```bash!
php artisan make:controller PageController
# INFO Controller [app/Http/Controllers/PageController.php] created successfully.
```
Controller命名慣例為`{單數首字大寫}Controller`

接著在Controller中創建function：
```php!
class PageController extends Controller
{
    public function index(){
        return view('home');
    }
}
```

### 指定Controller給Route
將新產生的Controller與function指定給特定Route：
```php!
// web.php
use App\Http\Controllers\PageController;

Route::get('/', [PageController::class, 'index'] );
```
### Query String & Request
在Laravel中不太會使用傳統PHP的`$_GET`語法，因為在Controller的function，有可能同時被GET或POST所呼叫，因此有可能要使用`$_GET`或`$_POST`來取值，因此建議使用Laravel提供的`Request`功能。
```php!
use Illuminate\Http\Request;

class PageController extends Controller
{
    public function index(Request $request){
        return view('home', ['name' => $request -> input('name')]);
    }
}
```

---

## RESTful CRUD
### 手刻基礎Controller
[路徑參數](https://laravel.com/docs/8.x/routing#required-parameters)
想要製作`products/{id}`風格的路徑：
```php!
Route::get('/products/{id}', function($id){
    ...
})
```
可以透過Regex限制參數的形式：
```php!
Route::get('/user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```
### Resources Controller
[官方說明](https://laravel.com/docs/8.x/controllers#resource-controllers)
使用指令快速產生Controller：
`php artisan make:controller PhotoController --resource`

撰寫對應的路徑：
```php!
use App\Http\Controllers\PhotoController;
 
Route::resource('photos', PhotoController::class);
```
一次撰寫多個resources 路徑：
```php!
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```
可以透過`php artisan route:list`查看當前所有路徑資訊。

![image](https://hackmd.io/_uploads/HJQdJk3Op.png)
![image](https://hackmd.io/_uploads/B1rLsy2u6.png)

以下範例中，使用到`redirect()`重新導向、`route()`route helper。
基礎CRUD Controller範例：
```php=
public function index(){
        $products = $this->getProducts();
        return view('products.index', ['products' => $products]);
    }

    public function create(){
        $products = $this->getProducts();
        return view('products.create', ['products' => $products]);
    }

    public function store(){
        $products = $this->getProducts();
        echo '新增資料...';
        return redirect() -> route('products.create');
    }

    public function show($id){
        $products = $this->getProducts();
        return view('products.show', ['products' => $products, 'id' => $id]);
    }

    public function edit($id){
        $products = $this->getProducts();
        return view('products.edit', ['products' => $products, 'id' => $id]);
    }

    public function update($id){
        echo '更新資料...';
        return redirect() -> route('products.edit', ['product' => $id]);
    }

    public function destroy($id){
        echo '執行刪除...';
        return redirect() -> route('products.index');
    }
```
首頁範例：
```php=
<h1>Product Index</h1>
    <div>
        <ul>
            @foreach($products as $product)
                <li>
                    <div><a href="{{ route('products.show', ['product' => $product['id']]) }}">Product Name: {{ $product['name'] }}</a></div>
                    <div><a href="{{ route('products.edit', ['product' => $product['id']]) }}">Edit</a></div>
                    <div>
                        <form action="{{ route('products.destroy', ['product' => $product['id']]) }}" method="post">
                            @csrf
                            @method('delete')
                            <button type="submit">刪除</button>
                        </form>
                    </div>
                </li>
            @endforeach
        </ul>
        <a href="{{ route('products.create') }}">新增商品</a>
    </div>
```

### Form
[Blade Form官方說明](https://laravel.com/docs/8.x/blade#forms)
在`create`, `edit`等要提交表單的Action頁面中需要使用到CSRF的token，並且在PUT/PATCH/DELETE等的REST操作中還要使用到`@method`helper：
```php!
// create
<form action="{{ route('products.store') }}" method="POST">
    @csrf
    <label>
        Product Name: <input type="text" name="name">
    </label>
    <br>
    <button type="submit">Submit</button>
</form>

//edit
<form action="{{ route('products.update', ['product' => $id]) }}" method="POST">
    @csrf
    @method('PUT')
    <label>
        Product Name: <input type="text" name="name" value="{{ $products[$id]['name'] }}">
    </label>
    <br>
    <button type="submit">Submit</button>
</form>
```
`@csrf`會產生編碼變動的token，而`@method`則會產生type為hidden的input，如果不想要使用`@method`自己手打也是可以：
![image](https://hackmd.io/_uploads/SJ9zINnuT.png)


---

## Cookie
### Cookie介紹與設置Cookie
Cookie是儲存在用戶端（通常是瀏覽器）的小型文本數據，用於跟蹤與識別網站用戶，可以使用JS的`document.cookie`來讀寫cookie，但由於是純文字形式，操作上並不是很方便，因此可以使用第三方套件 [js-cookie](https://www.npmjs.com/package/js-cookie) 等方便讀寫cookie。

Cookie會在關閉瀏覽器時刪除，或者使用者手動刪除，也可以透過設置參數`expires`and`mas-age`指定存活時間。[閱讀更多cookie資訊](https://www.shubo.io/cookies/#cookie-%E7%9A%84%E5%8F%83%E6%95%B8)
### Laravel 的 Cookie
[讀取Cookie](https://laravel.com/docs/8.x/requests#cookies)
Laravel產生的cookie是有被加密過的，因此客戶端修改的cookie不會被認證，獲取cookie的方式：
```php!
// 方式 1
$value = $request->cookie('cookie_name');

// 方式 2
use Illuminate\Support\Facades\Cookie;
 
Cookie::get('cookie_name');
```

request header中的cookie：
![截圖 2024-01-12 下午4.04.39](https://hackmd.io/_uploads/Skqkm_0da.png)
response中的cookie只有被加密過的才會被讀取，可至`app/Http/Moddleware/EncryptCookies.php`中設定：
```php!
// EncryptCookies
/**
 * The names of the cookies that should not be encrypted.
 *
 * @var array
 */
protected $except = [
    'cookie_name',
];
```
[Response寫入Cookie](https://laravel.com/docs/8.x/responses#attaching-cookies-to-responses)
當reponse header中有設定的Set-Cookie時，瀏覽器會根據Cookie中有被修改的部分cookie進行更新。
```php!
// 方式 1
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);

// 方式 2
use Illuminate\Support\Facades\Cookie;
 
Cookie::queue('name', 'value', $minutes);
```
---
## Ajax
### 產生token
laravel提供Helper：`csrf_token()`，可以產生Ajax操作所需要的token，建議將其放置於Head中的meta，再透過JS取用。
```htmlembedded!
<meta name="csrf-token" content="{{ csrf_token() }}">
```
```javascript
let csrfTokenMeta = document.querySelector('meta[name="csrf-token"]');
formData.append("_token", csrfTokenMeta.content);
```
## response
使用Helper：`response`產生response物件
```php
return response('Hello World', 200, $headers);
// or
return response()->json(['foo' => 'bar'], 200, $headers);
```
---

## 例外處理
### 手動觸發404
`abort(404);`

---
## URL Generating
### 基礎產生url
如果路徑如下：
```php!
Route::get('/post/{post}', function (Post $post) {
    //
})->name('post.show');
```
則可以使用`route`helper產生URL：
```php!
echo route('post.show', ['post' => 1])
```
## Helper

**Asset**
asset能夠創建URL，從`public`資料夾中抓取檔案：
```php!
$url = asset('img/01.jpg'); // http://example.com/assets/img/photo.jpg
```
## Request
## 獲取 Input Data
可以使用`$request->all()`或`$request->collect()`來獲取所有的輸入資料，前者回傳傳統PHP矩陣，後者回傳Laravel Collection物件，推薦使用後者。
例如我們可以在Controller中印出Input Data到本地端的serve log中：
```php
class XxxController extends Controller{
    public function show(Request $request){
        $input = $request->collect();
        error_log($input); // {"_method":"GET","_token":"8n...","id":"1"}
        return view('show');
    }
}
```
---

## Collection
Laravel提供`Collection`作為操作PHP矩陣的抽象層工具，具有許多方法可以輕鬆地處理矩陣中的數據，例如：過濾、排序、轉換等。
...