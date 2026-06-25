# Building an Arm

**Goal:** Create an `Arm` subsystem with position control and physics simulation.

---

{% stepper %}
{% step %}

#### Create and configure the `SmartMotorController`

Follow [Configuring a Motor Controller](configuring-a-motor.md) to build the `SmartMotorController` for your arm joint. Use `ArmFeedforward` in the config and set mechanism limits that match your physical range of motion.

{% endstep %}

{% step %}

#### Create an `ArmConfig`

`ArmConfig` describes the physical arm and its telemetry. Arm length is required for simulation.

```java
ArmConfig armConfig = new ArmConfig()
    .withLength(Meters.of(0.6))
    .withTelemetry("Shoulder", TelemetryVerbosity.HIGH);
```

{% endstep %}

{% step %}

#### Construct the `Arm`

```java
Arm arm = new Arm(armConfig, motor);
```

{% endstep %}

{% step %}

#### Integrate into a `SubsystemBase`

Expose command factories and triggers from the subsystem. `setAngle` moves to a fixed position; `runTo` moves and holds until a tolerance condition is met.

```java
public class ShoulderSubsystem extends SubsystemBase {

    private final SmartMotorController motor;
    private final Arm arm;

    public ShoulderSubsystem() {
        SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig()
            .withMotorInverted(false)
            .withIdleMode(MotorMode.BRAKE)
            .withGearing(new MechanismGearing(GearBox.fromTeeth(14, 72)))
            .withClosedLoopController(0.5, 0.0, 0.01)
            .withFeedforward(new ArmFeedforward(0.1, 0.5, 0.01))
            .withSoftLimits(Degrees.of(-90), Degrees.of(90))
            .withStatorCurrentLimit(Amps.of(40))
            .withTelemetry("ShoulderMotor", TelemetryVerbosity.HIGH);

        motor = new SparkWrapper(
            new SparkMax(1, MotorType.kBrushless),
            DCMotor.getNEO(1),
            motorConfig
        );

        ArmConfig armConfig = new ArmConfig()
            .withLength(Meters.of(0.6))
            .withTelemetry("Shoulder", TelemetryVerbosity.HIGH);

        arm = new Arm(armConfig, motor);
    }

    /** Move to a fixed angle. */
    public Command setAngleCommand(Angle angle) {
        return arm.setAngle(angle);
    }

    /** Move to angle, finish when within tolerance. */
    public Command runToAngle(Angle angle, Angle tolerance) {
        return arm.runTo(angle, tolerance);
    }

    public Angle getAngle() {
        return arm.getAngle();
    }

    @Override
    public void periodic() {
        arm.updateTelemetry();
    }

    @Override
    public void simulationPeriodic() {
        arm.simIterate();
    }
}
```

{% endstep %}

{% step %}

#### Wire up triggers

`Arm` exposes `isNear(angle, tolerance)`, `max()`, and `min()` as `Trigger` factories. Bind them in your `RobotContainer` or use them to gate commands.

```java
// In RobotContainer:
Trigger atStow  = shoulder.arm.isNear(Degrees.of(-90), Degrees.of(2));
Trigger atScore = shoulder.arm.isNear(Degrees.of(45),  Degrees.of(2));
```

{% endstep %}
{% endstepper %}

---

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_arm/java/frc/robot/subsystems/ArmSubsystem.java#L82-L91" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/exponential_arm/java/frc/robot/subsystems/ExponentiallyProfiledArmSubsystem.java#L157-L171" %}

---

## Related pages

- [Arm](../api/java/mechanisms/arm.md)
- [ArmConfig](../api/java/config/arm-config.md)
- [Configuring a Motor Controller](configuring-a-motor.md)
- [Simulation](simulation.md)
