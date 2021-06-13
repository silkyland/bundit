## โจทย์

1. ให้น.ศ. สร้างโปรเจคขึ้นใหม่ชื่อว่า todoapp เอาขึ้น github เป็นแบบ public
   ```bash
      composer create-project laravel/laravel todoapp --prefer-dist
   ```
2. สร้าง HomeController, CategoryController โดยพิมพ์
   ```bash
       php artisan make:controller HomeController
       php artisan make:controller CategoryController
   ```
3. สร้างโมเดลชื่อว่า Activity พิมพ์
   ```bash
       php artisan make:model Activity -m
   ```
4. สร้างโมเดลชื่อว่า Category ทำเหมือนกันกับข้อ 3
5. สร้างไฟล์ดังนี้

   - resources/views/layout/master.blade.php
   - resources/views/home.blade.php
   - resources/views/create.blade.php
   - resources/views/edit.blade.php
   - resources/views/category/index.blade.php
   - resources/views/category/create.blade.php
   - resources/views/category/edit.blade.php
   - resources/views/auth/login.blade.php
   - resources/views/auth/register.blade.php

6. ในไฟล์ routes/web.php ให้เพิ่มเหล่านี้ลงไป

   ```php
       Route::get('/', [HomeController::class, 'index']);
       Route::get('/create', [HomeController::class, 'create']);
       Route::post('/add', [HomeController::class, 'add']);
       Route::get('/edit/{id}', [HomeController::class, 'edit']);
       Route::post('/update/{id}', [HomeController::class, 'edit']);
       Route::get('/delete/{id}', [HomeController::class, 'delete']);

       Route::get('/category', [CategoryController::class, 'index']);
       Route::get('/category/create', [CategoryController::class, 'create']);
       Route::post('/category/add', [CategoryController::class, 'add']);
       Route::get('/category/edit/{id}', [CategoryController::class, 'edit']);
       Route::post('/category/update/{id}', [CategoryController::class, 'edit']);
       Route::get('/category/delete/{id}', [CategoryController::class, 'delete']);
   ```

7. นำไฟล์ mockup ที่ดาวน์โหลดมา นำมาใช้งาน
8. ใน `databases/migrations`

   - `2021..create_activities_tables...php` ให้เพิ่ม

   ```php
       $table->integer('category_id');
       $table->string('name');
       $table->integer('status');
   ```

   - `2021..create_categories_tables...php` ให้เพิ่ม

   ```php
       $table->string('name');
   ```

9. สร้างฐานข้อมูลชื่อ `todo` collation เป็น `utf8mb4_general_ci`
10. พิมพ์
    ```bash
        php artisan migrate
    ```
11. สร้าง Controller ชื่อว่า `AuthController` เพิ่มฟังก์ชั่น `login()`, `doLogin()`, `logout($id)`, `register()` โดยมีข้อมูลเหล่านี้

    ```php
        public function login(){
            return view('auth.login');
        }

        public function doLogin(Request $request){
            $email = $request->input('email');
            $password = $request->input('password');

            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                return redirect('/');
            }

            return back()->withErrors(['message' => "ไม่สามารถเข้าสู่ระบบได้"]);
        }

        public function register(){
            return view('auth.register');
        }

        public function doRegister(Request $request){
            $email = $request->input('email');
            $name = $request->input('name');
            $password = $request->input("password");
            $password_confirmation = $request->input("password_confirmation");

            if($password != $password_confirmation){
                return back()->withErrors(['message' => 'รหัสผ่าน และ รหัสผ่านยืนยันไม่ตรงกัน']);
            }

            $user = new User();
            $user->email = $email;
            $user->name = $name;
            $user->password = Hash::make($password);
            $user->save();

            return redirect('login');
        }

        public function logout(){
            Auth::logout();
            return redirect('/login');
        }
    ```

12. เพิ่ม routes ใน `routes/web.php` ดังนี้
    ```php
       ...
       Route::get('/login', [AuthController::class, 'login']);
       Route::post('/doLogin', [AuthController::class, 'doLogin']);
       Route::get('/register', [AuthController::class, 'register']);
       Route::post('/doRegister', [AuthController::class, 'doRegister']);
       Route::get('/logout', [AuthController::class, 'logout']);
    ```
13. ในไฟล์ `app/Http/Controllers/HomeController.php` ให้เพิ่ม

    ```php
        public function index(){
            $result = Activity::all();
            $data = [
                "items" => $result,
            ];
            return view('home', $data);
        }

        public function create(){
            $result = Category::all();
            $data = [
                "categories" => $result
            ];
            return view('create', $data);
        }

        public function add(Request $request){
            $category_id = $request->input('category_id');
            $name = $request->input('name');

            $todo = new Activity();
            $todo->category_id = $category_id;
            $todo->name = $name;
            $todo->complete = 0;
            $todo->save();
            return redirect('/');
        }

        public function edit($id){
            $result = Activity::find($id);
            $category_result = Category::all();

            $data = [
                'activity' => $result,
                'categories' => $category_result
            ];

            return view('edit', $data);
        }

        public function update(Request $request, $id){
            $activity = Activity::find($id);
            $activity->name = $request->input('name');
            $activity->category_id = $request->input('category_id');
            $activity->complete = $request->input('complete');
            $activity->save();
            return redirect('/');
        }

        public function delete($id){
            $activity = Activity::find($id);
            $activity->delete();
            return redirect('/');
        }
    ```

14. ในไฟล์ `app/Http/Controllers/CategoryController.php` ให้เพิ่มข้อมูลเหล่านี้

    ```php

        public function index(){
            $result = Category::all();

            $data = [
                'categories' => $result,
            ];

            return view('category.index', $data);
        }

        public function create(){
            return view('category.create');
        }

        public function add(Request $request){
            $name = $request->input('name');

            $category = new Category();
            $category->name = $name;
            $category->save();

            return redirect('/category');
        }

        public function edit($id){
            $result = Category::find($id);

            $data = [
                'category' => $result
            ];

            return view('category.edit');
        }

        public function update(Request $request, $id){
            $name = $request->input('name');

            $result = Category::find($id);
            $result->name = $name;
            $result->save();

            return redirect('/category');
        }

        public function delete($id){
            $category = Category::find($id);
            $category->delete();
            return redirect('/category');
        }
    ```

15. ไฟล์ `resources/views/layout/master.blade.php` เพิ่ม

    ```
    คัดลอกจาก :
     https://raw.githubusercontent.com/silkyland/todos/master/resources/views/layout/master.blade.php
    ```

16. ไฟล์ `resources/views/home.blade.php` เพิ่ม

    ```
    คัดลอกจาก :

    https://raw.githubusercontent.com/silkyland/todos/master/resources/views/home.blade.php
    ```

17. ไฟล์ `resources/views/create.blade.php` เพิ่ม

    ```
    คัดลอกจาก :

    https://raw.githubusercontent.com/silkyland/todos/master/resources/views/create.blade.php
    ```

18. ไฟล์ `resources/views/layout/auth/login.blade.php` เพิ่ม

    ```
    คัดลอกจาก :

    https://raw.githubusercontent.com/silkyland/todos/master/resources/views/login.blade.php
    ```

19. สร้าง user ตัวอย่างมา 1 คนโดยเปิด `routes/web.php` แล้วเพิ่มดังนี้

    ```php
        Route::get('/เพิ่มผู้ใช้งานตัวอย่าง', function(){
            $user = new User();
            $user->name = "สมชาย ใจดี";
            $user->email = "somchai@gmail.com";
            $user->password = Hash::make("1234");
            $user->save();
            return "Success! โปรดอย่ารีเฟรชหน้านี้มันจะเพิ่มสามชายอีกคน หรือไม่ก็ฟ้องว่าอีเมล์ซ้ำ";
        });
    ```

    แล้วเรียกลิงค์

    ```bash
        http://127.0.0.1:8000/เพิ่มผู้ใช้งานตัวอย่าง
    ```

20. ไฟล์ `resources/views/category/index.blade.php` เพิ่ม
    ```html
    @extends('layout.master') @section('content')
    <h1>หมวดหมู่</h1>
    <a class="btn btn-primary" href="/category/create">+ เพิ่มหมวดหมู่</a>
    <table class="table table-striped table-hover">
      <thead>
        <tr>
          <th>#</th>
          <th>ชื่อ</th>
          <th>จัดการ</th>
        </tr>
      </thead>
      <tbody>
        @foreach($categories as $category)
        <tr>
          <td>{{$category->id}}</td>
          <td>{{$category->name}}</td>
          <td>
            <a class="btn btn-warning" href="/category/edit/{{$category->id}}"
              >แก้ไข</a
            >
            <a class="btn btn-danger" href="/category/delete/{{$category->id}}"
              >ลบ</a
            >
          </td>
        </tr>
        @endforeach
      </tbody>
    </table>
    @endsection
    ```
21. ไฟล์ `resources/views/category/create.blade.php` เพิ่ม

    ```html
    @extends('layout.master') @section('content')
    <h1>เพิ่มหมวดหมู่</h1>
    <form action="/category/add" method="post">
      @csrf
      <div class="form-group">
        <label>ชื่อ</label>
        <input
          type="text"
          name="name"
          placeholder="กรอกชื่อหมวดหมู่"
          required="true"
          min="3"
        />
      </div>
      <button type="submit">บันทึก</button>
    </form>
    @endsection
    ```

22. ไฟล์ `resources/views/category/edit.blade.php` เพิ่ม

    ```html
    @extends('layout.master') @section('content')
    <h1>แก้ไขหมวดหมู่</h1>
    <form action="/category/update/{{$category->id}}" method="post">
      @csrf
      <div class="form-group">
        <label>ชื่อ</label>
        <input
          type="text"
          name="name"
          value="{{$category->name}}"
          placeholder="กรอกชื่อหมวดหมู่"
          required="true"
          min="3"
        />
      </div>
      <button type="submit">บันทึก</button>
    </form>
    @endsection
    ```

23. ไฟล์ `resources/views/auth/login.blade.php` แก้ไขเป็น

```html
@extends('layout.master') @section('content')
<div class="row" style="margin-top:50px;">
  <div class="col-md-8 col-md-offset-2">
    <div class="panel panel-default">
      <div class="panel-heading">
        <h4 class="panel-title">โปรดเข้าสู่ระบบ</h4>
      </div>
      <div class="panel-body">
        <form action="/doLogin" method="post">
          @csrf
          <div class="form-group">
            <label>Email: </label>
            <input
              type="email"
              name="email"
              placeholder="กรอก email"
              class="form-control"
            />
          </div>
          <div class="form-group">
            <label for="inputUsername">PASSWORD: </label>
            <input
              type="password"
              name="password"
              placeholder="กรอก password"
              class="form-control"
            />
          </div>
          <button class="btn btn-primary">
            <i class="fa fa-key"></i> เข้าสู่ระบบ
          </button>
        </form>
      </div>
    </div>
  </div>
</div>
@endsection
```

24. ไฟล์ `resources/views/home.blade.php` แก้ไขเป็น

```php
@extends('layout.master')
@section('content')
<p>กรองสถานะ : <a href="#">ทั้งหมด</a> | <a href="#">Completed</a> | <a href="#">Incomplete</a></p>
<div class="panel panel-default">
  <div class="panel-heading">
      <h4 class="panel-title">
          <i class="fa fa-list"></i> รายการที่ต้องทำ
          <span class="pull-right"><a href="/create" class="btn btn-xs btn-success"><i class="fa fa-plus"></i> เพิ่มรายการ</a></span>
      </h4>
  </div>
  <table class="table table-striped table-hover">
      <thead>
          <tr>
              <th>#</th>
              <th>ชื่อรายการ</th>
              <th>หมวดหมู่</th>
              <th>สถานะ</th>
              <th>จัดการ</th>
          </tr>
      </thead>
      <tbody>
          @foreach($items as $item)
          <tr>
              <td>{{$item->id}}</td>
              <td>{{$item->detail}}</td>
              <td>{{$item->category->name}}</td>
              <td>@if($item->status == 1)สำเร็จ @else ไม่สำเร็จ @endif</td>
              <td>
                  <a href="#" class="btn btn-warning btn-xs"><i class="fa fa-edit"></i> edit</a>
                  <a href="/delete/{{$item->id}}" class="btn btn-danger btn-xs"><i class="fa fa-times"></i> delete</a>
              </td>
          </tr>
          @endforeach
      </tbody>
  </table>
</div>
@endsection

```

25. เพิ่มลิงค์ไปยัง `/category` ใน `resources/views/master.blade.php`

    เดิม 
    ```html
      @if(auth()->check()) สวัสดี, {{auth()->user()->name}} |
      <a href="/logout">ออกจากระบบ</a> @else สวัสดี, บุคคลทั่วไป โปรด
      <a href="/login">เข้าสู่ระบบ</a>
      @endif
    ```
    โดยแก้ไขเป็น
    ```html
      @if(auth()->check()) สวัสดี, {{auth()->user()->name}} |
      <a href="/category">จัดการหมวดหมู่</a> | <a href="/logout">ออกจากระบบ</a> @else
      สวัสดี, บุคคลทั่วไป โปรด <a href="/login">เข้าสู่ระบบ</a> @endif
    ```
  

26. ทำการ Join ตางรางระหว่าง `Activity` กับ `Category` โดย

- ที่ไฟล์ app/Models/Category.php เพิ่ม
  ```php
    public function activities(){
      return $this->hasMany(Activity::class);
    }
  ```
  - ที่ไฟล์ app/Models/Activity.php เพิ่ม
  ```php
    public function category(){
      return $this->belongsTo(Category::class);
    }
  ```

27. อัพเดทงานขึ้น `GitHub`
