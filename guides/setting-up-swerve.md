# Setting Up Swerve Drive

**Goal:** Create a fully functional `SwerveDrive` subsystem with field-relative driving, odometry, and simulation.

***

{% stepper %}
{% step %}
#### Configure drive and azimuth motor controllers for each module

Each swerve module needs two `SmartMotorController` instances: one for drive (velocity) and one for azimuth (position). Follow [Configuring a Motor Controller](configuring-a-motor.md) for each.

Drive controllers use velocity PID and a `SimpleMotorFeedforward`. Azimuth controllers use position PID with continuous wrapping, and optionally a `SimpleMotorFeedforward` for friction compensation.
{% endstep %}

{% step %}
#### Create a `SwerveModuleConfig` for each module

Pass the drive and azimuth motor controllers into the `SwerveModuleConfig` constructor. Then set the wheel radius, the module's location relative to the robot center, and the absolute encoder.

```java
SwerveModuleConfig frontLeftConfig = new SwerveModuleConfig(driveMotorFL, azimuthMotorFL)
    .withWheelRadius(Meters.of(0.0508))
    .withLocation(new Translation2d(0.381, 0.381))
    .withAbsoluteEncoder(cancoderFL.getAbsolutePosition().asSupplier())
    .withAbsoluteEncoderOffset(Rotations.of(0.24))
    .withTelemetry("FrontLeft", TelemetryVerbosity.HIGH);
```

Repeat for `FrontRight`, `BackLeft`, and `BackRight`, adjusting the `Translation2d` signs and encoder offsets.
{% endstep %}

{% step %}
#### Build `SwerveModule` instances and create a `SwerveDriveConfig`

Wrap each `SwerveModuleConfig` in a `SwerveModule`, then collect the four modules into a `SwerveDriveConfig`. Pass the subsystem (Commands v2) or mechanism (Commands v3) and all four modules in the constructor, then set kinematic limits and the gyro supplier.

`SwerveModuleConfig` and `SwerveModule` come from `yams.core`. `SwerveDriveConfig`, `SwerveDrive`, and `SwerveInputStream` come from `yams.commands2` or `yams.commands3`.

```java
SwerveModule frontLeft  = new SwerveModule(frontLeftConfig);
SwerveModule frontRight = new SwerveModule(frontRightConfig);
SwerveModule backLeft   = new SwerveModule(backLeftConfig);
SwerveModule backRight  = new SwerveModule(backRightConfig);

SwerveDriveConfig driveConfig = new SwerveDriveConfig(this, frontLeft, frontRight, backLeft, backRight)
    .withGyro(gyro::getRotation3d)
    .withMaximumChassisSpeed(MetersPerSecond.of(4.5), DegreesPerSecond.of(360))
    .withTranslationController(new PIDController(1.0, 0, 0))
    .withRotationController(new PIDController(1.0, 0, 0))
    .withTelemetry("swerve", TelemetryVerbosity.HIGH);
```

`withGyro` takes a `Supplier<Rotation3d>` for the robot's attitude, such as `pigeon::getRotation3d`. Its yaw is the robot's heading, and its roll and pitch are what anti-tipping (`withAntiTipping`) corrects with.

{% hint style="info" %}
**Gyro mounted on its side?** Call `.withGyroHeadingAxis(GyroAxis.ROLL)` or `.withGyroHeadingAxis(GyroAxis.PITCH)` (`yams.core.mechanisms.config.enums.GyroAxis`) to tell YAMS which gyro axis points up through the robot. The default is `GyroAxis.YAW`, for a gyro mounted flat.
{% endhint %}
{% endstep %}

{% step %}
#### Construct `SwerveDrive` and wire command factories

{% tabs %}
{% tab title="Commands v2" %}
```java
package first.robot.subsystems;

import static org.wpilib.units.Units.Amps;
import static org.wpilib.units.Units.DegreesPerSecond;
import static org.wpilib.units.Units.Meters;
import static org.wpilib.units.Units.MetersPerSecond;
import static org.wpilib.units.Units.Radians;
import static org.wpilib.units.Units.Rotations;

import com.ctre.phoenix6.CANBus;
import com.ctre.phoenix6.hardware.CANcoder;
import com.ctre.phoenix6.hardware.Pigeon2;
import com.ctre.phoenix6.hardware.TalonFX;
import org.wpilib.command2.Command;
import org.wpilib.command2.SubsystemBase;
import org.wpilib.hardware.bus.CANPort;
import org.wpilib.math.controller.PIDController;
import org.wpilib.math.controller.SimpleMotorFeedforward;
import org.wpilib.math.geometry.Pose2d;
import org.wpilib.math.geometry.Rotation3d;
import org.wpilib.math.geometry.Translation2d;
import org.wpilib.math.system.DCMotor;
import yams.commands2.config.SmartMotorControllerConfig;
import yams.commands2.config.SwerveDriveConfig;
import yams.commands2.swerve.SwerveDrive;
import yams.commands2.swerve.SwerveInputStream;
import yams.core.gearing.GearBox;
import yams.core.gearing.MechanismGearing;
import yams.core.mechanisms.config.SwerveModuleConfig;
import yams.core.mechanisms.swerve.SwerveModule;
import yams.core.motorcontrollers.SmartMotorController;
import yams.core.motorcontrollers.enums.MotorMode;
import yams.core.motorcontrollers.remote.TalonFXWrapper;
import yams.core.telemetry.enums.TelemetryVerbosity;

public class DriveSubsystem extends SubsystemBase {

    private final CANBus canBus = new CANBus(CANPort.CAN_S0);
    private final Pigeon2 gyro = new Pigeon2(14, canBus);
    private final SwerveDrive swerveDrive;

    public DriveSubsystem() {

        // --- Front Left ---
        SmartMotorControllerConfig driveConfigFL = new SmartMotorControllerConfig(this)
            .withZeroPower(MotorMode.BRAKE)
            .withGearing(new MechanismGearing(GearBox.fromReductionStages(6.75)))
            .withClosedLoopController(0.1, 0.0, 0.0)
            .withFeedforward(new SimpleMotorFeedforward(0.12, 2.2, 0.3))
            .withStatorCurrentLimit(Amps.of(60));

        SmartMotorControllerConfig azimuthConfigFL = new SmartMotorControllerConfig(this)
            .withZeroPower(MotorMode.BRAKE)
            .withGearing(new MechanismGearing(GearBox.fromReductionStages(12.8)))
            .withClosedLoopController(7.0, 0.0, 0.3)
            .withContinuousWrapping(Radians.of(-Math.PI), Radians.of(Math.PI))
            .withStatorCurrentLimit(Amps.of(40));

        SmartMotorController driveMotorFL = new TalonFXWrapper(
            new TalonFX(10, canBus), DCMotor.getKrakenX60(1), driveConfigFL);
        SmartMotorController azimuthMotorFL = new TalonFXWrapper(
            new TalonFX(11, canBus), DCMotor.getKrakenX60(1), azimuthConfigFL);
        CANcoder cancoderFL = new CANcoder(12, canBus);

        // Repeat for FR, BL, BR with their CAN IDs and encoder offsets...

        SwerveModuleConfig frontLeftModuleConfig = new SwerveModuleConfig(driveMotorFL, azimuthMotorFL)
            .withWheelRadius(Meters.of(0.0508))
            .withLocation(new Translation2d(0.381, 0.381))
            .withAbsoluteEncoder(cancoderFL.getAbsolutePosition().asSupplier())
            .withAbsoluteEncoderOffset(Rotations.of(0.24))
            .withTelemetry("FrontLeft", TelemetryVerbosity.HIGH);

        // SwerveModuleConfig frontRightModuleConfig = ...
        // SwerveModuleConfig backLeftModuleConfig  = ...
        // SwerveModuleConfig backRightModuleConfig = ...

        SwerveModule frontLeft = new SwerveModule(frontLeftModuleConfig);
        // SwerveModule frontRight = new SwerveModule(frontRightModuleConfig);
        // SwerveModule backLeft   = new SwerveModule(backLeftModuleConfig);
        // SwerveModule backRight  = new SwerveModule(backRightModuleConfig);

        SwerveDriveConfig driveConfig = new SwerveDriveConfig(this, frontLeft /*, frontRight, backLeft, backRight */)
            .withGyro(gyro::getRotation3d)
            .withMaximumChassisSpeed(MetersPerSecond.of(4.5), DegreesPerSecond.of(360))
            .withTranslationController(new PIDController(1.0, 0, 0))
            .withRotationController(new PIDController(1.0, 0, 0))
            .withTelemetry("swerve", TelemetryVerbosity.HIGH);

        swerveDrive = new SwerveDrive(driveConfig);
    }

    public SwerveDrive getSwerveDrive() {
        return swerveDrive;
    }

    /** Field-relative drive command driven by a SwerveInputStream. */
    public Command driveFieldRelativeCommand(SwerveInputStream inputStream) {
        return run(() -> swerveDrive.setFieldRelativeChassisSpeeds(inputStream.get()))
            .withName("Field Oriented Drive");
    }

    /** Turn the wheels into an X pattern to resist being pushed. */
    public Command lock() {
        return run(swerveDrive::lockPose).withName("Lock");
    }

    /** Pass vision pose estimates to the odometry estimator. */
    public void addVisionMeasurement(Pose2d pose, double timestampSeconds) {
        swerveDrive.addVisionMeasurement(pose, timestampSeconds);
    }

    public Pose2d getPose() {
        return swerveDrive.getPose();
    }

    /** Robot attitude from the gyro. Its yaw is the robot's heading. */
    public Rotation3d getGyroRotation3d() {
        return swerveDrive.getGyroRotation3d();
    }

    @Override
    public void periodic() {
        swerveDrive.updateTelemetry();
    }

    @Override
    public void simulationPeriodic() {
        swerveDrive.simIterate();
    }
}
```
{% endtab %}

{% tab title="Commands v3" %}
The hardware and config code is identical. The class implements `Mechanism`, the YAMS command-layer imports come from `yams.commands3`, and the command factories are coroutines.

```java
package first.robot.mechanisms;

import org.wpilib.command3.Command;
import org.wpilib.command3.Mechanism;
import yams.commands3.config.SmartMotorControllerConfig;
import yams.commands3.config.SwerveDriveConfig;
import yams.commands3.swerve.SwerveDrive;
import yams.commands3.swerve.SwerveInputStream;
// ... the yams.core, org.wpilib.math, and vendor imports are the same as Commands v2

public class DriveMechanism implements Mechanism {

    private final SwerveDrive swerveDrive;

    public DriveMechanism() {
        // ... same motor, module, and SwerveDriveConfig setup as Commands v2
        swerveDrive = new SwerveDrive(driveConfig);
    }

    public SwerveDrive getSwerveDrive() {
        return swerveDrive;
    }

    /** Field-relative drive command driven by a SwerveInputStream. Runs until canceled. */
    public Command driveFieldRelativeCommand(SwerveInputStream inputStream) {
        return run(coroutine -> {
            while (true) {
                swerveDrive.setFieldRelativeChassisSpeeds(inputStream.get());
                coroutine.yield();
            }
        }).named("Field Oriented Drive");
    }

    /** Turn the wheels into an X pattern to resist being pushed. Runs until canceled. */
    public Command lock() {
        return run(coroutine -> {
            while (true) {
                swerveDrive.lockPose();
                coroutine.yield();
            }
        }).named("Lock");
    }

    /** Call from Robot.robotPeriodic(), before the scheduler runs. */
    public void periodic() {
        swerveDrive.updateTelemetry();
    }

    /** Call from Robot.robotPeriodic() in simulation. */
    public void simulationPeriodic() {
        swerveDrive.simIterate();
    }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
`SwerveDrive` already publishes a `Field2d` with the robot pose. Use `swerveDrive.getField2d()` if you want to draw on it instead of creating your own.
{% endhint %}
{% endstep %}

{% step %}
#### Set up driver input with `SwerveInputStream`

`SwerveInputStream` translates raw joystick axes into field-relative `ChassisVelocities` (WPILib 2027's name for `ChassisSpeeds`), applying deadbands and scaling. Pass the `SwerveDrive` instance as the first argument to `SwerveInputStream.of()`. Rotation is set with `.withControllerRotationAxis()`, or as a fourth argument to `of()`.

```java
SwerveInputStream driverInput = SwerveInputStream.of(
    driveSubsystem.getSwerveDrive(),      // SwerveDrive instance
    () -> -driverController.getLeftY(),   // forward/back
    () -> -driverController.getLeftX()    // strafe
).withControllerRotationAxis(() -> -driverController.getRightX())
 .withDeadband(0.1)
 .withMaximumLinearVelocity(MetersPerSecond.of(4.5))
 .withMaximumAngularVelocity(DegreesPerSecond.of(360))
 .withAllianceRelativeControl()
 .withTelemetry("Driver", TelemetryVerbosity.HIGH);

driveSubsystem.setDefaultCommand(
    driveSubsystem.driveFieldRelativeCommand(driverInput)
);
```

`withTelemetry(name, verbosity)` publishes the stream every time it is read. `LOW` publishes the current drive mode, `MID` adds the stream's configuration, and `HIGH` adds editable copies of the configuration plus a `Live Tuning` command on the dashboard. Call it last, after the rest of the configuration. A `clone()` of a stream has no telemetry.
{% endstep %}

{% step %}
#### Azimuth encoder seeding

`seedAzimuthEncoder()` is called automatically on each `SwerveModule` during construction. No additional call is required at startup. If you need to re-seed after the CAN bus has fully settled, you can call `seedAzimuthEncoder()` on each `SwerveModule` instance individually.
{% endstep %}

{% step %}
#### Auto-align (`driveToPose`) and live PID tuning

`driveToPose(Pose2d)` returns a `Command` that drives the robot to a field-relative pose using the `withTranslationController`/`withRotationController` PID gains set on `SwerveDriveConfig`:

```java
public Command driveToPose(Pose2d target) {
    return swerveDrive.driveToPose(target);
}
```

`SwerveDrive` also publishes a live-tuning command to SmartDashboard at `Mechanisms/<name>/tuning/driveToPose`. Running it checks a tunable `autoalign/enabled` boolean and, while `true`, drives toward a target pose (`autoalign/setpoint/x`/`autoalign/setpoint/y` in meters, `autoalign/setpoint/rot` in degrees) you can edit live in NetworkTables, so you can tune those PID gains without redeploying code. The same command also reads a shared set of module drive/azimuth PID, feedforward, and setpoint fields (`modules/drive/*`, `modules/azimuth/*`, with PID gains under `feedback/` and `kS`/`kV`/`kA` under `feedforward/`) that apply to every module on the drive at once, useful for tuning drive/azimuth motor gains without repeating the same numbers per module. These fields are already live-tunable at `TelemetryVerbosity.HIGH` (the default); see [SwerveDrive: Auto-Align](../java-reference/swerve/swerve-drive.md#auto-align-drive-to-pose).
{% endstep %}
{% endstepper %}

***

## Notes

{% hint style="warning" %}
**Breaking change:** `getGyroAngle()` was removed from `SwerveDriveConfig` and `SwerveDrive`. Use `getGyroRotation3d()` instead, for example `swerveDrive.getGyroRotation3d().toRotation2d()` for the heading. The heading wraps at +/-180 degrees.
{% endhint %}

{% hint style="info" %}
`lockPose()` turns each wheel toward the robot center (an X pattern) without driving it. Call it every loop from a command, as in `lock()` above.
{% endhint %}

{% hint style="info" %}
Simulated SPARK MAX and SPARK FLEX controllers close their loop at 1 kHz, like the real hardware. See [Simulation](simulation.md).
{% endhint %}

***

## Examples

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/commands2/simple_robot-2027/src/main/java/first/robot/subsystems/SwerveSubsystem.java" %}

Commands v3 version of the swerve drive:

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/commands3/simple_robot-2027/src/main/java/first/robot/mechanisms/SwerveMechanism.java" %}

***

## Related pages

* [SwerveDrive](../java-reference/swerve/swerve-drive.md)
* [SwerveInputStream](../java-reference/swerve/swerve-input-stream.md)
* [SwerveDriveConfig](../java-reference/swerve/swerve-drive-config.md)
* [Configuring a Motor Controller](configuring-a-motor.md)
* [Simulation](simulation.md)
