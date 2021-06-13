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
       $table->int('category_id');
       $table->string('name');
       $table->int('status');
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
            return view('index', $data);
        }

        public function create(){
            $result = Category::all();
            $data = [
                "categories" => $result
            ]
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
