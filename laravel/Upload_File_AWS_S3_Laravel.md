
# How to Upload Files to AWS S3 with Laravel

AWS S3 (Simple Storage Service) is a highly reliable, secure, and scalable object storage service. With Laravel, you can easily integrate S3 for storing files. Below are the steps to upload files to AWS S3 using Laravel.

## 1. **Configure AWS S3**

1. Log in to the [AWS Management Console](https://aws.amazon.com/console/) and create a new bucket in S3.
2. Note the following important details:
   - Bucket name
   - Bucket region
3. Create an IAM user with specific access to S3. Ensure you save:
   - `Access Key ID`
   - `Secret Access Key`

## 2. **Install AWS SDK in Laravel**

Laravel supports integration with AWS through the SDK package. To get started, install the required library:

```bash
composer require league/flysystem-aws-s3-v3 "^3.0"
```

## 3. **Configure Laravel for S3**

### a. Set Environment Variables in the `.env` File
Add the following configuration to your `.env` file:

```env
AWS_ACCESS_KEY_ID=your-access-key-id
AWS_SECRET_ACCESS_KEY=your-secret-access-key
AWS_DEFAULT_REGION=your-region
AWS_BUCKET=your-bucket-name
AWS_URL=https://your-bucket-name.s3.amazonaws.com
```

### b. Configure Filesystem
Open the `config/filesystems.php` file and ensure the S3 driver is configured like this:

```php
'disks' => [
    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION'),
        'bucket' => env('AWS_BUCKET'),
        'url' => env('AWS_URL'),
    ],
],
```

## 4. **Create an Upload Function in the Controller**

Next, create a function in the controller to handle file uploads to S3:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Storage;

public function uploadToS3(Request $request)
{
    // Validate the file
    $request->validate([
        'file' => 'required|file|mimes:jpg,png,pdf,docx|max:10240', // Example validation
    ]);

    // Retrieve the file from the request
    $file = $request->file('file');

    // Generate a unique name for the file
    $fileName = time() . '_' . $file->getClientOriginalName();

    // Upload the file to S3
    $path = Storage::disk('s3')->put('folder-name/' . $fileName, file_get_contents($file), 'public');

    // Retrieve the uploaded file's URL
    $url = Storage::disk('s3')->url('folder-name/' . $fileName);

    return response()->json([
        'message' => 'File uploaded successfully!',
        'url' => $url,
    ]);
}
```

## 5. **Setup Route**

Add a new route for the upload endpoint in the `routes/web.php` or `routes/api.php` file:

```php
use App\Http\Controllers\YourController;

Route::post('/upload-to-s3', [YourController::class, 'uploadToS3']);
```

## 6. **Testing with an HTML Form**

You can test the upload function using Postman or a simple HTML form like this:

```html
<form action="/upload-to-s3" method="POST" enctype="multipart/form-data">
    @csrf
    <input type="file" name="file">
    <button type="submit">Upload</button>
</form>
```

## 7. **Final Result**

After the upload process is successful:
1. The file will be stored in your AWS S3 bucket.
2. The file URL can be accessed if the permissions are set to public.

If you encounter any issues or want to configure file permissions further, you can modify the bucket settings in the AWS Console.

## Conclusion

Integrating AWS S3 with Laravel is very straightforward with the support of Laravel's SDK and filesystem. By following the steps above, you can quickly create a secure and reliable file upload feature.
