# Alpine.js + Livewire Integration

## $wire — Access Livewire from Alpine

```blade
<div x-data="{ open: false }">
    <button @click="open = !open">Toggle</button>

    <div x-show="open" x-transition>
        <!-- Call Livewire methods from Alpine -->
        <button @click="$wire.save()">Save via Livewire</button>
        <button @click="$wire.delete({{ $id }})">Delete</button>

        <!-- Access Livewire properties -->
        <span x-text="$wire.name"></span>

        <!-- Two-way bind Alpine to Livewire -->
        <input x-model="$wire.email" />
    </div>
</div>
```

## @entangle — Sync Alpine and Livewire State

```blade
<div x-data="{ showFilters: $wire.entangle('showFilters') }">
    <button @click="showFilters = !showFilters">
        Filters
    </button>

    <div x-show="showFilters" x-transition>
        <!-- Filter controls -->
    </div>
</div>
```

```php
// Component
class ProductList extends Component
{
    public bool $showFilters = false;
}
```

## JavaScript Interop

```blade
{{-- Dispatch browser events from Livewire --}}
<div
    x-data="{ message: '' }"
    @notify.window="message = $event.detail.message; setTimeout(() => message = '', 3000)"
>
    <div x-show="message" x-text="message" class="bg-green-100 p-2 rounded"></div>
</div>
```

```php
// In Livewire component
$this->dispatch('notify', message: 'Item saved successfully!');
```

## Third-Party JS Libraries

```blade
<div
    x-data="chartComponent(@js($chartData))"
    x-init="initChart()"
    wire:ignore {{-- Prevent Livewire from touching this DOM --}}
>
    <canvas x-ref="canvas"></canvas>
</div>

@script
<script>
    Alpine.data('chartComponent', (initialData) => ({
        chart: null,
        initChart() {
            this.chart = new Chart(this.$refs.canvas, {
                type: 'line',
                data: initialData,
            });
        },
    }));
</script>
@endscript
```

## Key Patterns

- Use `$wire.method()` to call Livewire from Alpine (no round-trip for Alpine state)
- Use `$wire.entangle()` to sync state bidirectionally
- Use `wire:ignore` on DOM managed by third-party JS
- Use `@script` / `@endscript` for component-scoped JS
- Use `x-transition` for client-side animations (faster than Livewire round-trip)
