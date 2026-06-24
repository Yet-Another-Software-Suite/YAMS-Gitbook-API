# Sprocket

**Namespace:** `yams::gearing`
**Java equivalent:** [Java Sprocket](../../java/gearing/sprocket.md)

`Sprocket` models a chain or belt drive stage. The API is identical to `GearBox`; the distinction is semantic. Use `GearBox` for meshed gears.

---

## Constructors

```cpp
explicit Sprocket(double reductionStage)
explicit Sprocket(std::initializer_list<double> reductionStages)
explicit Sprocket(const std::vector<double>& reductionStages)
explicit Sprocket(const std::vector<std::string>& stages)  // "IN:OUT" format
```

---

## Factory Methods

```cpp
static Sprocket FromStages(std::initializer_list<std::string_view> stages)  // "IN:OUT"
```

{% hint style="info" %}
`Sprocket` does not provide `FromTeeth`. Use `FromStages` with tooth-count strings (e.g., `"18:36"`) to express the ratio directly.
{% endhint %}

---

## Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetInputToOutputConversionFactor() const` | `double` | Mechanism rotations per motor rotation (OUT/IN) |
| `GetOutputToInputConversionFactor() const` | `double` | Motor rotations per mechanism rotation (IN/OUT) |
| `Times(double x)` | `Sprocket&` | Multiplies the reduction by `x`. Returns `*this` for chaining. |
| `Div(double x)` | `Sprocket&` | Divides the reduction by `x`. Returns `*this` for chaining. |

---

## Exceptions

| Exception | Thrown When |
|-----------|-------------|
| `exceptions::NoStagesGivenException` | Stage list is empty |
| `exceptions::InvalidStageGivenException` | A string stage is not in `"IN:OUT"` format |

---

## Example

```cpp
// 18T driving sprocket, 36T driven sprocket = 2:1
auto s = gearing::Sprocket::FromStages({"18:36"});

// Multi-stage belt drive
auto s2 = gearing::Sprocket::FromStages({"18:36", "12:24"});
```
