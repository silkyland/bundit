โจทย์
---
1. ให้น.ศ. สร้างโปรเจคขึ้นใหม่ชื่อว่า todoapp เอาขึ้น github เป็นแบบ public
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
    - resources/views/auth/login.blade.php

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
9. พิมพ์ 
    ```bash
        php artisan migrate
    ```