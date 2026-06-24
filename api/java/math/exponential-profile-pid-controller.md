# ExponentialProfilePIDController

**Package:** `yams.math`

`ExponentialProfilePIDController` wraps a `PIDController` with an `ExponentialProfile` to produce smooth, physically-motivated setpoints. The exponential profile models a first-order system response — unlike the trapezoidal profile, it does not have a constant-velocity segment, so it tends to feel more natural for arm and flywheel mechanisms.

## Constructors

```java
ExponentialProfilePIDController(PIDController controller, ExponentialProfile.Constraints constraints)
ExponentialProfilePIDController(double kP, double kI, double kD, ExponentialProfile.Constraints constraints)
```

## Constraints Factory Methods

These static methods produce `ExponentialProfile.Constraints` suited to specific mechanism types:

| Method | Description |
| ------ | ----------- |
| `static Constraints createElevatorConstraints(Voltage maxVolts, DCMotor motor, Mass mass, Distance drumRadius, MechanismGearing gearing)` | Constraints for an elevator (linear system). |
| `static Constraints createArmConstraints(Voltage maxVolts, DCMotor motor, MomentOfInertia moi, MechanismGearing gearing)` | Constraints for an arm using MOI directly. |
| `static Constraints createArmConstraints(Voltage maxVolts, DCMotor motor, Mass mass, Distance length, MechanismGearing gearing)` | Constraints for an arm — computes MOI from mass and length. |
| `static Constraints createFlywheelConstraints(Voltage maxVolts, DCMotor motor, MomentOfInertia moi, MechanismGearing gearing)` | Constraints for a flywheel using MOI. |
| `static Constraints createFlywheelConstraints(Voltage maxVolts, DCMotor motor, Mass mass, Distance radius, MechanismGearing gearing)` | Constraints for a flywheel — computes MOI from mass and radius. |
| `static Constraints createConstraints(Voltage maxVolts, AngularVelocity maxVelocity, AngularAcceleration maxAcceleration)` | Generic constraints from physical limits. |

## Control Methods

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `calculate(double measurementPosition, double setpointPosition)` | `double` | Advances the profile by one timestep and returns PID output. Uses FPGA clock for dt. |
| `calculate(double measurementPosition, double setpointVelocity, double setpointPosition)` | `double` | Same, but also provides an explicit velocity setpoint for feedforward. |
| `reset(State measurement)` | `void` | Resets the profile and PID to the given state. Call on enable. |
| `reset(double position, double velocity)` | `void` | Resets with position and velocity scalars. |

## State Query Methods

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `atSetpoint()` | `boolean` | True when the measurement is within the configured tolerance of the setpoint. |
| `getSetpoint()` | `double` | The current profile position target. |
| `getCurrentState()` | `State` | The current profile state (position + velocity). |
| `getCurrentAngle()` | `Angle` | Current profile position as an `Angle`. |
| `getNextAngle()` | `Angle` | Next profile position (one timestep ahead). |
| `getCurrentVelocitySetpoint()` | `AngularVelocity` | Current velocity setpoint from the profile. |
| `getNextVelocitySetpoint()` | `AngularVelocity` | Next velocity setpoint. |
| `getNextState()` | `Optional<State>` | The next profile state, or empty if not yet calculated. |
| `getConstraints()` | `Optional<Constraints>` | The configured constraints, or empty. |

## PID Tuning Methods

| Method | Description |
| ------ | ----------- |
| `setTolerance(double tolerance)` | Sets the position tolerance for `atSetpoint()`. |
| `getPositionTolerance()` | Returns the current tolerance. |
| `getP()`, `setP(double kP)` | Proportional gain. |
| `getI()`, `setI(double kI)` | Integral gain. |
| `getD()`, `setD(double kD)` | Derivative gain. |
| `setConstraints(Constraints constraints)` | Updates the profile constraints at runtime. |
| `enableContinuousInput(double minimumInput, double maximumInput)` | Enables continuous input (for wrapping mechanisms). |

## Example

```java
ExponentialProfile.Constraints constraints =
    ExponentialProfilePIDController.createArmConstraints(
        Volts.of(12),
        DCMotor.getNEO(1),
        Kilograms.of(2), Meters.of(0.5),
        new MechanismGearing(GearBox.fromTeeth(14, 72))
    );

ExponentialProfilePIDController controller =
    new ExponentialProfilePIDController(0.5, 0.0, 0.0, constraints);
controller.setTolerance(0.05); // radians
```

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/exponential_arm/java/frc/robot/subsystems/ExponentiallyProfiledArmSubsystem.java#L157-L171" %}

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/exponential_elevator/java/frc/robot/subsystems/ExponentiallyProfiledElevatorSubsystem.java#L134-L146" %}
