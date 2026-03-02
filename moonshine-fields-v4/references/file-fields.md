# File and Editor Fields Reference (MoonShine v4)

## File

`MoonShine\UI\Fields\File` - File upload field.

> Before using, ensure a symbolic link is set up: `php artisan storage:link`

```php
use MoonShine\UI\Fields\File;

File::make('File')
```

> Set `APP_URL` in `.env` to correctly generate file URLs.

### Disk

```php
File::make('File')
    ->disk('public') // default: 'public'
```

### Directory

```php
File::make('File')
    ->dir('docs') // relative to disk root
```

### Allowed Extensions

```php
File::make('File')
    ->allowedExtensions(['pdf', 'doc', 'txt'])
```

### Multiple Upload

```php
File::make('File')
    ->multiple()
```

> When using `multiple()` with a model, add a cast: `'field' => 'json'` or `'field' => 'array'`.

### File Removal

```php
File::make('File')
    ->removable()

// With custom attributes for the remove button
File::make('File')
    ->removable(
        attributes: ['@click.prevent' => '$event.target.closest(`.x-removeable`).remove()']
    )
```

> When deleting a resource, files are also deleted. Mass deletion does NOT auto-delete files.

#### disableDeleteFiles

Delete only the database record, not the actual file:

```php
File::make('File')
    ->removable()
    ->disableDeleteFiles()
```

#### enableDeleteDir

Delete the directory (specified in `dir()`) if it becomes empty:

```php
File::make('File')
    ->dir('docs')
    ->removable()
    ->enableDeleteDir()
```

### Disable Download

```php
File::make('File', 'file')
    ->disableDownload()
```

### Keep Original File Name

```php
File::make('File')
    ->keepOriginalFileName()
```

### Custom File Name

```php
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Str;

File::make('File', 'file')
    ->customName(fn(UploadedFile $file, Field $field) => Str::random(10) . '.' . $file->extension())
```

### Display Names

Change displayed name without altering the file name:

```php
File::make('File', 'file')
    ->names(fn(string $filename, int $index = 0) => 'File ' . $index + 1)
```

### Reorderable (Drag and Drop)

```php
File::make('Files')
    ->reorderable(fn(File $ctx) => "/reorder/" . $ctx->getData()->getKey())
    ->multiple()
```

### Item Attributes

```php
File::make('File', 'file')
    ->itemAttributes(fn(string $filename, int $index = 0) => [
        'style' => 'width: 250px; height: 250px;'
    ])
```

### Dropzone Attributes

```php
File::make('Files')
    ->dropzoneAttributes(fn(File $ctx) => ['class' => 'custom-class'])
    ->multiple()
```

### Helper Methods

```php
// Get remaining values after deletions
$field->getRemainingValues();

// Physically delete files during process
$field->removeExcludedFiles();
```

### Refresh After Apply

File fields auto-refresh preview after apply. Control this behavior:

```php
use MoonShine\UI\Fields\Image;

// Disable auto-refresh
Image::make('Avatar')->disableRefreshAfterApply()

// Custom refresh logic
Image::make('Avatar')->refreshAfterApply(fn(Image $ctx) => $ctx)
```

---

## Image

`MoonShine\UI\Fields\Image` - Image upload with preview. Inherits from File (all File methods available).

```php
use MoonShine\UI\Fields\Image;

Image::make('Thumbnail')
```

### Extra Attributes (Modal Customization)

Customize the modal window that shows the image in preview mode:

```php
use MoonShine\Support\DTOs\FileItemExtra;

Image::make('avatar')
    ->extraAttributes(
        fn(string $filename, int $index): ?FileItemExtra => new FileItemExtra(
            wide: false,   // XL size modal window
            auto: true,    // Window size adjusts to content
            styles: 'width: 250px;'  // Additional image styles in modal
        )
    )
```

### Common Image Usage

```php
Image::make('Photo', 'photo')
    ->disk('public')
    ->dir('photos')
    ->allowedExtensions(['jpg', 'png', 'webp'])
    ->multiple()
    ->removable()
```

---

## Code (Ace Editor)

Requires package installation:

```shell
composer require moonshine/ace
```

> See [moonshine/ace repository](https://github.com/moonshine-software/ace) for full documentation.

---

## Markdown (EasyMDE)

Requires package installation:

```shell
composer require moonshine/easymde
```

> See [moonshine/easymde repository](https://github.com/moonshine-software/easymde) for full documentation.

---

## TinyMCE

Requires package installation:

```shell
composer require moonshine/tinymce
```

> See [moonshine/tinymce repository](https://github.com/moonshine-software/tinymce) for full documentation.

---

## Complete File Upload Example

```php
use MoonShine\UI\Fields\File;
use MoonShine\UI\Fields\Image;

// Simple file upload
File::make('Document', 'document')
    ->disk('public')
    ->dir('documents')
    ->allowedExtensions(['pdf', 'doc', 'docx'])
    ->removable()

// Multiple image upload with reordering
Image::make('Gallery', 'gallery')
    ->disk('public')
    ->dir('gallery')
    ->allowedExtensions(['jpg', 'png', 'webp'])
    ->multiple()
    ->removable()
    ->reorderable(fn(Image $ctx) => "/reorder/" . $ctx->getData()->getKey())

// File with custom naming
File::make('Avatar', 'avatar')
    ->disk('public')
    ->dir('avatars')
    ->customName(fn(UploadedFile $file) => auth()->id() . '.' . $file->extension())
    ->removable()
    ->disableDeleteFiles()
```
