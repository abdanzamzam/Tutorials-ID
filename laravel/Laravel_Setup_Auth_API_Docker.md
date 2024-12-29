
# Complete Guide: Setting Up Laravel with Authentication, Authorization, CRUD API, and Docker Deployment

This guide provides a comprehensive walkthrough for setting up Laravel, implementing authentication and authorization, creating a simple CRUD API for managing products with authentication and authorization, and deploying the application using Docker.

---

## **1. Setting Up Laravel**

### **Step 1: Install Laravel**
Ensure you have PHP, Composer, and a database server installed. Then, run:

```bash
composer create-project --prefer-dist laravel/laravel laravel-api
cd laravel-api
```

### **Step 2: Set Up the Database**

1. Configure your `.env` file with database credentials:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_api
DB_USERNAME=root
DB_PASSWORD=password
```

2. Create the database:

```bash
mysql -u root -p
CREATE DATABASE laravel_api;
```

### **Step 3: Run the Development Server**

Start the Laravel development server:

```bash
php artisan serve
```

Access your application at `http://localhost:8000`.

---

## **2. Implementing Authentication**

### **Step 1: Install Laravel Breeze**

Laravel Breeze provides a simple implementation of authentication:

```bash
composer require laravel/breeze --dev
php artisan breeze:install
npm install && npm run dev
php artisan migrate
```

### **Step 2: Test Authentication**

Visit `http://localhost:8000/register` to create a user and `http://localhost:8000/login` to log in. Laravel will automatically set up authentication routes.

---

## **3. Implementing Authorization**

### **Step 1: Add Policies**

Generate a policy for the `Product` model:

```bash
php artisan make:policy ProductPolicy --model=Product
```

### **Step 2: Define Permissions in the Policy**

Edit `app/Policies/ProductPolicy.php`:

```php
public function viewAny(User $user)
{
    return $user->role === 'admin';
}

public function update(User $user, Product $product)
{
    return $user->id === $product->user_id;
}

public function delete(User $user, Product $product)
{
    return $user->id === $product->user_id;
}
```

### **Step 3: Register the Policy**

Edit `app/Providers/AuthServiceProvider.php`:

```php
use App\Models\Product;
use App\Policies\ProductPolicy;

protected $policies = [
    Product::class => ProductPolicy::class,
];
```

---

## **4. Creating a CRUD API for Products**

### **Step 1: Create the Product Model and Migration**

Run the following commands:

```bash
php artisan make:model Product -m
```

Edit the generated migration file in `database/migrations/`:

```php
Schema::create('products', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->text('description');
    $table->decimal('price', 8, 2);
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->timestamps();
});
```

Run migrations:

```bash
php artisan migrate
```

### **Step 2: Create the Controller**

Generate a controller:

```bash
php artisan make:controller ProductController --api
```

Implement the CRUD logic in `ProductController.php`:

```php
use App\Models\Product;
use Illuminate\Http\Request;

public function index()
{
    $this->authorize('viewAny', Product::class);
    return Product::all();
}

public function store(Request $request)
{
    $this->authorize('create', Product::class);

    $request->validate([
        'name' => 'required|string|max:255',
        'description' => 'required|string',
        'price' => 'required|numeric',
    ]);

    return Product::create([
        'name' => $request->name,
        'description' => $request->description,
        'price' => $request->price,
        'user_id' => $request->user()->id,
    ]);
}

public function show(Product $product)
{
    $this->authorize('view', $product);
    return $product;
}

public function update(Request $request, Product $product)
{
    $this->authorize('update', $product);

    $request->validate([
        'name' => 'sometimes|required|string|max:255',
        'description' => 'sometimes|required|string',
        'price' => 'sometimes|required|numeric',
    ]);

    $product->update($request->only(['name', 'description', 'price']));

    return $product;
}

public function destroy(Product $product)
{
    $this->authorize('delete', $product);
    $product->delete();
    return response(null, 204);
}
```

### **Step 3: Register API Routes**

Edit `routes/api.php`:

```php
use App\Http\Controllers\ProductController;

Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('products', ProductController::class);
});
```

---

## **5. Deploying with Docker**

### **Step 1: Create a `Dockerfile`**

```dockerfile
FROM php:8.2-fpm

RUN apt-get update && apt-get install -y \
    libpng-dev \
    zip \
    unzip \
    git \
    curl \
    && docker-php-ext-install pdo_mysql gd

COPY . /var/www
WORKDIR /var/www

RUN composer install

CMD ["php", "artisan", "serve", "--host=0.0.0.0", "--port=8000"]
EXPOSE 8000
```

### **Step 2: Create a `docker-compose.yml`**

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
    ports:
      - "8000:8000"
    volumes:
      - .:/var/www
    environment:
      - DB_CONNECTION=mysql
      - DB_HOST=db
      - DB_PORT=3306
      - DB_DATABASE=laravel_api
      - DB_USERNAME=root
      - DB_PASSWORD=password

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: laravel_api
    ports:
      - "3306:3306"
```

### **Step 3: Build and Run the Containers**

Run the following commands:

```bash
docker-compose up --build
```

### **Step 4: Access the Application**

Visit `http://localhost:8000` to access your application running inside Docker.

---

With this setup, you have a Laravel application with authentication, authorization, a CRUD API for products, and deployment using Docker. Let me know if you need further assistance or modifications! 😊
