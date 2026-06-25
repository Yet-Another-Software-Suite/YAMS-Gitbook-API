# Building a FlyWheel

**Goal:** Create a `FlyWheel` subsystem with velocity control and physics simulation.

---

## Steps

### 1. Configure the motor for velocity control

Use `withClosedLoopController` for the closed-loop PID and `withFeedforward(SimpleMotorFeedforward)` to reduce steady-state error. Optionally add an exponential profile for smooth spin-up.

```java
SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig()
    .withMotorInverted(false)
    .withIdleMode(MotorMode.COAST)
    .withGearing(new MechanismGearing(GearBox.fromRatio(1.5)))
    .withClosedLoopController(0.0003, 0.0, 0.0)
    .withFeedforward(new SimpleMotorFeedforward(0.1, 0.002, 0.0))
    .withMomentOfInertia(Meters.of(0.0508), Kilograms.of(0.18))
    .withStatorCurrentLimit(Amps.of(80))
    .withTelemetry("ShooterMotor", TelemetryVerbosity.HIGH);

SmartMotorController motor = new TalonFXWrapper(
    new TalonFX(3),
    DCMotor.getKrakenX60(1),
    motorConfig
);
```

### 2. Create a `FlyWheelConfig`

Wheel diameter is used for linear velocity conversion. Moment of inertia (for simulation) is set on `SmartMotorControllerConfig` via `withMomentOfInertia`.

```java
FlyWheelConfig flywheelConfig = new FlyWheelConfig()
    .withDiameter(Meters.of(0.1016))
    .withTelemetry("Shooter", TelemetryVerbosity.HIGH);
```

### 3. Construct the `FlyWheel`

```java
FlyWheel flyWheel = new FlyWheel(flywheelConfig, motor);
```

### 4. Integrate into a `SubsystemBase`

```java
public class ShooterSubsystem extends SubsystemBase {

    private final SmartMotorController motor;
    private final FlyWheel flyWheel;

    private static final AngularVelocity SHOOT_SPEED     = RPM.of(4500);
    private static final AngularVelocity SPEED_TOLERANCE = RPM.of(100);

    public ShooterSubsystem() {
        SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig()
            .withMotorInverted(false)
            .withIdleMode(MotorMode.COAST)
            .withGearing(new MechanismGearing(GearBox.fromRatio(1.5)))
            .withClosedLoopController(0.0003, 0.0, 0.0)
            .withFeedforward(new SimpleMotorFeedforward(0.1, 0.002, 0.0))
            .withMomentOfInertia(Meters.of(0.0508), Kilograms.of(0.18))
            .withStatorCurrentLimit(Amps.of(80))
            .withTelemetry("ShooterMotor", TelemetryVerbosity.HIGH);

        motor = new TalonFXWrapper(
            new TalonFX(3),
            DCMotor.getKrakenX60(1),
            motorConfig
        );

        FlyWheelConfig flywheelConfig = new FlyWheelConfig()
            .withDiameter(Meters.of(0.1016))
            .withTelemetry("Shooter", TelemetryVerbosity.HIGH);

        flyWheel = new FlyWheel(flywheelConfig, motor);
    }

    /** Spin to a target velocity and hold indefinitely. */
    public Command spinUpCommand(AngularVelocity velocity) {
        return flyWheel.run(velocity);
    }

    /** Spin up and finish once within tolerance — use to gate shooting. */
    public Command spinUpAndWait(AngularVelocity velocity) {
        return flyWheel.runTo(velocity, SPEED_TOLERANCE);
    }

    /** Trigger that is true whenever the wheel is near the current target. */
    public Trigger readyToShoot() {
        return flyWheel.isNear(SHOOT_SPEED, SPEED_TOLERANCE);
    }

    @Override
    public void periodic() {
        flyWheel.updateTelemetry();
    }

    @Override
    public void simulationPeriodic() {
        flyWheel.simIterate();
    }
}
```

### 5. Sequence shooter commands

Use `runTo` to block until the wheel is at speed before feeding a game piece:

```java
Command shoot = shooter.spinUpAndWait(SHOOT_SPEED)
    .andThen(indexer.feedCommand());
```

Or bind the `readyToShoot()` trigger to allow feeding at any time once the wheel stabilizes:

```java
shooter.readyToShoot().whileTrue(indexer.feedCommand());
```

---

## Notes

{% hint style="info" %}
`runTo(velocity, tolerance)` ends once the wheel reaches the target. Use it in command sequences where the next step should not begin until the wheel is up to speed. `isNear(velocity, tolerance)` (exposed as a `Trigger`) stays true as long as the wheel remains within tolerance — use it for continuously gated logic like a conveyor that feeds whenever the shooter is ready.
{% endhint %}

---

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/simple_shooter/java/frc/robot/subsystems/ShooterSubsystem.java" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/hooded_shooter/java/frc/robot/subsystems/FlywheelSubsystem.java" %}

---

## Related pages

- [FlyWheel](../api/java/mechanisms/flywheel.md)
- [FlyWheelConfig](../api/java/config/flywheel-config.md)
- [Configuring a Motor Controller](configuring-a-motor.md)
- [Simulation](simulation.md)
