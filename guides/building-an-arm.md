# Building an Arm Subsystem

**Goal:** Create an `Arm` subsystem with position control and physics simulation.

***

{% stepper %}
{% step %}
#### Create and configure the `SmartMotorController`

Follow [Configuring a Motor Controller](configuring-a-motor.md) to build the `SmartMotorController` for your arm joint. Use `ArmFeedforward` in the config and set soft limits that match your physical range of motion. The arm's moment of inertia (for simulation) and starting position are also set on `SmartMotorControllerConfig`, via `withMomentOfInertia(length, mass)` and `withStartingPosition(angle)`.
{% endstep %}

{% step %}
#### Create an `ArmConfig`

`ArmConfig` (`yams.core.mechanisms.config.ArmConfig`) describes the physical arm and its telemetry. Arm length is required for simulation.

```java
ArmConfig armConfig = new ArmConfig()
    .withLength(Meters.of(0.6))
    .withHardLimits(Degrees.of(-100), Degrees.of(100))
    .withTelemetry("Shoulder", TelemetryVerbosity.HIGH);
```
{% endstep %}

{% step %}
#### Construct the `Arm`

Import `Arm` from `yams.commands2.mechanisms` (Commands v2) or `yams.commands3.mechanisms` (Commands v3). The constructor takes the config and the motor controller.

```java
Arm arm = new Arm(armConfig, motor);
```
{% endstep %}

{% step %}
#### Integrate into a subsystem

Expose command factories and triggers from the subsystem. `setAngle` moves to a fixed position and holds it; `runTo` moves and ends once the arm is within tolerance.

{% tabs %}
{% tab title="Commands v2" %}
```java
package first.robot.subsystems;

import static org.wpilib.units.Units.Amps;
import static org.wpilib.units.Units.Degrees;
import static org.wpilib.units.Units.Kilograms;
import static org.wpilib.units.Units.Meters;

import com.revrobotics.spark.SparkLowLevel.MotorType;
import com.revrobotics.spark.SparkMax;
import com.revrobotics.util.CANPorts;
import org.wpilib.command2.Command;
import org.wpilib.command2.SubsystemBase;
import org.wpilib.command2.button.Trigger;
import org.wpilib.math.controller.ArmFeedforward;
import org.wpilib.math.system.DCMotor;
import org.wpilib.units.measure.Angle;
import yams.commands2.config.SmartMotorControllerConfig;
import yams.commands2.mechanisms.Arm;
import yams.core.gearing.GearBox;
import yams.core.gearing.MechanismGearing;
import yams.core.mechanisms.config.ArmConfig;
import yams.core.motorcontrollers.SmartMotorController;
import yams.core.motorcontrollers.enums.MotorMode;
import yams.core.motorcontrollers.local.SparkWrapper;
import yams.core.telemetry.enums.TelemetryVerbosity;

public class ShoulderSubsystem extends SubsystemBase {

    private final SmartMotorController motor;
    private final Arm arm;

    public ShoulderSubsystem() {
        SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig(this)
            .withMotorInverted(false)
            .withZeroPower(MotorMode.BRAKE)
            .withGearing(new MechanismGearing(GearBox.fromTeeth(14, 72)))
            .withClosedLoopController(0.5, 0.0, 0.01)
            .withFeedforward(new ArmFeedforward(0.1, 0.5, 0.01))
            .withSoftLimits(Degrees.of(-90), Degrees.of(90))
            .withMomentOfInertia(Meters.of(0.6), Kilograms.of(2.0))
            .withStartingPosition(Degrees.of(0))
            .withStatorCurrentLimit(Amps.of(40))
            .withTelemetry("ShoulderMotor", TelemetryVerbosity.HIGH);

        motor = new SparkWrapper(
            new SparkMax(CANPorts.fromBusId(1), 1, MotorType.kBrushless),
            DCMotor.getNEO(1),
            motorConfig
        );

        ArmConfig armConfig = new ArmConfig()
            .withLength(Meters.of(0.6))
            .withHardLimits(Degrees.of(-100), Degrees.of(100))
            .withTelemetry("Shoulder", TelemetryVerbosity.HIGH);

        arm = new Arm(armConfig, motor);
    }

    /** Move to a fixed angle and hold it. */
    public Command setAngleCommand(Angle angle) {
        return arm.setAngle(angle);
    }

    /** Move to angle, finish when within tolerance. */
    public Command runToAngle(Angle angle, Angle tolerance) {
        return arm.runTo(angle, tolerance);
    }

    /** Trigger that is true while the arm is within tolerance of an angle. */
    public Trigger near(Angle angle, Angle tolerance) {
        return arm.near(angle, tolerance);
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
{% endtab %}

{% tab title="Commands v3" %}
The configuration is identical. The class implements `Mechanism` instead of extending `SubsystemBase`, and the YAMS imports come from `yams.commands3`.

```java
package first.robot.mechanisms;

import static org.wpilib.units.Units.Degrees;

import org.wpilib.command3.Command;
import org.wpilib.command3.Mechanism;
import org.wpilib.command3.Trigger;
import org.wpilib.units.measure.Angle;
import yams.commands3.config.SmartMotorControllerConfig;
import yams.commands3.mechanisms.Arm;
// ... the yams.core, org.wpilib.math, and vendor imports are the same as Commands v2

public class ShoulderMechanism implements Mechanism {

    public static final Angle STOW_ANGLE = Degrees.of(0);
    public static final Angle TOLERANCE  = Degrees.of(2);

    private final SmartMotorController motor;
    private final Arm arm;

    public ShoulderMechanism() {
        SmartMotorControllerConfig motorConfig = new SmartMotorControllerConfig(this)
            // ... same builder calls as Commands v2
            .withTelemetry("ShoulderMotor", TelemetryVerbosity.HIGH);

        motor = new SparkWrapper(
            new SparkMax(CANPorts.fromBusId(1), 1, MotorType.kBrushless),
            DCMotor.getNEO(1),
            motorConfig
        );

        ArmConfig armConfig = new ArmConfig()
            .withLength(Meters.of(0.6))
            .withHardLimits(Degrees.of(-100), Degrees.of(100))
            .withTelemetry("Shoulder", TelemetryVerbosity.HIGH);

        arm = new Arm(armConfig, motor);
    }

    /** Hold an angle at the lowest priority, so any other arm command can take over. */
    public Command hold(Angle angle) {
        return run(coroutine -> coroutine.await(arm.setAngle(angle)))
            .withPriority(Command.LOWEST_PRIORITY)
            .named("Shoulder Hold " + angle.in(Degrees) + " deg");
    }

    /** Default command: hold the stow angle. */
    public Command stow() {
        return hold(STOW_ANGLE);
    }

    /** Move to angle, finish when within tolerance. */
    public Command moveTo(Angle angle) {
        return arm.runTo(angle, TOLERANCE);
    }

    /** Trigger that is true while the arm is within tolerance of an angle. */
    public Trigger near(Angle angle) {
        return arm.near(angle, TOLERANCE);
    }

    public Angle getAngle() {
        return arm.getAngle();
    }

    /** Call from Robot.robotPeriodic(), before the scheduler runs. */
    public void periodic() {
        arm.updateTelemetry();
    }

    /** Call from Robot.robotPeriodic() in simulation. */
    public void simulationPeriodic() {
        arm.simIterate();
    }
}
```

In `Robot`, set the default command with `shoulder.setDefaultCommand(shoulder.stow())` and call `shoulder.periodic()` (plus `shoulder.simulationPeriodic()` when `RobotBase.isSimulation()`) from `robotPeriodic()` before `Scheduler.getDefault().run()`.
{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
#### Wire up triggers

`Arm` exposes `near(angle, tolerance)`, `gte(angle)`, `lte(angle)`, `between(start, end)`, `max()`, and `min()` as `Trigger` factories. Bind them in your `RobotContainer` or use them to gate commands.

```java
// In RobotContainer:
Trigger atStow  = shoulder.near(Degrees.of(-90), Degrees.of(2));
Trigger atScore = shoulder.near(Degrees.of(45),  Degrees.of(2));
```

{% hint style="info" %}
The `Trigger` factory was renamed from `isNear(...)` to `near(...)`. `isNear(angle, tolerance)` still exists, but it now returns a plain `boolean` for use outside a `Command`/`Trigger` context.
{% endhint %}
{% endstep %}
{% endstepper %}

***

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/commands2/simple_robot-2027/src/main/java/first/robot/subsystems/ArmSubsystem.java#L80-L85" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/commands2/simple_robot-2027/src/main/java/first/robot/subsystems/ExponentiallyProfiledArmSubsystem.java" %}

Commands v3 version of the arm:

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/commands3/simple_robot-2027/src/main/java/first/robot/mechanisms/ArmMechanism.java" %}

***

## Related pages

* [Arm](../java-reference/mechanisms/arm.md)
* [ArmConfig](../java-reference/config/arm-config.md)
* [Configuring a Motor Controller](configuring-a-motor.md)
* [Simulation](simulation.md)
