# 📊 Pemilihan Chart Library untuk Livewire 3

Dokumentasi ini menjelaskan opsi library chart yang bisa dipakai bersama **Livewire 3**. Fokus kita adalah kemudahan integrasi dan dukungan **reactive update** menggunakan `dispatch()` (bukan `dispatchBrowserEvent`).

---

## 📝 Tabel Perbandingan

| Library / Package | Engine Chart | Integrasi Livewire | Reactive Support | Tingkat Ribet |
|---|---|---|---|---|
| **Livewire Charts (asantibanez)** | ApexCharts | Native Livewire Component | ✅ Full reactive (otomatis update) | ⭐⭐ |
| **Larapex Charts** | ApexCharts | Laravel wrapper (Blade + Service) | ❌ Tidak otomatis reactive (perlu trik update) | ⭐ |
| **ApexCharts.js langsung** | ApexCharts (JS murni) | Manual (`wire:ignore` + JS) | ✅ Bisa reactive (manual dispatch) | ⭐⭐⭐ |

---

## 🔑 Opsi Library

### 1. **Livewire Charts (asantibanez/livewire-charts)**
- Engine: **ApexCharts**
- Integrasi: Native Livewire Component
- Reactive: ✅ Full support (otomatis update data)
- Tingkat Ribet: ⭐⭐
- 📦 Install:
```bash
composer require asantibanez/livewire-charts
```

**Contoh Component:**
```php
use Asantibanez\LivewireCharts\Models\ColumnChartModel;

class IncomeExpenseChart extends \Livewire\Component
{
    public $chart;

    public function mount()
    {
        $this->chart = (new ColumnChartModel())
            ->setTitle('Pemasukan vs Pengeluaran')
            ->addColumn('Pemasukan', 500000, '#4CAF50')
            ->addColumn('Pengeluaran', 300000, '#F44336');
    }

    public function updateChart()
    {
        $this->dispatch('updateChart', chart: (new ColumnChartModel())
            ->setTitle('Update Data')
            ->addColumn('Pemasukan', rand(100000, 700000), '#4CAF50')
            ->addColumn('Pengeluaran', rand(100000, 700000), '#F44336'));
    }

    public function render()
    {
        return view('livewire.income-expense-chart');
    }
}
```

**Blade View:**
```blade
<div>
    <button wire:click="updateChart">Update Chart</button>

    <livewire:livewire-column-chart
        key="chart"
        :column-chart-model="$chart"
    />
</div>
```

---

### 2. **Larapex Charts**
- Engine: **ApexCharts**
- Integrasi: Laravel wrapper (Blade + Controller/Service)
- Reactive: ❌ Tidak otomatis, perlu event manual.
- Tingkat Ribet: ⭐ (paling mudah setup, tapi kurang reactive)
- 📦 Install:
```bash
composer require arielmejiadev/larapex-charts
```

**Contoh Service Chart:**
```php
use ArielMejiaDev\LarapexCharts\LarapexChart;

class IncomeExpenseChartService
{
    public function build(): \ArielMejiaDev\LarapexCharts\BarChart
    {
        return (new LarapexChart)->barChart()
            ->setTitle('Pemasukan vs Pengeluaran')
            ->addData('Pemasukan', [500000])
            ->addData('Pengeluaran', [300000])
            ->setColors(['#4CAF50', '#F44336'])
            ->setXAxis(['Data']);
    }
}
```

**Blade View:**
```blade
<div>
    {!! $chart->container() !!}
</div>

@push('scripts')
    {!! $chart->script() !!}
@endpush
```

> ⚠️ Untuk reactive, harus dikombinasi dengan Livewire `dispatch()` manual → update data chart via AJAX/Livewire, lalu re-render chart.

---

### 3. **ApexCharts.js langsung**
- Engine: **ApexCharts (JS murni)**
- Integrasi: Manual dengan `wire:ignore`
- Reactive: ✅ Bisa, tapi butuh `dispatch()` dan JS handler
- Tingkat Ribet: ⭐⭐⭐ (perlu coding JS sendiri)

**Blade View:**
```blade
<div wire:ignore>
    <div id="chart"></div>
</div>
```

**JavaScript:**
```js
document.addEventListener("livewire:init", () => {
    let chart = new ApexCharts(document.querySelector("#chart"), {
        chart: { type: 'bar' },
        series: [{ name: 'Pemasukan', data: [500000] }],
        xaxis: { categories: ['Data'] }
    });
    chart.render();

    Livewire.on("updateChart", (data) => {
        chart.updateSeries(data.series);
    });
});
```

**Livewire Component:**
```php
$this->dispatch('updateChart', series: [
    ["name" => "Pemasukan", "data" => [rand(100000, 700000)]],
    ["name" => "Pengeluaran", "data" => [rand(100000, 700000)]],
]);
```

---

## 🎯 Rekomendasi
- **Paling mudah & native Livewire 3** → `asantibanez/livewire-charts`
- **Paling simpel setup (non-reactive)** → `larapex-charts`
- **Paling fleksibel (tapi ribet)** → ApexCharts.js manual
