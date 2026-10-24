# Configuring a motor controller

**Goal:** Build and instantiate a `SmartMotorController` ready for use in a mechanism or standalone.

***

{% stepper %}
{% step %}
#### Choose your hardware and import vendor libraries

Identify which motor controller you are using:

* **REV SPARK MAX / SPARK FLEX**: requires the REVLib 2027 vendordep. Wrapped by `SparkWrapper`.
* **CTRE TalonFX / TalonFXS**: requires the Phoenix 6 vendordep. Wrapped by `TalonFXWrapper` / `TalonFXSWrapper`.
* **Thrifty Nova**: requires the ThriftyLib 2027 vendordep. Wrapped by `NovaWrapper`.

Add the appropriate vendordep to your project before proceeding.
{% endstep %}

{% step %}
#### Create a `SmartMotorControllerConfig` and chain builder methods

`SmartMotorControllerConfig` follows the builder pattern. Call the methods you need and chain them together. At minimum, set gearing, a closed loop controller (PID), and zero power mode.

In Java, import `SmartMotorControllerConfig` from the command framework package you installed and pass the owning `Subsystem` (Commands v2) or `Mechanism` (Commands v3) to the constructor. Everything else comes from `yams.core`.

{% tabs %}
{% tab title="Commands v2" %}
```java
import yams.commands2.config.SmartMotorControllerConfig;
import yams.core.gearing.GearBox;
import yams.core.gearing.MechanismGearing;
import yams.core.motorcontrollers.enums.MotorMode;
import yams.core.telemetry.enums.TelemetryVerbosity;
```
{% endtab %}

{% tab title="Commands v3" %}
```java
import yams.commands3.config.SmartMotorControllerConfig;
import yams.core.gearing.GearBox;
import yams.core.gearing.MechanismGearing;
import yams.core.motorcontrollers.enums.MotorMode;
import yams.core.telemetry.enums.TelemetryVerbosity;
```
{% endtab %}
{% endtabs %}

**Java: SPARK MAX with arm control**

```java
// "this" is the Subsystem (Commands v2) or Mechanism (Commands v3) that owns the motor.
SmartMotorControllerConfig config = new SmartMotorControllerConfig(this)
    .withMotorInverted(false)
    .withZeroPower(MotorMode.BRAKE)
    .withGearing(new MechanismGearing(GearBox.fromTeeth(14, 72)))
    .withClosedLoopController(0.5, 0.0, 0.01)
    .withFeedforward(new ArmFeedforward(0.1, 0.5, 0.01))
    .withSoftLimits(Degrees.of(-90), Degrees.of(90))
    .withStatorCurrentLimit(Amps.of(40))
    .withTelemetry("ShoulderMotor", TelemetryVerbosity.HIGH);
```

**C++: TalonFX with arm control**

```cpp
#include "yams/gearing/GearBox.hpp"
#include "yams/gearing/MechanismGearing.hpp"
#include "yams/motorcontrollers/SmartMotorControllerConfig.hpp"

using namespace yams;
using Cfg = motorcontrollers::SmartMotorControllerConfig;

Cfg config;
config.WithSubsystem(this)
      .WithMotorInverted(false)
      .WithZeroPower(Cfg::MotorMode::BRAKE)
      .WithMotorGearing(gearing::MechanismGearing{
          gearing::GearBox::FromTeeth({14, 72})})
      .WithFeedback(0.5, 0.0, 0.01)
      .WithFeedforward(wpi::math::ArmFeedforward{0.1_V, 0.0_V,
                                                 wpi::units::unit_t<wpi::math::ArmFeedforward::kv_unit>{0.5},
                                                 wpi::units::unit_t<wpi::math::ArmFeedforward::ka_unit>{0.01}})
      .WithMechanismLimits(-90_deg, 90_deg)
      .WithStatorCurrentLimit(40_A)
      .WithTelemetry("ShoulderMotor", Cfg::TelemetryVerbosity::HIGH);
```
{% endstep %}

{% step %}
#### Construct the concrete motor controller

Pass the vendor hardware object, the WPILib `DCMotor` model, and the config to the appropriate constructor. The Java wrappers live in `yams.core.motorcontrollers.local` (`SparkWrapper`, `NovaWrapper`) and `yams.core.motorcontrollers.remote` (`TalonFXWrapper`, `TalonFXSWrapper`).

**Java (REV SPARK):**

```java
SmartMotorController motor = new SparkWrapper(
    new SparkMax(CANPorts.fromBusId(1), 1, MotorType.kBrushless),
    DCMotor.getNEO(1),
    config
);
```

**Java (CTRE TalonFX):**

```java
SmartMotorController motor = new TalonFXWrapper(
    new TalonFX(1, new CANBus(CANPort.CAN_S0)),
    DCMotor.getKrakenX60(1),
    config
);
```

**Java (Thrifty Nova):**

```java
// Nova driving a NEO on CAN bus 0, CAN ID 3
SmartMotorController motor = new NovaWrapper(
    new Nova(0, 3, MotorType.NEO),
    DCMotor.getNEO(1),
    config
);
```

**C++ (REV SPARK):**

```cpp
#include "yams/motorcontrollers/local/SparkWrapper.hpp"

rev::spark::SparkMax spark{wpi::CANPort::CAN_S0, 1, rev::spark::SparkMax::MotorType::kBrushless};
yams::motorcontrollers::local::SparkWrapper motor{&spark, wpi::math::DCMotor::NEO(1), &config};
```

**C++ (CTRE TalonFX / TalonFXS):**

```cpp
#include "yams/motorcontrollers/remote/TalonFXWrapper.hpp"
#include "yams/motorcontrollers/remote/TalonFXSWrapper.hpp"

using namespace yams::motorcontrollers::remote;

TalonFXWrapper motor{&talon, wpi::math::DCMotor::KrakenX60(1), &config};
TalonFXSWrapper motor{&talonFXS, wpi::math::DCMotor::NEO(1), TalonFXSWrapper::MotorArrangement::NEO, &config};
```

**C++ (Thrifty Nova):**

```cpp
#include "yams/motorcontrollers/local/NovaWrapper.hpp"

thrifty::Nova nova{0, 3, thrifty::Motor::NEO};
yams::motorcontrollers::local::NovaWrapper motor{&nova, wpi::math::DCMotor::NEO(1), &config};
```

In C++ the hardware object and the config are passed by pointer and must outlive the wrapper, so declare them as subsystem members.
{% endstep %}

{% step %}
#### Pass the result to a mechanism or use it directly

The returned `SmartMotorController` is ready to hand to `Arm`, `Elevator`, `FlyWheel`, or `SwerveDrive`. See the relevant how-to guide for the next steps.

```java
Arm arm = new Arm(armConfig, motor);
```
{% endstep %}
{% endstepper %}

***

## Notes

{% hint style="info" %}
If you are using a SPARK MAX with an **absolute encoder**, you should call `.withExternalEncoderDiscontinuityPoint(angle)` on the config to set the wrap point (`Rotations.of(0.5)` for a [-0.5, 0.5] rotation range, `Rotations.of(1)` for [0, 1]). Without it, the encoder will report a discontinuity at 0/360 degrees and the closed loop controller will behave erratically near that boundary.
{% endhint %}

{% hint style="info" %}
**Live tuning:** with `TelemetryVerbosity.HIGH`, the config automatically registers a "Live Tuning" command for the owning subsystem or mechanism. If you set up the motor controller's telemetry yourself with `setupTelemetry(NetworkTable, NetworkTable)`, call `config.setupLiveTuning()` afterwards (Java). Tunable fields are in degrees and RPM (meters and meters per second for linear mechanisms), while read-only fields are in rotations and rotations per second.
{% endhint %}

{% hint style="info" %}
**Encoder getters:** `getMechanismPosition()` / `getMechanismVelocity()` report the feedback the closed loop uses. Use `getRelativeMechanismPosition()` / `getRelativeMechanismVelocity()` to read only the motor's relative encoder, and `getExternalEncoderMechanismPosition()` / `getExternalEncoderMechanismVelocity()` to read only the external encoder. The C++ equivalents use the same names in `PascalCase`.
{% endhint %}

***

## Related pages

* [SmartMotorControllerConfig](../java-reference/motor-controllers/smart-motor-controller-config.md)
* [MechanismGearing](../java-reference/gearing/mechanism-gearing.md)
