# SwerveInputStream\<N>

**Namespace:** `yams::mechanisms::swerve::utility`\
**Template:** `template <size_t NumModules = 4>`\
**Java equivalent:** [Java SwerveInputStream](../../java/swerve/swerve-input-stream.md)

`SwerveInputStream<N>` converts raw joystick inputs into `frc::ChassisSpeeds` suitable for `SwerveDrive::Drive()`. Handles deadbands, cubic input curves, alliance flipping, and heading control.

---

## Factory Method

```cpp
static SwerveInputStream Of(SwerveDrive<NumModules>& drive,
                            std::function<double()> x,
                            std::function<double()> y)
```

Constructs a `SwerveInputStream` with translation axes. Chain builder methods to add rotation and options.

---

## Constructors

```cpp
SwerveInputStream(SwerveDrive<NumModules>& drive,
                  std::function<double()> x,
                  std::function<double()> y,
                  std::function<double()> rot)

SwerveInputStream(SwerveDrive<NumModules>& drive,
                  std::function<double()> x,
                  std::function<double()> y,
                  std::function<double()> headingX,
                  std::function<double()> headingY)
```

The second constructor enables heading control mode (point in a direction rather than rotate at a rate).

---

## Builder Methods

All builder methods return `SwerveInputStream&` for chaining.

| Method | Signature | Description |
|--------|-----------|-------------|
| `WithMaximumLinearVelocity` | `WithMaximumLinearVelocity(units::meters_per_second_t velocity)` | Scales joystick to this maximum translational speed. |
| `WithMaximumAngularVelocity` | `WithMaximumAngularVelocity(units::radians_per_second_t velocity)` | Scales rotation axis to this maximum rotational speed. |
| `WithDeadband` | `WithDeadband(double deadband)` | Deadband applied to all axes (0.0–1.0). |
| `WithScaleTranslation` | `WithScaleTranslation(double scale)` | Multiplies translation outputs by `scale`. |
| `WithScaleRotation` | `WithScaleRotation(double scale)` | Multiplies rotation output by `scale`. |
| `WithCubeTranslationControllerAxis` | `WithCubeTranslationControllerAxis()` or `WithCubeTranslationControllerAxis(std::function<bool()>)` | Applies cubic input curve to translation for finer low-speed control. |
| `WithCubeRotationControllerAxis` | `WithCubeRotationControllerAxis()` or `WithCubeRotationControllerAxis(std::function<bool()>)` | Applies cubic curve to rotation. |
| `WithAllianceRelativeControl` | `WithAllianceRelativeControl()` or `WithAllianceRelativeControl(std::function<bool()>)` | Flips X axis for Red alliance. |
| `WithRobotRelative` | `WithRobotRelative()` or `WithRobotRelative(std::function<bool()>)` | Switches to robot-relative control when true. |
| `WithHeadingControl` | `WithHeadingControl(std::function<bool()> trigger)` | Enables heading-angle control mode when trigger is true. |
| `WithTranslationOnly` | `WithTranslationOnly(std::function<bool()> trigger)` | Suppresses rotation when trigger is true. |
| `WithAim` | `WithAim(std::function<frc::Pose2d()> aimTarget, std::function<bool()> trigger)` | Rotates robot to face `aimTarget` when trigger is true. |
| `WithTranslationHeadingOffset` | `WithTranslationHeadingOffset(frc::Rotation2d angle)` or with trigger | Offsets the translation direction by `angle`. |
| `Clone` | `SwerveInputStream Clone() const` | Returns a copy of the stream with all settings preserved. |

---

## Output

| Method | Signature | Description |
|--------|-----------|-------------|
| `Get` | `frc::ChassisSpeeds Get()` | Returns computed field-relative speeds. |
| `operator()` | `frc::ChassisSpeeds operator()()` | Same as `Get()` — allows using the stream as a `std::function<frc::ChassisSpeeds()>`. |

---

## Example

```cpp
swerve::utility::SwerveInputStream<4> driveInput{
    m_drive,
    [this] { return -m_controller.GetLeftY(); },
    [this] { return -m_controller.GetLeftX(); },
    [this] { return -m_controller.GetRightX(); }
};
driveInput.WithMaximumLinearVelocity(4.5_mps)
          .WithMaximumAngularVelocity(units::radians_per_second_t{2 * std::numbers::pi})
          .WithDeadband(0.1)
          .WithCubeTranslationControllerAxis()
          .WithAllianceRelativeControl();

frc2::CommandPtr driveCmd = m_drive.Drive([this] { return driveInput.Get(); });
```

---

## See Also

- [Java SwerveInputStream](../../java/swerve/swerve-input-stream.md)
- [C++ SwerveDrive\<N>](swerve-drive.md)
