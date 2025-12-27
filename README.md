# WordPress Flysystem S3

A WordPress plugin that automatically uploads images, videos, documents, and any other media added through WordPress' media uploader to S3-compatible remote storage.

## Description

This plugin seamlessly integrates with WordPress' native media uploader to automatically sync uploaded files to S3-compatible object storage services. Once configured, all media files are automatically uploaded to your remote storage while maintaining WordPress' standard media library functionality.

## Features

- 🚀 **Automatic Upload**: Automatically uploads media files to S3 when added through WordPress media uploader
- 📁 **All Media Types**: Supports images, videos, documents, and any file type WordPress allows
- 🔄 **Deduplication**: Smart file management using MD5 hashing to avoid duplicate uploads
- 🗄️ **Reference Counting**: Tracks file usage to safely manage deletions
- 🌐 **S3-Compatible**: Works with AWS S3, MinIO, DigitalOcean Spaces, and any S3-compatible storage
- 🔗 **Transparent URLs**: Automatically rewrites attachment URLs to point to remote storage
- 🗑️ **Automatic Cleanup**: Removes files from remote storage when deleted from WordPress

## Requirements

- PHP 7.4 or higher (8.0+ supported)
- WordPress 5.0 or higher
- S3-compatible storage service credentials

**Note:** Composer is only required for development. End users can install the plugin directly from releases.

## Installation

### Method 1: Install via WordPress Admin (Recommended)

1. Download the latest release ZIP file from [GitHub Releases](https://github.com/itsmattius/wordpress-flysystem-s3/releases)
2. Go to your WordPress admin panel
3. Navigate to `Plugins` → `Add New` → `Upload Plugin`
4. Click `Choose File` and select the downloaded ZIP file
5. Click `Install Now`
6. After installation, click `Activate Plugin`

### Method 2: Manual Installation

1. Download the latest release ZIP file from [GitHub Releases](https://github.com/itsmattius/wordpress-flysystem-s3/releases)
2. Extract the ZIP file
3. Upload the `wordpress-flysystem-s3` folder to `/wp-content/plugins/` directory via FTP or file manager
4. Go to `Plugins` → `Installed Plugins` in WordPress admin
5. Find "WordPress Flysystem S3" and click `Activate`

### Method 3: Install for Development

If you're developing or contributing to the plugin:

```bash
cd wp-content/plugins
git clone https://github.com/itsmattius/wordpress-flysystem-s3.git
cd wordpress-flysystem-s3
composer install
```

### Configure S3 Credentials

After installation and activation, add the following constants to your `wp-config.php` file:

```php
// Required S3 Configuration
define('S3_KEY', 'your-access-key');
define('S3_SECRET', 'your-secret-key');
define('S3_REGION', 'us-east-1'); // Your S3 region
define('S3_BUCKET', 'your-bucket-name');
define('S3_ENDPOINT', 'https://s3.amazonaws.com'); // Or your S3-compatible endpoint

// Optional: Specify a root directory within the bucket
define('S3_ROOT', 'wordpress/media'); // Leave undefined for bucket root
```

**Note:** The plugin will automatically create the necessary database table upon activation.

## Configuration Examples

### AWS S3

```php
define('S3_KEY', 'AKIAIOSFODNN7EXAMPLE');
define('S3_SECRET', 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY');
define('S3_REGION', 'us-east-1');
define('S3_BUCKET', 'my-wordpress-media');
define('S3_ENDPOINT', 'https://s3.amazonaws.com');
```

### MinIO

```php
define('S3_KEY', 'minioadmin');
define('S3_SECRET', 'minioadmin');
define('S3_REGION', 'us-east-1');
define('S3_BUCKET', 'wordpress');
define('S3_ENDPOINT', 'https://minio.example.com');
```

### DigitalOcean Spaces

```php
define('S3_KEY', 'your-spaces-key');
define('S3_SECRET', 'your-spaces-secret');
define('S3_REGION', 'nyc3');
define('S3_BUCKET', 'your-space-name');
define('S3_ENDPOINT', 'https://nyc3.digitaloceanspaces.com');
```

## How It Works

The plugin hooks into WordPress' media upload process:

1. **Upload Detection**: Intercepts files when uploaded through WordPress media uploader
2. **MD5 Hashing**: Generates MD5 hash of the file for deduplication
3. **Remote Upload**: Uploads file to S3 storage with organized directory structure (`/ab/cd/abcd1234...`)
4. **Database Tracking**: Records file mapping in custom database table
5. **URL Rewriting**: Automatically rewrites attachment URLs to point to S3 storage
6. **Smart Deletion**: Handles file deletion with reference counting

### Database Schema

The plugin creates a table `wp_fs_s3_files` with the following structure:

| Column | Type | Description |
|--------|------|-------------|
| `id` | int(10) | Auto-increment primary key |
| `local_file` | varchar(512) | Original WordPress file path |
| `remote_file` | varchar(512) | Full S3 URL |
| `md5` | varchar(32) | MD5 hash of the file |
| `count` | smallint(5) | Reference count for safe deletion |

## Usage

Once installed and configured, the plugin works automatically:

### Uploading Media

1. Go to `Media` → `Add New` in WordPress admin
2. Upload your files as usual
3. Files are automatically uploaded to S3 in the background
4. Media library displays files with S3 URLs

### Accessing Files

All attachment URLs automatically point to your S3 storage:

```php
// Standard WordPress function
$url = wp_get_attachment_url($attachment_id);
// Returns: https://s3.amazonaws.com/your-bucket/ab/cd/abcd1234ef567890.jpg
```

### Deleting Files

When you delete media from WordPress:
- If the file is used once, it's removed from both WordPress and S3
- If the file has multiple references, the reference count is decremented
- Only when the last reference is removed, the file is deleted from S3

## File Organization

Files are stored in S3 with a hierarchical structure based on MD5 hash:

```
bucket/
  └── ab/
      └── cd/
          └── abcd1234ef567890.jpg
```

This structure helps distribute files evenly and improves performance.

## Troubleshooting

### Plugin Not Working

1. Verify all required S3 constants are defined in `wp-config.php`
2. Check S3 credentials are correct and have proper permissions
3. Ensure the S3 bucket exists and is accessible
4. Check WordPress debug log for errors

### Files Not Uploading

1. Verify PHP has write permissions to WordPress upload directory
2. Check S3 bucket permissions allow `PutObject` operation
3. Ensure the endpoint URL is correct for your S3 provider
4. Check for PHP memory limit or timeout issues

### Enable WordPress Debug Mode

Add to `wp-config.php`:

```php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
```

Check `/wp-content/debug.log` for error messages.

## Security Considerations

- Store S3 credentials securely in `wp-config.php` (never commit to version control)
- Use IAM credentials with minimal required permissions:
  - `s3:PutObject` - Upload files
  - `s3:GetObject` - Read files (if private)
  - `s3:DeleteObject` - Delete files
- Consider using environment variables for credentials
- Enable bucket versioning for data protection
- Configure appropriate bucket CORS settings if needed

## Development

To contribute to this plugin or run it in development mode:

```bash
# Clone the repository
git clone https://github.com/itsmattius/wordpress-flysystem-s3.git
cd wordpress-flysystem-s3

# Install dependencies
composer install

# Configure wp-config.php with test S3 credentials
# Activate the plugin in WordPress
```

### Dependencies

- [league/flysystem-aws-s3-v3](https://github.com/thephpleague/flysystem-aws-s3-v3) - Flysystem adapter for AWS S3

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Author

**Mehdi Abedi**
- Email: abedi1667@gmail.com

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

For issues, questions, or contributions, please visit the [GitHub repository](https://github.com/itsmattius/wordpress-flysystem-s3).

## Changelog

### Version 1.0.0
- Initial release
- Automatic media upload to S3
- Support for all media types
- MD5-based deduplication
- Reference counting for safe deletions
- S3-compatible storage support

