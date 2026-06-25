# Setting Up Swerve Drive (C++)

**Goal:** Create a fully functional `SwerveDrive` subsystem in C++ with field-relative driving, odometry, and simulation.

---

## Prerequisites

```cpp
#include <yams/YAMS.hpp>

// Or individually:
#include <yams/mechanisms/swerve/SwerveDrive.hpp>
#include <yams/mechanisms/swerve/SwerveModule.hpp>
#include <yams/mechanisms/swerve/utility/SwerveInputStream.hpp>
#include <yams/mechanisms/config/SwerveDriveConfig.hpp>
#include <yams/mechanisms/config/SwerveModuleConfig.hpp>
#include <yams/motorcontrollers/SmartMotorControllerConfig.hpp>
#include <yams/motorcontrollers/remote/TalonFXWrapper.hpp>
```

---

## Steps

### 1. Configure drive and azimuth motor controllers

Each swerve module needs two `SmartMotorController` instances: one for the drive wheel (velocity control) and one for the azimuth (steering angle, position control).

```cpp
// Drive motor config — velocity PID + feedforward
yams::motorcontrollers::SmartMotorControllerConfig driveConfig;
driveConfig
    .WithIdleMode(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromRatio(6.75)))
    .WithFeedback(0.1, 0.0, 0.0)
    .WithSimpleFeedforward(0.12, 2.2, 0.3)   // kS, kV, kA
    .WithStatorCurrentLimit(60_A)
    .WithSubsystem(this)
    .WithTelemetry("DriveFL");

// Azimuth motor config — position PID, continuous wrapping
yams::motorcontrollers::SmartMotorControllerConfig azimuthConfig;
azimuthConfig
    .WithIdleMode(yams::motorcontrollers::SmartMotorControllerConfig::MotorMode::BRAKE)
    .WithMotorGearing(yams::gearing::MechanismGearing(yams::gearing::GearBox::FromRatio(12.8)))
    .WithFeedback(7.0, 0.0, 0.3)
    .WithExternalEncoderDiscontinuityPoint(0.5_tr)
    .WithStatorCurrentLimit(40_A)
    .WithSubsystem(this)
    .WithTelemetry("AzimuthFL");
```

Repeat for each module (FR, BL, BR), adjusting the telemetry name and CAN IDs.

### 2. Construct motor controllers

```cpp
yams::motorcontrollers::remote::TalonFXWrapper driveMotorFL{
    new ctre::phoenix6::hardware::TalonFX(10),
    frc::DCMotor::KrakenX60(1),
    driveConfig
};
yams::motorcontrollers::remote::TalonFXWrapper azimuthMotorFL{
    new ctre::phoenix6::hardware::TalonFX(11),
    frc::DCMotor::KrakenX60(1),
    azimuthConfig
};
```

### 3. Create a `SwerveModuleConfig` for each module

Pass drive and azimuth motor controllers, then set wheel radius, module location (relative to robot center), and absolute encoder offset.

```cpp
yams::mechanisms::config::SwerveModuleConfig frontLeftModuleConfig{
    &driveMotorFL, &azimuthMotorFL
};
frontLeftModuleConfig
    .WithWheelRadius(0.0508_m)                       // 2-inch wheel radius
    .WithLocation(frc::Translation2d{0.381_m, 0.381_m})  // front-left position
    .WithAbsoluteEncoderOffset(130.0_deg)
    .WithTelemetry("FrontLeft",
        yams::motorcontrollers::SmartMotorControllerConfig::TelemetryVerbosity::HIGH);
```

Repeat for `FrontRight`, `BackLeft`, `BackRight` with their positions and offsets:

| Module | Location |
|--------|----------|
| Front Left | `{ +0.381_m, +0.381_m }` |
| Front Right | `{ +0.381_m, -0.381_m }` |
| Back Left | `{ -0.381_m, +0.381_m }` |
| Back Right | `{ -0.381_m, -0.381_m }` |

### 4. Construct `SwerveModule` instances

```cpp
yams::mechanisms::swerve::SwerveModule frontLeft{&frontLeftModuleConfig};
yams::mechanisms::swerve::SwerveModule frontRight{&frontRightModuleConfig};
yams::mechanisms::swerve::SwerveModule backLeft{&backLeftModuleConfig};
yams::mechanisms::swerve::SwerveModule backRight{&backRightModuleConfig};
```

### 5. Create a `SwerveDriveConfig`

Collect the four modules, set kinematic limits, and provide a gyro supplier.

```cpp
yams::mechanisms::swerve::SwerveDriveConfig driveConfig;
driveConfig
    .WithSubsystem(this)
    .WithModules({&frontLeft, &frontRight, &backLeft, &backRight})
    .WithGyro([this]() { return imu_.GetYaw().GetValueAsDouble() * 1_deg; })
    .WithMaximumChassisSpeed(4.5_mps, units::degrees_per_second_t{360.0})
    .WithTranslationController(frc::PIDController{1.0, 0, 0})
    .WithRotationController(frc::PIDController{1.0, 0, 0})
    .WithTelemetry(
        yams::motorcontrollers::SmartMotorControllerConfig::TelemetryVerbosity::HIGH);
```

### 6. Construct `SwerveDrive`

The template parameter must match the number of modules passed to `WithModules`:

```cpp
yams::mechanisms::swerve::SwerveDrive<4> swerveDrive{&driveConfig};
```

### 7. Integrate into a subsystem

```cpp
class DriveSubsystem : public frc2::SubsystemBase {
public:
    DriveSubsystem() { /* configure as shown above */ }

    /** Field-relative drive command driven by a SwerveInputStream. */
    frc2::CommandPtr DriveFieldRelativeCommand(
        yams::mechanisms::swerve::utility::SwerveInputStream<4>& inputStream)
    {
        return Run([&] {
            swerveDrive_.SetFieldRelativeChassisSpeeds(inputStream.Get());
        }).WithName("Field Oriented Drive");
    }

    void AddVisionMeasurement(frc::Pose2d pose, units::second_t timestamp) {
        swerveDrive_.AddVisionMeasurement(pose, timestamp);
    }

    frc::Pose2d GetPose() const { return swerveDrive_.GetPose(); }
    void ZeroGyro() { swerveDrive_.ZeroGyro(); }

    void Periodic() override { swerveDrive_.UpdateTelemetry(); }
    void SimulationPeriodic() override { swerveDrive_.SimIterate(); }

private:
    // ... motor configs, module configs, modules, drive config ...
    yams::mechanisms::swerve::SwerveDrive<4> swerveDrive_{&driveConfig_};
};
```

### 8. Set up driver input with `SwerveInputStream`

`SwerveInputStream<4>` translates raw joystick axes into `frc::ChassisSpeeds`. Pass the `SwerveDrive` reference as the first argument, then set a rotation axis with `WithControllerRotationAxis`.

```cpp
// In RobotContainer or a drive command factory:
yams::mechanisms::swerve::utility::SwerveInputStream<4> driverInput =
    yams::mechanisms::swerve::utility::SwerveInputStream<4>::Of(
        driveSubsystem_->GetSwerveDrive(),
        [&] { return -driverController_.GetLeftY(); },   // forward/back
        [&] { return -driverController_.GetLeftX(); }    // strafe
    )
    .WithControllerRotationAxis([&] { return -driverController_.GetRightX(); })
    .WithDeadband(0.1)
    .WithMaximumLinearVelocity(4.5_mps)
    .WithMaximumAngularVelocity(units::radians_per_second_t{2 * std::numbers::pi})
    .WithAllianceRelativeControl()
    .WithCubeTranslationControllerAxis();

driveSubsystem_->SetDefaultCommand(
    driveSubsystem_->DriveFieldRelativeCommand(driverInput)
);
```

### 9. Heading-snap mode (optional)

Clone the input stream to add heading control without affecting the base stream:

```cpp
yams::mechanisms::swerve::utility::SwerveInputStream<4> headingInput =
    driverInput.Clone()
    .WithControllerHeadingAxis(
        [&] { return driverController_.GetRightX(); },
        [&] { return driverController_.GetRightY(); }
    )
    .WithHeadingControl([&] { return driverController_.GetRightStickButton(); });
```

### 10. Azimuth encoder seeding

Azimuth encoders are seeded automatically during `SwerveModule` construction. If you need to re-seed after the CAN bus has fully settled (e.g., in `Robot::RobotInit`), call `SeedAzimuthEncoder()` on each module:

```cpp
frontLeft.SeedAzimuthEncoder();
frontRight.SeedAzimuthEncoder();
backLeft.SeedAzimuthEncoder();
backRight.SeedAzimuthEncoder();
```

---

## Key SwerveDrive Methods

| Method | Description |
|--------|-------------|
| `SetFieldRelativeChassisSpeeds(ChassisSpeeds)` | Field-relative drive |
| `SetRobotRelativeChassisSpeeds(ChassisSpeeds)` | Robot-relative drive |
| `Drive(std::function<ChassisSpeeds()>)` | Returns a run `CommandPtr` for continuous driving |
| `LockPose()` | X-pattern to resist pushing |
| `GetPose()` | Current field-relative pose from odometry |
| `ResetOdometry(Pose2d)` | Reset odometry to a known pose |
| `ZeroGyro()` | Zero the gyro heading |
| `AddVisionMeasurement(Pose2d, second_t)` | Fuse vision pose into odometry |
| `UpdateTelemetry()` | Call in `Periodic()` |
| `SimIterate()` | Call in `SimulationPeriodic()` |

---

## Notes

{% hint style="warning" %}
`WithSubsystem(this)` must be set on every `SmartMotorControllerConfig` **and** on `SwerveDriveConfig` before construction. Without it, command scheduling will not work correctly.
{% endhint %}

{% hint style="info" %}
`SwerveInputStream::Of()` takes translation axes only. Add rotation later with `WithControllerRotationAxis()` or switch to heading control with `WithControllerHeadingAxis()` + `WithHeadingControl()`.
{% endhint %}

---

## Related Pages

- [SwerveDrive (C++)](../api/cpp/swerve/swerve-drive.md)
- [SwerveModule (C++)](../api/cpp/swerve/swerve-module.md)
- [SwerveInputStream (C++)](../api/cpp/swerve/swerve-input-stream.md)
- [SmartMotorControllerConfig (C++)](../api/cpp/motor-controllers/smart-motor-controller-config.md)
- [Java equivalent: Setting Up Swerve Drive](setting-up-swerve.md)
