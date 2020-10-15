# Laravel 8 Rest API CRUD を Passport 認証で

Passport 認証のチュートリアルで、Laravel 8 による Rest API を使った CRUD を学習する。このチュートリアルでは、Laravel 8 で Passport 認証を使用して RESTful CRUD API を作成する方法を学習する。Passport 認証は通常、デジタル署名を使用して信頼および検証できる情報を送信するために使用される。

RESTful API では、アクションとして HTTP メソッドを使用し、エンドポイントは作用を受けるリソースです。その意味のために HTTP 動詞を使用している。

- GET：リソースを取得する
- POST：リソースを作成する
- PUT：リソースを更新します
- 削除：リソースを削除します

それでは、LaravelAPI を段階的に使用する方法を見てみましょう。パスポート認証を使用した Laravel8 の RESTfulAPI。また、API を使用したユーザー製品の完全に機能する CRUD も紹介します。

- Login API
- Register API
- GetUser Info API
- Product List API
- Create Product API
- Edit Product API
- Update Product API
- Delete Product API

## 実施環境

- Windows 10 Home 64bit バージョン 1909（OS ビルド 18363.1082）
- Node.js v14.11.0

## Step 0: ワークスペースの確保

チュートリアルを行うために今回は、デスクトップにワークスペースを用意する。以降のコマンドは、Git Bash で実行する。

デスクトップに移動する。

```console
cd /c/Users/＜ユーザ名>/Desktop
```

作業フォルダ bogota を作成する。

```console
mkdir ./workspace.test.laravel8.api
```

作業フォルダ内に移動する。

```console
cd ./workspace.test.laravel8.api
```

## Step 1: Download Laravel 8 App

Laravel 8 プロジェクトを作成する。

```console
composer create-project --prefer-dist laravel/laravel bogota
```

プロジェクト内に移動する。

```console
cd ./bogota
```

## Step 2: Database Configuration

次に、パスポートチュートリアルプロジェクトを使用して、インストールした laravel8 の RESTful 認証 API のルートディレクトリに移動します。そして、.env ファイルを開きます。次に、データベースの詳細を次のように追加します。

.env:

```php
DB_CONNECTION=mysql
DB_HOST=192.168.99.100
DB_PORT=13306
DB_DATABASE=products
DB_USERNAME=fsedu
DB_PASSWORD=secret
```

## Step 3: Install Passport Auth

passport パッケージをインストールする。

```console
composer require laravel/passport
```

passport パッケージをインストールしたら、プロバイダーを追加する。

config / app.php:

```php
'providers' =>[
    Laravel\Passport\PassportServiceProvider::class,
],
```

一度、マイグレーションを実行しておく。

```console
php artisan migrate
```

次に、パスポート暗号化キーを生成するために laravel8 をインストールする必要があります。このコマンドは、安全なアクセストークンを生成するために必要な暗号化キーを作成します。

```console
php artisan passport:install
```

## Step 4: Passport Configuration

App/Models/User.php を編集する。

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasFactory, HasApiTokens, Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];

    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];
}
```

App/Providers/AuthServiceProvider.php を以下のように更新する。

```php
<?php

namespace App\Providers;

use Laravel\Passport\Passport;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Gate;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Models\Model' => 'App\Policies\ModelPolicy',
    ];

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();
        Passport::routes();
    }
}
```

config / auth.php に移動し、auth.php ファイルを開きます。次に、API ドライバーをパスポートへのセッションに変更します。このコード'driver' => 'passport'を API に配置します。

```php
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],
```

## Step 5: Create Product Table And Model

Product モデルと migration ファイルと model ファイル、factory ファイルを作成する。

```console
php artisan make:model Product -mf
```

実行結果例:

```
Model created successfully.
Factory created successfully.
Created Migration: 2020_10_13_141640_create_products_table
```

その後、 database / migrations ディレクトリに移動して create_products_table.php ファイルを開きます。次に、次のコードを更新します。

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateProductsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('products', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->text('detail');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('products');
    }
}
```

で充填可能なプロパティを追加 product.php ファイルにナビゲートので、アプリ/モデルディレクトリとオープン product.php のファイルとそれに次のコードを更新します。

app/Models/Product.php

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    use HasFactory;
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'detail'
    ];
}
```

次に、Product モデルのダミーレコードを追加します。したがって、次のようなファクトリを追加する必要があります。

database / factories / ProductFactory.php:

```php
<?php

namespace Database\Factories;

use App\Models\Product;
use Illuminate\Database\Eloquent\Factories\Factory;

class ProductFactory extends Factory
{
    /**
     * The name of the factory's corresponding model.
     *
     * @var string
     */
    protected $model = Product::class;

    /**
     * Define the model's default state.
     *
     * @return array
     */
    public function definition()
    {
        return [
            'name' => $this->faker->name,
            'detail' => $this->faker->text,
        ];
    }
}
```

config/app.php の Faker ロケールを日本語・日本にする。

```php
'faker_locale' => 'ja_JP',
```

オートロードを実行する。

```console
composer dump-autoload
```

## Step 6: Run Migration

マイグレーションを行い、データベースにテーブルを作成する。

```condole
php artisan migrate
```

対話モードに切り替える。

```console
php artisan tinker
```

Users テーブルのダミーデータを 100 件作成する。

```console
>>> App\Models\User::factory()->count(10)->create()
```

次に Products テーブルのダミーデータを 10 件作成する。

```php
>>> App\Models\Product::factory()->count(10)->create()
```

## Step 7: Create Auth And CRUD APIs Route

このステップでは、REST API 認証と CRUD 操作の Laravel 8 ルートを作成します。

したがって、routes ディレクトリに移動し、api.php を開きます。次に、次のルートを api.php ファイルに更新します。

routes/api.php:

```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\PassportAuthController;
use App\Http\Controllers\Api\ProductController;
/*
|--------------------------------------------------------------------------
| API Routes
|--------------------------------------------------------------------------
|
| Here is where you can register API routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| is assigned the "api" middleware group. Enjoy building your API!
|
*/

Route::post('register', [PassportAuthController::class, 'register']);
Route::post('login', [PassportAuthController::class, 'login']);

Route::middleware('auth:api')->group(function () {
    Route::get('get-user', [PassportAuthController::class, 'userInfo']);

    Route::resource('products', ProductController::class);
});
```

## Step 8: Passport 認証と CRUD コントローラーを作成する

PassportAuthController および ProductController という名前のコントローラーを作成します。

```console
php artisan make:controller Api/PassportAuthController
php artisan make:controller Api/ProductController
```

app / Http / Controllers / Api / PassportAuthController.php:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;

use Illuminate\Http\Request;

use App\Models\User;

class PassportAuthController extends Controller
{
    /**
     * Registration Req
     */
    public function register(Request $request)
    {
        $this->validate($request, [
            'name' => 'required|min:4',
            'email' => 'required|email',
            'password' => 'required|min:8',
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => bcrypt($request->password)
        ]);

        $token = $user->createToken('Laravel8PassportAuth')->accessToken;

        return response()->json(['token' => $token], 200);
    }

    /**
     * Login Req
     */
    public function login(Request $request)
    {
        $data = [
            'email' => $request->email,
            'password' => $request->password
        ];

        if (auth()->attempt($data)) {
            $token = auth()->user()->createToken('Laravel8PassportAuth')->accessToken;
            return response()->json(['token' => $token], 200);
        } else {
            return response()->json(['error' => 'Unauthorised'], 401);
        }
    }

    public function userInfo()
    {

        $user = auth()->user();

        return response()->json(['user' => $user], 200);
    }
}
```

app/Http/Controllers/Api/ProductController.php:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class ProductController extends Controller
{
    /**
     * Display a listing of the resource.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        $products = Product::all();
        return response()->json([
            "success" => true,
            "message" => "Product List",
            "data" => $products
        ]);
    }
    /**
     * Store a newly created resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $input = $request->all();
        $validator = Validator::make($input, [
            'name' => 'required',
            'detail' => 'required'
        ]);
        if ($validator->fails()) {
            return $this->sendError('Validation Error.', $validator->errors());
        }
        $product = Product::create($input);
        return response()->json([
            "success" => true,
            "message" => "Product created successfully.",
            "data" => $product
        ]);
    }
    /**
     * Display the specified resource.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function show($id)
    {
        $product = Product::find($id);
        if (is_null($product)) {
            return $this->sendError('Product not found.');
        }
        return response()->json([
            "success" => true,
            "message" => "Product retrieved successfully.",
            "data" => $product
        ]);
    }
    /**
     * Update the specified resource in storage.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Product $product)
    {
        $input = $request->all();
        $validator = Validator::make($input, [
            'name' => 'required',
            'detail' => 'required'
        ]);
        if ($validator->fails()) {
            return $this->sendError('Validation Error.', $validator->errors());
        }
        $product->name = $input['name'];
        $product->detail = $input['detail'];
        $product->save();
        return response()->json([
            "success" => true,
            "message" => "Product updated successfully.",
            "data" => $product
        ]);
    }
    /**
     * Remove the specified resource from storage.
     *
     * @param  int  $id
     * @return \Illuminate\Http\Response
     */
    public function destroy(Product $product)
    {
        $product->delete();
        return response()->json([
            "success" => true,
            "message" => "Product deleted successfully.",
            "data" => $product
        ]);
    }
}
```

```console
php artisan serve
```

## POSTMAN で疎通確認

### ユーザ登録

#### リクエスト

POST http://localhost:8000/api/register

```json
{
  "name": "山本 一郎",
  "email": "ymmt.icr@aaa.com",
  "password": "12345678"
}
```

#### レスポンス

```json
{
  "token": "****"
}
```

### ログイン

#### リクエスト

POST http://localhost:8000/api/login

```json
{
  "name": "山本 一郎",
  "email": "ymmt.icr@aaa.com",
  "password": "12345678"
}
```

#### レスポンス

```json
{
  "token": "****"
}
```

### Bearer Token の入力

POSTMAN の Authorization 欄の Type を Bearer Token にする。
Token をログインのレスポンスにある token にする。

### Product 一覧の取得

#### リクエスト

GET http://localhost:8000/api/products

#### レスポンス

```json
{
  "success": true,
  "message": "Product List",
  "data": [
    {
      "id": 1,
      "name": "吉本 知実",
      "detail": "Officiis eius corporis blanditiis.",
      "created_at": "2020-10-14T02:28:26.000000Z",
      "updated_at": "2020-10-14T02:28:26.000000Z"
    },
    // ・・・
  ]
}
```

### Product ID 4 の Product 情報を取得

#### リクエスト

GET http://localhost:8000/api/products/4

#### レスポンス

```json
{
  "success": true,
  "message": "Product retrieved successfully.",
  "data": [
    {
      "id": 4,
      "name": "近藤 和也",
      "detail": "Ea odio rerum exercitationem.",
      "created_at": "2020-10-14T02:28:26.000000Z",
      "updated_at": "2020-10-14T02:28:26.000000Z"
    }
  ]
}
```
