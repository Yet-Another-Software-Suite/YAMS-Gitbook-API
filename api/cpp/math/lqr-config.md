# LQRConfig

**Namespace:** `yams::math`
**Java equivalent:** [Java LQRConfig](../../java/math/lqr-config.md)

`LQRConfig` is a builder for `LinearSystemLoop` objects consumed by `LQRController`.

---

## Enum: `LQRType`

Defined inside `LQRConfig`.

| Value | Description |
|-------|-------------|
| `FLYWHEEL` | Single-state system (angular velocity) |
| `ARM` | Two-state system (angle, angular velocity) |
| `ELEVATOR` | Two-state system (position, velocity) |

---

## Type Aliases

```cpp
using Loop1 = frc::LinearSystemLoop<1, 1, 1>;  // Used for FLYWHEEL
using Loop2 = frc::LinearSystemLoop<2, 1, 1>;  // Used for ARM and ELEVATOR
```

---

## Builder Methods

All builder methods return `LQRConfig&` for chaining.

| Method | Description |
|--------|-------------|
| `WithType(LQRType type)` | Set the mechanism type |
| `WithFlywheelSystem(const frc::DCMotor& motor, double momentOfInertia, double gearing)` | Configure the flywheel linear system |
| `WithArmSystem(const frc::DCMotor& motor, double momentOfInertia, double gearing)` | Configure the arm linear system |
| `WithElevatorSystem(const frc::DCMotor& motor, double mass, double drumRadius, double gearing)` | Configure the elevator linear system |
| `WithQElems(std::initializer_list<double> qElems)` | State cost weights (Q matrix diagonal) |
| `WithRElems(std::initializer_list<double> rElems)` | Control effort cost weights (R matrix diagonal) |
| `WithStateStdDevs(std::initializer_list<double> stateStdDevs)` | Model noise standard deviations (Kalman Q) |
| `WithMeasurementStdDevs(std::initializer_list<double> measStdDevs)` | Sensor noise standard deviations (Kalman R) |
| `WithMaxVoltage(units::volt_t maxVoltage)` | Saturate output at this voltage |
| `WithPeriod(units::second_t period)` | Control loop period (default: 20 ms) |

---

## Getters

```cpp
LQRType GetType() const;
units::second_t GetPeriod() const;
units::volt_t GetMaxVoltage() const;
std::variant<frc::LinearSystem<1, 1, 1>, frc::LinearSystem<2, 1, 1>> GetSystem() const;
std::variant<Loop1, Loop2> GetLoop() const;
```

{% hint style="info" %}
`GetSystem()` and `GetLoop()` return `std::variant`. Access the correct alternative using `std::get`:

```cpp
auto loop1 = std::get<LQRConfig::Loop1>(cfg.GetLoop());  // FLYWHEEL
auto loop2 = std::get<LQRConfig::Loop2>(cfg.GetLoop());  // ARM or ELEVATOR
```
{% endhint %}

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
   .WithMaxVoltage(12_V)
   .WithPeriod(20_ms);
```
