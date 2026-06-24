# LQRController

**Namespace:** `yams::math`
**Java equivalent:** [Java LQRController](../../java/math/lqr-controller.md)

`LQRController` wraps a `LinearSystemLoop` and exposes a unified `Calculate` interface for arm, elevator, and flywheel mechanisms.

---

## Constructor

```cpp
explicit LQRController(LQRConfig config)
```

---

## Methods

### Configuration

| Method | Signature | Description |
|--------|-----------|-------------|
| `UpdateConfig` | `void UpdateConfig(LQRConfig config)` | Replace the internal system loop at runtime |
| `GetType` | `LQRConfig::LQRType GetType() const` | Returns the configured mechanism type |
| `GetConfig` | `const std::optional<LQRConfig>& GetConfig() const` | Returns the config if constructed from `LQRConfig`, otherwise empty |

### Reset

| Method | Signature | Description |
|--------|-----------|-------------|
| `Reset` | `void Reset(units::radian_t angle, units::radians_per_second_t velocity)` | Reset the Kalman observer state for arm or flywheel |
| `Reset` | `void Reset(units::meter_t distance, units::meters_per_second_t velocity)` | Reset the Kalman observer state for elevator |

### Calculate

| Overload | Signature | Description |
|----------|-----------|-------------|
| Arm | `units::volt_t Calculate(units::radian_t measured, units::radian_t position, units::radians_per_second_t velocity)` | Compute output voltage for arm control |
| Elevator | `units::volt_t Calculate(units::meter_t measured, units::meter_t position, units::meters_per_second_t velocity)` | Compute output voltage for elevator control |
| Flywheel (angular) | `units::volt_t Calculate(units::radians_per_second_t measured, units::radians_per_second_t velocity)` | Compute output voltage for flywheel angular velocity control |
| Flywheel (linear) | `units::volt_t Calculate(units::meters_per_second_t measured, units::meters_per_second_t velocity)` | Compute output voltage for flywheel surface speed control |

---

## Example

```cpp
math::LQRConfig cfg;
cfg.WithType(math::LQRConfig::LQRType::ARM)
   .WithArmSystem(frc::DCMotor::Falcon500(1), 0.5, 50.0)
   .WithQElems({0.01745, 0.08727})
   .WithRElems({12.0})
   .WithStateStdDevs({0.015, 0.17})
   .WithMeasurementStdDevs({0.01})
   .WithMaxVoltage(12_V);

math::LQRController controller{cfg};

// In Periodic():
units::radian_t measured = GetArmAngle();
units::volt_t output = controller.Calculate(
    measured,
    targetAngle,
    targetVelocity
);
motor.SetVoltage(output);
```

{% hint style="info" %}
Call `Reset` whenever the mechanism is re-enabled or its state is known (e.g., at initialization) to prevent the Kalman filter from tracking stale state.
{% endhint %}
