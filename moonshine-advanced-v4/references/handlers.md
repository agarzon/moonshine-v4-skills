# Handlers Reference (MoonShine v4)

## Overview

Handlers in MoonShine are reusable action classes that allow you to add custom actions to resources without creating controllers. They provide automatic error handling, integration with the UI through automatic button generation, and are automatically displayed in the interface after registration.

**V4 CHANGE**: The base class is `MoonShine\Crud\Handlers\Handler` (moved to the Crud package). The `toast()` global helper replaces `MoonShineUI::toast()`.

---

## Creating a Handler

```shell
php artisan moonshine:handler MyCustomHandler
```

Signature:

```
moonshine:handler {className?} {--base-dir=} {--base-namespace=}
```

After executing the command, a Handler class will be created in `app/MoonShine/Handlers/`.

---

## Handler Structure

```php
namespace App\MoonShine\Handlers;

use MoonShine\Contracts\UI\ActionButtonContract;
use MoonShine\Crud\Handlers\Handler;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Exceptions\ActionButtonException;
use Symfony\Component\HttpFoundation\Response;

class MyCustomHandler extends Handler
{
    /**
     * @throws ActionButtonException
     */
    public function handle(): Response
    {
        if (! $this->hasResource()) {
            throw new ActionButtonException('Resource is required for action');
        }

        if ($this->isQueue()) {
            // Dispatch a job here

            toast(
                __('moonshine::ui.resource.queued')
            );

            return back();
        }

        self::process();

        return back();
    }

    public static function process()
    {
        // Your logic here
    }

    public function getButton(): ActionButtonContract
    {
        return ActionButton::make($this->getLabel(), $this->getUrl());
    }
}
```

---

## Registering a Handler

To register a Handler on the `IndexPage`, override the `handlers()` method in your index page class:

```php
namespace App\MoonShine\Resources\Post\Pages;

use MoonShine\Support\ListOf;
use MoonShine\Laravel\Pages\Crud\IndexPage;

class PostIndexPage extends IndexPage
{
    protected function handlers(): ListOf
    {
        return parent::handlers()->add(new MyCustomHandler());
    }
}
```

After registration, a button for launching the Handler will automatically appear on the right side of the index page.

---

## Handler Interaction

Handler is closely integrated with the resource and has access to:

- The current resource via `$this->getResource()`
- Queueing capabilities via `$this->isQueue()` and the `WithQueue` trait
- Notification system via `notifyUsers(array|Closure $ids)` for setting notification recipients
- Button modification via `modifyButton(Closure $callback)`

---

## Custom Import Handler Example

```php
namespace App\MoonShine\Handlers;

use MoonShine\Contracts\UI\ActionButtonContract;
use MoonShine\Crud\Handlers\Handler;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Exceptions\ActionButtonException;
use Symfony\Component\HttpFoundation\Response;
use Illuminate\Support\Facades\Storage;

class CsvImportHandler extends Handler
{
    public function handle(): Response
    {
        if (! $this->hasResource()) {
            throw new ActionButtonException('Resource is required');
        }

        $resource = $this->getResource();

        if ($this->isQueue()) {
            // Dispatch import job
            \App\Jobs\ImportCsvJob::dispatch(
                $resource->getModel()::class,
                'imports/data.csv'
            );

            toast(__('moonshine::ui.resource.queued'));
            return back();
        }

        // Direct import
        self::process($resource->getModel()::class, 'imports/data.csv');

        toast('Import completed successfully');
        return back();
    }

    public static function process(string $modelClass, string $filePath): void
    {
        $csv = Storage::get($filePath);
        $rows = array_map('str_getcsv', explode("\n", $csv));
        $headers = array_shift($rows);

        foreach ($rows as $row) {
            if (count($row) === count($headers)) {
                $data = array_combine($headers, $row);
                $modelClass::create($data);
            }
        }
    }

    public function getButton(): ActionButtonContract
    {
        return ActionButton::make('Import CSV', $this->getUrl())
            ->icon('arrow-up-tray')
            ->primary();
    }
}
```

---

## Custom Export Handler Example

```php
namespace App\MoonShine\Handlers;

use MoonShine\Contracts\UI\ActionButtonContract;
use MoonShine\Crud\Handlers\Handler;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Exceptions\ActionButtonException;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\StreamedResponse;

class CsvExportHandler extends Handler
{
    public function handle(): Response
    {
        if (! $this->hasResource()) {
            throw new ActionButtonException('Resource is required');
        }

        $resource = $this->getResource();

        if ($this->isQueue()) {
            \App\Jobs\ExportCsvJob::dispatch(
                $resource->getModel()::class
            );

            toast(__('moonshine::ui.resource.queued'));
            return back();
        }

        return self::process($resource->getModel()::class);
    }

    public static function process(string $modelClass): StreamedResponse
    {
        $items = $modelClass::all();
        $headers = array_keys($items->first()->toArray());

        return response()->streamDownload(function () use ($items, $headers) {
            $file = fopen('php://output', 'w');
            fputcsv($file, $headers);

            foreach ($items as $item) {
                fputcsv($file, $item->toArray());
            }

            fclose($file);
        }, 'export.csv', [
            'Content-Type' => 'text/csv',
        ]);
    }

    public function getButton(): ActionButtonContract
    {
        return ActionButton::make('Export CSV', $this->getUrl())
            ->icon('arrow-down-tray')
            ->success();
    }
}
```

---

## Handler with Queue and Notifications

```php
namespace App\MoonShine\Handlers;

use MoonShine\Contracts\UI\ActionButtonContract;
use MoonShine\Crud\Handlers\Handler;
use MoonShine\UI\Components\ActionButton;
use MoonShine\UI\Exceptions\ActionButtonException;
use Symfony\Component\HttpFoundation\Response;

class BatchProcessHandler extends Handler
{
    public function handle(): Response
    {
        if (! $this->hasResource()) {
            throw new ActionButtonException('Resource is required');
        }

        // Set users who will receive notifications after processing
        $this->notifyUsers(fn () => [auth()->id()]);

        if ($this->isQueue()) {
            \App\Jobs\BatchProcessJob::dispatch(
                $this->getResource()->getModel()::class
            );

            toast(__('moonshine::ui.resource.queued'));
            return back();
        }

        self::process();

        toast('Batch processing completed');
        return back();
    }

    public static function process(): void
    {
        // Batch processing logic
    }

    public function getButton(): ActionButtonContract
    {
        return ActionButton::make('Batch Process', $this->getUrl())
            ->icon('cog')
            ->warning()
            ->withConfirm('Are you sure?', 'This will process all items.');
    }
}
```

---

## Registering Multiple Handlers

```php
namespace App\MoonShine\Resources\Post\Pages;

use App\MoonShine\Handlers\CsvImportHandler;
use App\MoonShine\Handlers\CsvExportHandler;
use App\MoonShine\Handlers\BatchProcessHandler;
use MoonShine\Support\ListOf;
use MoonShine\Laravel\Pages\Crud\IndexPage;

class PostIndexPage extends IndexPage
{
    protected function handlers(): ListOf
    {
        return parent::handlers()
            ->add(new CsvImportHandler())
            ->add(new CsvExportHandler())
            ->add(new BatchProcessHandler());
    }
}
```

---

## Modifying Handler Button

You can customize the appearance and behavior of the handler's button:

```php
class MyHandler extends Handler
{
    public function getButton(): ActionButtonContract
    {
        return ActionButton::make($this->getLabel(), $this->getUrl())
            ->icon('arrow-path')
            ->success()
            ->withConfirm(
                'Confirm action',
                'Are you sure you want to proceed?'
            );
    }
}
```

Or modify it during registration:

```php
protected function handlers(): ListOf
{
    $handler = new MyCustomHandler();
    $handler->modifyButton(function (ActionButtonContract $button) {
        return $button->icon('wrench')->warning();
    });

    return parent::handlers()->add($handler);
}
```

---

## Handler vs AsyncMethod

| Feature | Handler | AsyncMethod |
|---------|---------|-------------|
| Location | Separate class in `app/MoonShine/Handlers/` | Method on resource or page |
| UI | Auto-generates button on index page | Manual button creation |
| Queue | Built-in queue support | Manual queue dispatch |
| Notifications | Built-in notification support | Manual notification |
| Best for | Reusable, complex actions | Simple inline actions |

Use Handlers for reusable, complex actions that need queue support and notification recipients. Use `#[AsyncMethod]` for simpler inline actions on specific pages or resources.
