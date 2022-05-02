
## Compress whole directory into zip file using PHP

Let's some directory have files as shown below.

```php
Directory/
    index.html
	picture.jpg
	style.css
```

The following code will be compress the all files avialble in the directory.

```php
<?php
// Get real path for our folder
$rootPath = realpath('folder-to-zip');

// Initialize archive object
$zip = new ZipArchive();
$zip->open('filename.zip', ZipArchive::CREATE | ZipArchive::OVERWRITE);

// Create recursive directory iterator
/** @var SplFileInfo[] $files */
$files = new RecursiveIteratorIterator(
    new RecursiveDirectoryIterator($rootPath),
    RecursiveIteratorIterator::LEAVES_ONLY
);

foreach ($files as $name => $file)
{
    // Skip directories (they would be added automatically)
    if (!$file->isDir())
    {
        // Get real and relative path for current file
        $filePath = $file->getRealPath();
        $relativePath = substr($filePath, strlen($rootPath) + 1);

        // Add current file to archive
        $zip->addFile($filePath, $relativePath);
    }
}

// Zip archive will be created only after closing object
$zip->close();
?>
```
