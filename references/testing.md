# Livewire Testing Patterns

## Basic Component Testing

```php
use Livewire\Livewire;

it('renders the user table', function () {
    Livewire::test(UserTable::class)
        ->assertStatus(200)
        ->assertSee('Users');
});
```

## Property & Method Testing

```php
it('filters users by search', function () {
    User::factory()->create(['name' => 'John Doe']);
    User::factory()->create(['name' => 'Jane Smith']);

    Livewire::test(UserTable::class)
        ->set('search', 'John')
        ->assertSee('John Doe')
        ->assertDontSee('Jane Smith');
});

it('sorts users by column', function () {
    Livewire::test(UserTable::class)
        ->call('sort', 'name')
        ->assertSet('sortBy', 'name')
        ->assertSet('sortDir', 'asc')
        ->call('sort', 'name')
        ->assertSet('sortDir', 'desc');
});
```

## Form Validation Testing

```php
it('validates required fields', function () {
    Livewire::test(CreateUser::class)
        ->set('name', '')
        ->set('email', '')
        ->call('save')
        ->assertHasErrors(['name' => 'required', 'email' => 'required']);
});

it('creates a user with valid data', function () {
    Livewire::test(CreateUser::class)
        ->set('name', 'Test User')
        ->set('email', 'test@example.com')
        ->set('password', 'password123')
        ->set('password_confirmation', 'password123')
        ->call('save')
        ->assertHasNoErrors()
        ->assertDispatched('user-created');

    expect(User::where('email', 'test@example.com')->exists())->toBeTrue();
});
```

## Event Testing

```php
it('dispatches event on save', function () {
    Livewire::test(CreateUser::class)
        ->fill(['name' => 'Test', 'email' => 'a@b.com', 'password' => '12345678', 'password_confirmation' => '12345678'])
        ->call('save')
        ->assertDispatched('user-created');
});

it('listens for refresh event', function () {
    Livewire::test(UserTable::class)
        ->dispatch('user-created')
        ->assertStatus(200);
});
```

## File Upload Testing

```php
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

it('uploads an avatar', function () {
    Storage::fake('public');

    Livewire::test(AvatarUpload::class)
        ->set('photo', UploadedFile::fake()->image('avatar.jpg'))
        ->call('save')
        ->assertHasNoErrors();

    Storage::disk('public')->assertExists('avatars/' . /* filename */);
});
```

## URL Query String Testing

```php
it('syncs search to URL', function () {
    Livewire::withQueryParams(['search' => 'hello'])
        ->test(UserTable::class)
        ->assertSet('search', 'hello');
});
```
