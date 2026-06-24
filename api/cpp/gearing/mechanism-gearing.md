# MechanismGearing

**Namespace:** `yams::gearing`
**Java equivalent:** [Java MechanismGearing](../../java/gearing/mechanism-gearing.md)

`MechanismGearing` is the composite gearing type accepted by `SmartMotorControllerConfig::WithMotorGearing()`. It may combine a `GearBox` with an optional `Sprocket`.

---

## Constants

```cpp
static const MechanismGearing kOne;  // 1:1 direct drive
```

---

## Constructors

```cpp
explicit MechanismGearing(double reductionRatio)
explicit MechanismGearing(const GearBox& gearBox)
MechanismGearing(const GearBox& gearBox, const Sprocket& sprockets)
```

---

## Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetRotorToMechanismRatio() const` | `double` | Motor rotations per mechanism rotation (IN/OUT) |
| `GetMechanismToRotorRatio() const` | `double` | Mechanism rotations per motor rotation (OUT/IN) |
| `Div(double i)` | `MechanismGearing&` | Divides the overall ratio by `i`. Returns `*this` for chaining. |

---

## Example

```cpp
auto gearing = gearing::MechanismGearing{
    gearing::GearBox::FromTeeth({14, 72}),    // 5.14:1
    gearing::Sprocket::FromStages({"18:24"})  // 1.33:1
};
// Total: ~6.85:1

config.WithMotorGearing(gearing);
```

{% hint style="info" %}
`GetRotorToMechanismRatio()` returns the value WPILib expects for the `rotorToMechanismRatio` field of motor controller configs (motor rotations per mechanism rotation, i.e., IN/OUT — greater than 1 for reductions).
{% endhint %}
