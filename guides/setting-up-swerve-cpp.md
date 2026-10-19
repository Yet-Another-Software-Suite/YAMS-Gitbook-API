# Setting Up Swerve Drive (C++)

**Goal:** Create a fully functional `SwerveDrive` subsystem in C++ with field-relative driving, odometry, and simulation.

***

## Prerequisites

Install the Commands v2 YAMS vendordep (`https://cdn.yassrobotics.com/yams_commands2.json`). `YAMS.hpp` does not pull in the vendor wrappers, so include the wrapper for your motor controller explicitly.

```cpp
#include <yams/YAMS.hpp>

// Or individually:
#include <yams/mechanisms/swerve/SwerveDrive.hpp>
#include <yams/mechanisms/swerve/SwerveDriveConfig.hpp>
#include <yams/mechanisms/swerve/SwerveModule.hpp>
#include <yams/mechanisms/swerve/utility/SwerveInputStream.hpp>
#include <yams/mechanisms/config/SwerveModuleConfig.hpp>
#include <yams/gearing/MechanismGearing.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>

// Vendor wrapper (always included individually):
#include <yams/motorcontrollers/remote/TalonFXWrapper.hpp>

// WPILib 2027 headers used below:
#include <wpi/commands2/CommandPtr.hpp>
#include <wpi/commands2/SubsystemBase.hpp>
#include <wpi/commands2/button/CommandXboxController.hpp>
#include <wpi/math/controller/PIDController.hpp>
#include <wpi/math/controller/SimpleMotorFeedforward.hpp>
#include <wpi/math/geometry/Pose2d.hpp>
#include <wpi/math/geometry/Translation2d.hpp>
#include <wpi/math/system/DCMotor.hpp>
```

WPILib 2027 renamed `ChassisSpeeds` to `wpi::math::ChassisVelocities` and `SwerveModuleState` to `wpi::math::SwerveModuleVelocity`. The YAMS method names (`SetFieldRelativeChassisSpeeds`, ...) are unchanged.

***

{% stepper %}
{% step %}
#### Configure drive and azimuth motor controllers

Each swerve module needs two `SmartMotorController` instances: one for the drive wheel (velocity control) and one for the azimuth (steering angle, position control).

```cpp
// Drive motor config: velocity PID + feedforward
yams::motorcontrollers::SmartMotorControllerConfig driveConfig;
driveConfig
    .WithSubsystem(this)
    .WithZeroPower(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(6.75))
    .WithFeedback(0.1, 0.0, 0.0)
    .WithFeedforward(wpi::math::SimpleMotorFeedforward<wpi::units::turns>{
        0.12_V, wpi::units::unit_t<wpi::math::SimpleMotorFeedforward<wpi::units::turns>::kv_unit>{2.2},
        wpi::units::unit_t<wpi::math::SimpleMotorFeedforward<wpi::units::turns>::ka_unit>{0.3}})   // kS, kV, kA
    .WithStatorCurrentLimit(60_A)
    .WithTelemetry("DriveFL");

// Azimuth motor config: position PID, continuous wrapping
yams::motorcontrollers::SmartMotorControllerConfig azimuthConfig;
azimuthConfig
    .WithSubsystem(this)
    .WithZeroPower(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(12.8))
    .WithFeedback(7.0, 0.0, 0.3)
    .WithContinuousWrapping(-0.5_tr, 0.5_tr)
    .WithStatorCurrentLimit(40_A)
    .WithTelemetry("AzimuthFL");
```

Repeat for each module (FR, BL, BR), adjusting the telemetry name and CAN IDs.
{% endstep %}

{% step %}
#### Construct motor controllers

The wrapper takes pointers to the vendor hardware object and to the config. Both must outlive the wrapper, so keep them as subsystem members.

```cpp
ctre::phoenix6::hardware::TalonFX driveTalonFL{10, ctre::phoenix6::CANBus{}};
ctre::phoenix6::hardware::TalonFX azimuthTalonFL{11, ctre::phoenix6::CANBus{}};

yams::motorcontrollers::remote::TalonFXWrapper driveMotorFL{
    &driveTalonFL,
    wpi::math::DCMotor::KrakenX60(1),
    &driveConfig
};
yams::motorcontrollers::remote::TalonFXWrapper azimuthMotorFL{
    &azimuthTalonFL,
    wpi::math::DCMotor::KrakenX60(1),
    &azimuthConfig
};
```
{% endstep %}

{% step %}
#### Create a `SwerveModuleConfig` for each module

Pass drive and azimuth motor controllers, then set the wheel radius, the module location (relative to robot center), the absolute encoder supplier, and its offset.

```cpp
yams::mechanisms::config::SwerveModuleConfig frontLeftModuleConfig{
    &driveMotorFL, &azimuthMotorFL
};
frontLeftModuleConfig
    .WithWheelRadius(0.0508_m)                                  // 2-inch wheel radius
    .WithLocation(wpi::math::Translation2d{0.381_m, 0.381_m})  // front-left position
    .WithAbsoluteEncoder([this]() -> wpi::units::degree_t {
        return wpi::units::degree_t{wpi::units::turn_t{encoderFL_.GetAbsolutePosition().GetValue()}};
    })
    .WithAbsoluteEncoderOffset(130.0_deg)
    .WithTelemetry("FrontLeft",
        yams::motorcontrollers::SmartMotorControllerConfig::TelemetryVerbosity::HIGH);
```

Repeat for `FrontRight`, `BackLeft`, `BackRight` with their positions and offsets:

| Module      | Location                 |
| ----------- | ------------------------ |
| Front Left  | `{ +0.381_m, +0.381_m }` |
| Front Right | `{ +0.381_m, -0.381_m }` |
| Back Left   | `{ -0.381_m, +0.381_m }` |
| Back Right  | `{ -0.381_m, -0.381_m }` |
{% endstep %}

{% step %}
#### Construct `SwerveModule` instances

```cpp
yams::mechanisms::swerve::SwerveModule frontLeft{&frontLeftModuleConfig};
yams::mechanisms::swerve::SwerveModule frontRight{&frontRightModuleConfig};
yams::mechanisms::swerve::SwerveModule backLeft{&backLeftModuleConfig};
yams::mechanisms::swerve::SwerveModule backRight{&backRightModuleConfig};
```
{% endstep %}

{% step %}
#### Create a `SwerveDriveConfig`

Collect the four modules, set kinematic limits, and provide a gyro supplier. In C++ the gyro supplier returns the yaw as a `wpi::units::degree_t`.

```cpp
yams::mechanisms::swerve::SwerveDriveConfig driveConfig;
driveConfig
    .WithSubsystem(this)
    .WithModules({&frontLeft, &frontRight, &backLeft, &backRight})
    .WithGyro([this]() -> wpi::units::degree_t {
        return wpi::units::degree_t{wpi::units::turn_t{imu_.GetYaw().GetValue()}};
    })
    .WithMaximumChassisSpeed(4.5_mps, wpi::units::degrees_per_second_t{360.0})
    .WithTranslationController(wpi::math::PIDController{1.0, 0, 0})
    .WithRotationController(wpi::math::PIDController{1.0, 0, 0})
    .WithTelemetry(
        "swerve",
        yams::motorcontrollers::SmartMotorControllerConfig::TelemetryVerbosity::HIGH);
```
{% endstep %}

{% step %}
#### Construct `SwerveDrive`

The template parameter must match the number of modules passed to `WithModules`:

```cpp
yams::mechanisms::swerve::SwerveDrive<4> swerveDrive{&driveConfig};
```
{% endstep %}

{% step %}
#### Integrate into a subsystem

The subsystem owns the `SwerveInputStream` so it lives as long as the drive command that reads it.

```cpp
class DriveSubsystem : public wpi::cmd::SubsystemBase {
public:
    using InputStream = yams::mechanisms::swerve::utility::SwerveInputStream<4>;

    DriveSubsystem() { /* configure as shown above, then swerveDrive_.emplace(&driveConfig_) */ }

    yams::mechanisms::swerve::SwerveDrive<4>& GetSwerveDrive() { return swerveDrive_.value(); }

    /** Field-relative drive command driven by a SwerveInputStream. */
    wpi::cmd::CommandPtr DriveFieldRelativeCommand(InputStream inputStream) {
        inputStream_.emplace(std::move(inputStream));
        return Run([this] {
            swerveDrive_->SetFieldRelativeChassisSpeeds(inputStream_->Get());
        }).WithName("Field Oriented Drive");
    }

    void AddVisionMeasurement(wpi::math::Pose2d pose, wpi::units::second_t timestamp) {
        swerveDrive_->AddVisionMeasurement(pose, timestamp);
    }

    wpi::math::Pose2d GetPose() { return swerveDrive_->GetPose(); }
    void ZeroGyro() { swerveDrive_->ZeroGyro(); }

    void Periodic() override { swerveDrive_->UpdateTelemetry(); }
    void SimulationPeriodic() override { swerveDrive_->SimIterate(); }

private:
    // ... hardware, motor configs, wrappers, module configs, modules, drive config ...
    std::optional<yams::mechanisms::swerve::SwerveDrive<4>> swerveDrive_;
    std::optional<InputStream> inputStream_;
};
```
{% endstep %}

{% step %}
#### Set up driver input with `SwerveInputStream`

`SwerveInputStream<4>` translates raw joystick axes into field-relative `wpi::math::ChassisVelocities`. Pass the `SwerveDrive` reference as the first argument, then set a rotation axis with `WithControllerRotationAxis`.

```cpp
// In RobotContainer or a drive command factory:
using InputStream = yams::mechanisms::swerve::utility::SwerveInputStream<4>;

InputStream driverInput =
    InputStream::Of(
        driveSubsystem_.GetSwerveDrive(),
        [this] { return -driverController_.GetLeftY(); },   // forward/back
        [this] { return -driverController_.GetLeftX(); }    // strafe
    )
    .WithControllerRotationAxis([this] { return -driverController_.GetRightX(); })
    .WithDeadband(0.1)
    .WithMaximumLinearVelocity(4.5_mps)
    .WithMaximumAngularVelocity(wpi::units::radians_per_second_t{2 * std::numbers::pi})
    .WithAllianceRelativeControl()
    .WithCubeTranslationControllerAxis()
    .WithTelemetry("Driver",
        yams::motorcontrollers::SmartMotorControllerConfig::TelemetryVerbosity::HIGH);

driveSubsystem_.SetDefaultCommand(
    driveSubsystem_.DriveFieldRelativeCommand(driverInput)
);
```

`WithTelemetry(name, verbosity)` publishes the stream under `SwerveInputStream/<name>` every time it is read. At `HIGH` the stream is also live tunable from the dashboard. Call it last, after the rest of the configuration.
{% endstep %}

{% step %}
#### Heading-snap mode (optional)

Clone the input stream to add heading control without affecting the base stream. A `Clone()` has no telemetry, so give it its own name if you want it published.

```cpp
InputStream headingInput =
    driverInput.Clone()
    .WithControllerHeadingAxis(
        [this] { return driverController_.GetRightX(); },
        [this] { return driverController_.GetRightY(); }
    )
    .WithHeadingControl([this] { return driverController_.RightStick().Get(); });
```
{% endstep %}

{% step %}
#### Azimuth encoder seeding

Azimuth encoders are seeded automatically during `SwerveModule` construction. If you need to re-seed after the CAN bus has fully settled, call `SeedAzimuthEncoder()` on each module:

```cpp
frontLeft.SeedAzimuthEncoder();
frontRight.SeedAzimuthEncoder();
backLeft.SeedAzimuthEncoder();
backRight.SeedAzimuthEncoder();
```
{% endstep %}

{% step %}
#### Auto-align (`DriveToPose`) and live PID tuning

`DriveToPose(wpi::math::Pose2d)` returns a `wpi::cmd::CommandPtr` that drives the robot to a field-relative pose using the `WithTranslationController`/`WithRotationController` PID gains set on `SwerveDriveConfig`:

```cpp
wpi::cmd::CommandPtr DriveSubsystem::DriveToPoseCommand(wpi::math::Pose2d target) {
    return swerveDrive_->DriveToPose(target);
}
```

`SwerveDrive<N>` also publishes a live-tuning command to SmartDashboard at `Mechanisms/<name>/tuning`. Running it checks a tunable `autoalign/enabled` boolean and, while `true`, drives toward a target pose (`autoalign/setpoint/x`/`autoalign/setpoint/y` in meters, `autoalign/setpoint/rot` in degrees) you can edit live in NetworkTables, so you can tune those PID gains without redeploying code. The same command also reads a shared set of module drive/azimuth PID, feedforward, and setpoint fields (`modules/drive/*`, `modules/azimuth/*`, with PID gains under `feedback/` and `kS`/`kV`/`kA` under `feedforward/`) that apply to every module on the drive at once. These are all already live-tunable at `TelemetryVerbosity::HIGH` (the default); see [SwerveDrive (C++): Auto-Align](../c++-reference/swerve/swerve-drive.md#auto-align-drive-to-pose--pid-control).
{% endstep %}
{% endstepper %}

***

## Key SwerveDrive methods

| Method                                                               | Description                                                                                 |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `SetFieldRelativeChassisSpeeds(ChassisVelocities)`                   | Field-relative drive                                                                        |
| `SetRobotRelativeChassisSpeeds(ChassisVelocities)`                   | Robot-relative drive                                                                        |
| `Drive(std::function<ChassisVelocities()>)`                          | Returns a run `CommandPtr` for continuous robot-relative driving                            |
| `DriveToPose(Pose2d)`                                                | Returns a `CommandPtr` that PID-drives to a field-relative pose                             |
| `DriveToPoseSetpoint(Pose2d)`                                        | Computes one loop's `ChassisVelocities` toward a pose (building block behind `DriveToPose`) |
| `SetTranslationPID(PIDController)` / `SetRotationPID(PIDController)` | Replace the auto-align PID controllers (integrator preserved unless gains changed)          |
| `ResetTranslationPID()` / `ResetRotationPID()`                       | Reset the auto-align PID controllers' internal state                                        |
| `LockPose()`                                                         | X-pattern to resist pushing                                                                 |
| `GetPose()`                                                          | Current field-relative pose from odometry                                                   |
| `GetGyroAngle()`                                                     | Current gyro heading as a `degree_t`                                                        |
| `ResetOdometry(Pose2d)`                                              | Reset odometry to a known pose                                                              |
| `ZeroGyro()`                                                         | Zero the gyro heading                                                                       |
| `AddVisionMeasurement(Pose2d, second_t)`                             | Fuse vision pose into odometry                                                              |
| `GetSimPose()`                                                       | Ground-truth simulated pose, useful for feeding a simulated vision system                   |
| `UpdateTelemetry()`                                                  | Call in `Periodic()`                                                                        |
| `SimIterate()`                                                       | Call in `SimulationPeriodic()`                                                              |

***

## Notes

{% hint style="warning" %}
`WithSubsystem(this)` must be set on every `SmartMotorControllerConfig` **and** on `SwerveDriveConfig` before construction. Without it, command scheduling will not work correctly.
{% endhint %}

{% hint style="info" %}
`SwerveInputStream::Of()` takes translation axes only. Add rotation later with `WithControllerRotationAxis()` or switch to heading control with `WithControllerHeadingAxis()` + `WithHeadingControl()`.
{% endhint %}

{% hint style="info" %}
The Java-only gyro changes (`GyroAxis`, `withGyroHeadingAxis`, `getGyroRotation3d()`) do not apply to C++. The C++ gyro supplier is still a yaw angle and `SwerveDrive::GetGyroAngle()` is still available.
{% endhint %}

***

## Examples

Complete C++ implementations from the `cpptest` reference project:

{% @github-files/github-code-block url="https://github.com/Yet-Another-Software-Suite/YAMS/blob/master/examples/cpptest/src/main/cpp/subsystems/SwerveSubsystem.cpp" %}

***

## Related pages

* [SwerveDrive (C++)](../c++-reference/swerve/swerve-drive.md)
* [SwerveModule (C++)](../c++-reference/swerve/swerve-module.md)
* [SwerveInputStream (C++)](../c++-reference/swerve/swerve-input-stream.md)
* [SmartMotorControllerConfig (C++)](../c++-reference/motor-controllers/smart-motor-controller-config.md)
* [Java equivalent: Setting Up Swerve Drive](setting-up-swerve.md)
