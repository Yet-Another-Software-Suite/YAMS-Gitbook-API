# Configuring a Motor Controller

**Goal:** Build and instantiate a `SmartMotorController` ready for use in a mechanism or standalone.

***

{% stepper %}
{% step %}

#### Choose your hardware and import vendor libraries

Identify which motor controller you are using:

* **REV SPARK MAX / SPARK FLEX** — requires the REVLib vendordep.
* **CTRE TalonFX / TalonFXS** — requires the Phoenix 6 vendordep.

Add the appropriate vendordep to your project before proceeding.

{% endstep %}

{% step %}

#### Create a `SmartMotorControllerConfig` and chain builder methods

`SmartMotorControllerConfig` follows the builder pattern. Call the methods you need and chain them together. At minimum, set gearing, a closed-loop controller (PID), and idle mode.

**Java — SPARK MAX with arm control:**

```java
SmartMotorControllerConfig config = new SmartMotorControllerConfig()
    .withMotorInverted(false)
    .withIdleMode(MotorMode.Brake)
    .withGearing(new MechanismGearing(GearBox.fromTeeth(14, 72)))
    .withClosedLoopController(0.5, 0.0, 0.01)
    .withFeedforward(new ArmFeedforward(0.1, 0.5, 0.01))
    .withMechanismUpperLimit(Degrees.of(90))
    .withMechanismLowerLimit(Degrees.of(-90))
    .withStatorCurrentLimit(Amps.of(40))
    .withTelemetry("ShoulderMotor", TelemetryVerbosity.HIGH);
```

**C++ — TalonFX with arm control:**

```cpp
motorcontrollers::SmartMotorControllerConfig config;
config.WithMotorInverted(false)
      .WithIdleMode(MotorMode::BRAKE)
      .WithMotorGearing(gearing::MechanismGearing{
          gearing::GearBox::FromTeeth({14, 72})})
      .WithFeedback(0.5, 0.0, 0.01)
      .WithArmFeedforward(0.1, 0.5, 0.01, 0.0)
      .WithMechanismLimits(-90_deg, 90_deg)
      .WithStatorCurrentLimit(40_A)
      .WithTelemetry("ShoulderMotor", TelemetryVerbosity::HIGH);
```

{% endstep %}

{% step %}

#### Construct the concrete motor controller

Pass the vendor hardware object, the WPILib `DCMotor` model, and the config to the appropriate constructor.

**Java (REV SPARK):**

```java
SmartMotorController motor = new SparkWrapper(
    new SparkMax(1, MotorType.kBrushless),
    DCMotor.getNEO(1),
    config
);
```

**Java (CTRE TalonFX):**

```java
SmartMotorController motor = new TalonFXWrapper(
    new TalonFX(1),
    DCMotor.getKrakenX60(1),
    config
);
```

**C++ (REV SPARK):**

```cpp
SparkWrapper motor{&spark, frc::DCMotor::NEO(1), &config};
```

**C++ (CTRE TalonFX):**

```cpp
TalonFXWrapper motor{&talon, frc::DCMotor::KrakenX60(1), &config};
TalonFXSWrapper motor{&m_talonFXS, DCMotor::NEO(1), MotorArrangement::NEO, &cfg};
```

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
If you are using a SPARK MAX with an **absolute encoder**, you should call `.withExternalEncoderDiscontinuityPoint(angle)` on the config to set the wrap point. Without it, the encoder will report a discontinuity at 0°/360° and the closed-loop controller will behave erratically near that boundary.
{% endhint %}

***

## Related pages

* [SmartMotorControllerConfig](../api/java/motor-controllers/smart-motor-controller-config.md)
* [MechanismGearing](../api/java/gearing/mechanism-gearing.md)
