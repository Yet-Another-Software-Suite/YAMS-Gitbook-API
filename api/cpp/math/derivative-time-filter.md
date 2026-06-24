# DerivativeTimeFilter

**Namespace:** `yams::math`
**Java equivalent:** [Java DerivativeTimeFilter](../../java/math/derivative-time-filter.md)

`DerivativeTimeFilter` computes a numerical derivative of a signal and applies a debounce filter to suppress transient spikes.

---

## Constructors

```cpp
DerivativeTimeFilter(double initial, units::second_t debouncerPeriod)
explicit DerivativeTimeFilter(units::second_t debouncerPeriod)  // initial = 0.0
```

| Parameter | Description |
|-----------|-------------|
| `initial` | Initial value of the signal. Defaults to `0.0` if omitted. |
| `debouncerPeriod` | Duration the debounce filter holds a spike before passing it through. |

---

## Methods

```cpp
double Derivative(double current, units::second_t dt)
double Derivative(double current)
```

| Overload | Description |
|----------|-------------|
| `Derivative(current, dt)` | Computes derivative using an explicit time step `dt`. |
| `Derivative(current)` | Computes derivative using the FPGA timestamp; `dt` is measured since the last call. |

Returns the filtered derivative value.

---

## Example

```cpp
math::DerivativeTimeFilter filter{20_ms};

// In periodic (called every 20 ms):
double velocity = filter.Derivative(encoder.GetPosition());
```

{% hint style="info" %}
Use the explicit `dt` overload in simulation or when calling at non-uniform intervals. Use the FPGA overload in robot periodic methods where real wall-clock time is authoritative.
{% endhint %}
