# minio-setup

## create minioServiceProvider
```
class MinioStorageServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        Storage::extend('minio', function ($app, $config) {
            $config = $this->formatConfig($config);
            $client = new S3Client($config);
            $root = $config['root'] ?? null;
            $options = $config['options'] ?? [];
            $streamReads = $config['stream_reads'] ?? false;

            return new Filesystem(new AwsS3Adapter(
                $client,
                $config['bucket'],
                $root,
                $options,
                $streamReads
            ), $config);
        });
    }

    /**
     * Format the minio configuration to follow the required standard.
     *
     * @param array $config
     *
     * @return array
     */
    protected function formatConfig(array $config): array
    {
        $config += [
            'credentials'             => [
                'key'    => $config['key'],
                'secret' => $config['secret'],
            ],
            'version'                 => 'latest',
            // 'bucket_endpoint'         => false,
            'use_path_style_endpoint' => true,
        ];

        return $config;
    }

    /**
     * Register the application services.
     *
     * @return void
     */
    public function register()
    {
    }
}
```

## add to config/filesystem.php
```
'minio' => [
            'driver'   => 'minio',
            'key'      => env('AWS_ACCESS_KEY_ID'),
            'secret'   => env('AWS_SECRET_ACCESS_KEY'),
            'region'   => env('AWS_REGION', 'us-east-1'),
            'bucket'   => env('AWS_BUCKET'),
            'endpoint' => env('AWS_ENDPOINT'),
            'root'     => '/storage',
            'url'      => env('AWS_URL'),
        ],

        'filemanager-minio' => [
            'driver'   => 'minio',
            'key'      => env('AWS_ACCESS_KEY_ID'),
            'secret'   => env('AWS_SECRET_ACCESS_KEY'),
            'region'   => env('AWS_REGION', 'us-east-1'),
            'bucket'   => env('AWS_BUCKET'),
            'endpoint' => env('AWS_ENDPOINT'),
            'root'     => '/storage/files',
            'url'      => env('AWS_URL'),
        ],

        'fileexport-minio' => [
            'driver'   => 'minio',
            'key'      => env('AWS_ACCESS_KEY_ID'),
            'secret'   => env('AWS_SECRET_ACCESS_KEY'),
            'region'   => env('AWS_REGION', 'us-east-1'),
            'bucket'   => env('AWS_BUCKET'),
            'endpoint' => env('AWS_ENDPOINT'),
            'root'     => '/storage/exportfiles',
            'url'      => env('AWS_URL'),
        ],
```
## register minioServiceProvider in config/app.php
```
$providers = [
    ...

App\Providers\MinioStorageServiceProvider::class,
];
```

## konfigurasi minio pada docker dapat dilihat pada repo berikut:
[https://gitlab.com/suitmedia/sctv-docker/-/tree/production](https://gitlab.com/suitmedia/sctv-docker/-/tree/production)

catatan: gunakan versi minio tertentu/fixed seperti yg terlihat di file docker-compose.yml repo diatas, agar compatible dgn konfigurasi nginx minio yang ada di repo tsb. 

## contoh konfigurasi di env laravel
- ASSET_URL=https://domain.static-assets.id/bucket-name
- AWS_DEFAULT_REGION="ap-southeast-1"
- AWS_BUCKET=bucket-name
- AWS_REGION="us-east-2"
