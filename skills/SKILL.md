---
name: laravel-livewire
description: "Laravel Livewire 3 reactive components, real-time validation, file uploads, lazy loading, and Alpine.js integration. Use when building Livewire components, handling reactive properties, implementing real-time search/filtering, creating modals, working with wire:model, wire:click, Livewire forms, Livewire pagination, Volt single-file components, or any server-driven reactive UI without a JS framework. Triggers on tasks involving Livewire component creation, reactive state, lifecycle hooks, computed properties, lazy components, teleport, or Livewire testing. Also use when the user mentions Livewire, TALL stack, Alpine.js with Laravel, or wants interactivity without Vue/React. Use PROACTIVELY whenever Livewire or TALL stack is mentioned."
compatible_agents:
  - Claude Code
  - Cursor
  - Windsurf
  - Copilot
tags:
  - laravel
  - livewire
  - alpine
  - tall-stack
  - reactive
  - components
  - php
---

# Laravel Livewire 3

Build reactive, dynamic interfaces using PHP components. No JavaScript framework needed — Livewire handles reactivity via server round-trips with automatic DOM diffing.

## Component Basics

```bash
php artisan make:livewire UserTable
# Creates: app/Livewire/UserTable.php + resources/views/livewire/user-table.blade.php
```

### Component Class

```php
namespace App\Livewire;

use App\Models\User;
use Livewire\Component;
use Livewire\WithPagination;
use Livewire\Attributes\Url;
use Livewire\Attributes\Computed;

class UserTable extends Component
{
    use WithPagination;

    #[Url]
    public string $search = '';

    #[Url]
    public string $sortBy = 'created_at';

    #[Url]
    public string $sortDir = 'desc';

    public function updatedSearch(): void
    {
        $this->resetPage();
    }

    public function sort(string $column): void
    {
        if ($this->sortBy === $column) {
            $this->sortDir = $this->sortDir === 'asc' ? 'desc' : 'asc';
        } else {
            $this->sortBy = $column;
            $this->sortDir = 'asc';
        }
    }

    #[Computed]
    public function users()
    {
        return User::query()
            ->when($this->search, fn ($q) => $q
                ->where('name', 'like', "%{$this->search}%")
                ->orWhere('email', 'like', "%{$this->search}%")
            )
            ->orderBy($this->sortBy, $this->sortDir)
            ->paginate(15);
    }

    public function render()
    {
        return view('livewire.user-table');
    }
}
```

### Component View

```blade
<div>
    <input wire:model.live.debounce.300ms="search" placeholder="Search users..." />

    <table>
        <thead>
            <tr>
                <th wire:click="sort('name')" class="cursor-pointer">Name</th>
                <th wire:click="sort('email')" class="cursor-pointer">Email</th>
                <th wire:click="sort('created_at')" class="cursor-pointer">Joined</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($this->users as $user)
                <tr wire:key="{{ $user->id }}">
                    <td>{{ $user->name }}</td>
                    <td>{{ $user->email }}</td>
                    <td>{{ $user->created_at->diffForHumans() }}</td>
                </tr>
            @endforeach
        </tbody>
    </table>

    {{ $this->users->links() }}
</div>
```

## Forms & Validation

```php
namespace App\Livewire;

use App\Models\User;
use Livewire\Component;
use Livewire\Attributes\Validate;

class CreateUser extends Component
{
    #[Validate('required|min:2|max:255')]
    public string $name = '';

    #[Validate('required|email|unique:users,email')]
    public string $email = '';

    #[Validate('required|min:8|confirmed')]
    public string $password = '';

    public string $password_confirmation = '';

    public function save(): void
    {
        $validated = $this->validate();

        User::create([
            ...$validated,
            'password' => bcrypt($validated['password']),
        ]);

        $this->reset();
        $this->dispatch('user-created');
        session()->flash('success', 'User created successfully.');
    }

    public function render()
    {
        return view('livewire.create-user');
    }
}
```

```blade
<form wire:submit="save">
    <div>
        <input wire:model.blur="name" placeholder="Name" />
        @error('name') <span class="text-red-500">{{ $message }}</span> @enderror
    </div>

    <div>
        <input wire:model.blur="email" type="email" placeholder="Email" />
        @error('email') <span class="text-red-500">{{ $message }}</span> @enderror
    </div>

    <div>
        <input wire:model="password" type="password" placeholder="Password" />
        @error('password') <span class="text-red-500">{{ $message }}</span> @enderror
    </div>

    <div>
        <input wire:model="password_confirmation" type="password" placeholder="Confirm" />
    </div>

    <button type="submit" wire:loading.attr="disabled">
        <span wire:loading.remove>Save</span>
        <span wire:loading>Saving...</span>
    </button>
</form>
```

## Wire Model Modifiers

```blade
wire:model="name"                  {{-- Updates on component dehydration only --}}
wire:model.live="search"           {{-- Updates on every keystroke --}}
wire:model.live.debounce.300ms="q" {{-- Debounced live updates --}}
wire:model.blur="email"            {{-- Updates on blur (leaving field) --}}
wire:model.change="role"           {{-- Updates on change event --}}
wire:model.number="price"          {{-- Cast to number --}}
wire:model.boolean="active"        {{-- Cast to boolean --}}
```

## Actions & Events

```php
// Dispatch browser events
$this->dispatch('close-modal');
$this->dispatch('notify', message: 'Saved!');

// Dispatch to specific component
$this->dispatch('refreshList')->to(UserTable::class);

// Listen for events
#[\Livewire\Attributes\On('user-created')]
public function handleUserCreated(): void
{
    // React to event
}
```

```blade
{{-- Listen in Alpine --}}
<div x-on:close-modal.window="open = false">

{{-- Confirm before action --}}
<button wire:click="delete({{ $id }})" wire:confirm="Are you sure?">Delete</button>
```

## File Uploads

```php
use Livewire\WithFileUploads;
use Livewire\Attributes\Validate;

class AvatarUpload extends Component
{
    use WithFileUploads;

    #[Validate('image|max:2048')]
    public $photo;

    public function save(): void
    {
        $this->validate();
        $path = $this->photo->store('avatars', 'public');
        auth()->user()->update(['avatar' => $path]);
    }
}
```

```blade
<input type="file" wire:model="photo" />
@if ($photo)
    <img src="{{ $photo->temporaryUrl() }}" class="w-20 h-20 rounded-full" />
@endif
<div wire:loading wire:target="photo">Uploading...</div>
```

## Lazy Loading & Performance

```blade
{{-- Lazy load heavy components --}}
<livewire:heavy-dashboard lazy />

{{-- Placeholder while loading --}}
{{-- In HeavyDashboard component: --}}
public function placeholder(): string
{
    return '<div class="animate-pulse h-64 bg-gray-200 rounded"></div>';
}

{{-- Wire:key for efficient list rendering --}}
@foreach ($items as $item)
    <livewire:item-row :item="$item" wire:key="{{ $item->id }}" />
@endforeach
```

## Volt (Single-File Components)

```php
// resources/views/livewire/counter.blade.php
<?php
use function Livewire\Volt\{state, computed};

state(['count' => 0]);

$increment = fn () => $this->count++;

?>

<div>
    <span>{{ $count }}</span>
    <button wire:click="increment">+</button>
</div>
```

## Key Rules

1. Always use `wire:key` on looped elements — without it, DOM diffing breaks on reorder
2. Always use `#[Computed]` for derived data — prevents re-querying on every render
3. Always use `wire:model.live.debounce` for search inputs — `.live` alone fires too many requests
4. Always use `#[Url]` for filterable/searchable properties — enables shareable URLs and back button
5. Never store large objects in public properties — they serialize on every request
6. Always use `wire:loading` states — users need feedback during server round-trips
7. Use `lazy` attribute on heavy components — loads after initial page render
8. Always call `$this->resetPage()` when filters change — prevents empty page results
9. Use `wire:model.blur` for validation-heavy fields — validates on blur, not every keystroke
10. Always use Livewire's `WithFileUploads` trait — never handle uploads manually
