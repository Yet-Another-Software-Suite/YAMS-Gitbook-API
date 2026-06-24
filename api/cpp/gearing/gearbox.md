# GearBox

**Namespace:** `yams::gearing`
**Java equivalent:** [Java GearBox](../../java/gearing/gearbox.md)

`GearBox` models a multi-stage gear reduction. Semantically represents meshed gears. Use `Sprocket` for chain or belt drives.

---

## Constructors

```cpp
explicit GearBox(double reductionStage)
explicit GearBox(std::initializer_list<double> reductionStages)
explicit GearBox(const std::vector<double>& reductionStages)
explicit GearBox(const std::vector<std::string>& stages)  // "IN:OUT" format
```

Each element of `reductionStages` is the ratio for one stage (input teeth / output teeth). Strings must be in `"IN:OUT"` format.

---

## Factory Methods

```cpp
static GearBox FromReductionStages(std::initializer_list<double> stages)
static GearBox FromStages(std::initializer_list<std::string_view> stages)  // "IN:OUT"
static GearBox FromTeeth(std::initializer_list<int> teeth)  // alternating driver, driven
```

`FromTeeth` takes tooth counts in alternating driver/driven order. Two values produce a single-stage reduction; four values produce two stages, etc.

---

## Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetInputToOutputConversionFactor() const` | `double` | Mechanism rotations per motor rotation (OUT/IN) |
| `GetOutputToInputConversionFactor() const` | `double` | Motor rotations per mechanism rotation (IN/OUT) |
| `Times(double x)` | `GearBox&` | Multiplies the reduction by `x`. Returns `*this` for chaining. |
| `Div(double x)` | `GearBox&` | Divides the reduction by `x`. Returns `*this` for chaining. |

---

## Exceptions

| Exception | Thrown When |
|-----------|-------------|
| `exceptions::NoStagesGivenException` | Stage list is empty |
| `exceptions::InvalidStageGivenException` | A string stage is not in `"IN:OUT"` format |

---

## Example

```cpp
// All three equivalent: 12T driver, 62T driven = 5.17:1
auto g1 = gearing::GearBox::FromTeeth({12, 62});
auto g2 = gearing::GearBox::FromStages({"12:62"});
auto g3 = gearing::GearBox::FromReductionStages({5.1667});

// Two-stage: (12→48) × (16→64) = 4 × 4 = 16:1
auto g4 = gearing::GearBox::FromTeeth({12, 48, 16, 64});
```

{% hint style="info" %}
`GetInputToOutputConversionFactor()` returns OUT/IN (less than 1 for reductions). `GetOutputToInputConversionFactor()` returns IN/OUT (greater than 1 for reductions). Pass `GearBox` to `MechanismGearing` for use with `SmartMotorControllerConfig`.
{% endhint %}
