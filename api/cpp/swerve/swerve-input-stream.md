# SwerveInputStream\<N>

**Namespace:** `yams::mechanisms::swerve::utility`\
**Template:** `template <size_t NumModules = 4>`\
**Java equivalent:** [Java SwerveInputStream](../../java/swerve/swerve-input-stream.md)

`SwerveInputStream<N>` converts raw joystick inputs into `frc::ChassisSpeeds` suitable for `SwerveDrive::Drive()`. Handles deadbands, cubic input curves, axis scaling, alliance flipping, and multiple drive modes (angular-velocity, heading-angle, aim-at-target, translation-only).

---

## Factory Method

```cpp
static SwerveInputStream Of(SwerveDrive<NumModules>& drive,
                            std::function<double()> x,
                            std::function<double()> y)
```

Constructs a `SwerveInputStream` with translation axes only. Chain `WithControllerRotationAxis()` or `WithControllerHeadingAxis()` to add a rotation source before calling `Get()`.

---

## Constructors

```cpp
// Variant 1: with an angular-velocity rotation axis
SwerveInputStream(SwerveDrive<NumModules>& drive,
                  std::function<double()> x,
                  std::function<double()> y,
                  std::function<double()> rot)

// Variant 2: with heading X/Y axes (heading-angle mode)
SwerveInputStream(SwerveDrive<NumModules>& drive,
                  std::function<double()> x,
                  std::function<double()> y,
                  std::function<double()> headingX,
                  std::function<double()> headingY)
```

The second constructor pre-configures heading axes; pair it with `WithHeadingControl()` to activate heading-angle mode.

---

## Builder Methods

All builder methods return `SwerveInputStream&` for chaining.

### Velocity Limits

| Method | Signature | Description |
|--------|-----------|-------------|
| `WithMaximumLinearVelocity` | `WithMaximumLinearVelocity(units::meters_per_second_t velocity)` | Maximum chassis translational speed. Default is 4 m/s. |
| `WithMaximumAngularVelocity` | `WithMaximumAngularVelocity(units::radians_per_second_t velocity)` | Maximum chassis rotational speed. Default is 2π rad/s (1 rotation/s). |

### Axis Configuration

| Method | Signature | Description |
|--------|-----------|-------------|
| `WithControllerRotationAxis` | `WithControllerRotationAxis(std::function<double()> rot)` | Set or replace the rotation axis supplier. Range `[-1, 1]`. Enables `ANGULAR_VELOCITY` mode. |
| `WithControllerHeadingAxis` | `WithControllerHeadingAxis(std::function<double()> headingX, std::function<double()> headingY)` | Set or replace the heading axis suppliers. Range `[-1, 1]`. Enables `HEADING` mode when `WithHeadingControl()` is active. |
| `WithDeadband` | `WithDeadband(double deadband)` | Deadband applied to all axes (0.0–1.0). Pass 0 to disable. |
| `WithScaleTranslation` | `WithScaleTranslation(double scale)` | Multiplies translation output magnitude by `scale`. Range `(0, 1]`. |
| `WithScaleRotation` | `WithScaleRotation(double scale)` | Multiplies rotation axis value by `scale`. Range `(0, 1]`. |

### Input Curves

| Method | Signature | Description |
|--------|-----------|-------------|
| `WithCubeTranslationControllerAxis` | `WithCubeTranslationControllerAxis()` or `WithCubeTranslationControllerAxis(std::function<bool()>)` | Applies a cubic input curve to translation magnitude for finer low-speed control. |
| `WithCubeRotationControllerAxis` | `WithCubeRotationControllerAxis()` or `WithCubeRotationControllerAxis(std::function<bool()>)` | Applies a cubic curve to the rotation axis. |

### Drive Modes

| Method | Signature | Description |
|--------|-----------|-------------|
| `WithHeadingControl` | `WithHeadingControl(std::function<bool()> trigger)` | Enables heading-angle control mode when trigger returns `true`. Requires `WithControllerHeadingAxis()`. |
| `WithAim` | `WithAim(std::function<frc::Pose2d()> aimTarget, std::function<bool()> trigger)` | Rotates the robot to face `aimTarget` while translating. Enables `AIM` mode when trigger is `true`. |
| `WithTranslationOnly` | `WithTranslationOnly(std::function<bool()> trigger)` | Suppresses rotation and locks the current heading while trigger returns `true`. |

### Field Orientation

| Method | Signature | Description |
|--------|-----------|-------------|
| `WithAllianceRelativeControl` | `WithAllianceRelativeControl()` or `WithAllianceRelativeControl(std::function<bool()>)` | Flips the X axis for Red alliance so "forward" always faces the opponent wall. |
| `WithRobotRelative` | `WithRobotRelative()` or `WithRobotRelative(std::function<bool()>)` | Outputs robot-relative `ChassisSpeeds` instead of field-relative. |
| `WithTranslationHeadingOffset` | `WithTranslationHeadingOffset(frc::Rotation2d angle)` or with `std::function<bool()>` trigger | Rotates the output translation vector by a fixed `angle`. |

### Utility

| Method | Signature | Description |
|--------|-----------|-------------|
| `Clone` | `SwerveInputStream Clone() const` | Returns a copy of this stream. Subsequent `With*` calls on the copy do not affect the original. |

---

## Output

| Method | Signature | Description |
|--------|-----------|-------------|
| `Get` | `frc::ChassisSpeeds Get()` | Computes and returns the current field-relative (or robot-relative) `ChassisSpeeds`. Call once per robot loop. |
| `operator()` | `frc::ChassisSpeeds operator()()` | Same as `Get()` — allows using the stream as a `std::function<frc::ChassisSpeeds()>` directly. |

---

## Drive Modes

`SwerveInputStream` automatically selects the active drive mode each loop based on which suppliers are configured and active.

| Mode | Description |
|------|-------------|
| `ANGULAR_VELOCITY` | Default. Rotates using the angular-velocity axis (`WithControllerRotationAxis`). |
| `HEADING` | Controls robot heading via right-stick X/Y angle. Active when `WithHeadingControl()` trigger is `true` and heading axes are configured. |
| `AIM` | Rotates to face a target `frc::Pose2d`. Active when `WithAim()` trigger is `true` and a target is set. |
| `TRANSLATION_ONLY` | Suppresses rotation and holds the current heading. Active when `WithTranslationOnly()` trigger is `true`, or when no rotation source is configured. |

Mode priority (highest to lowest): `TRANSLATION_ONLY` > `AIM` > `HEADING` > `ANGULAR_VELOCITY`.

---

## Example

```cpp
swerve::utility::SwerveInputStream<4> driveInput{
    *m_drive,
    [this] { return -m_controller.GetLeftY(); },
    [this] { return -m_controller.GetLeftX(); },
    [this] { return -m_controller.GetRightX(); }  // rotation axis
};
driveInput.WithMaximumLinearVelocity(4.5_mps)
          .WithMaximumAngularVelocity(units::radians_per_second_t{2 * std::numbers::pi})
          .WithDeadband(0.1)
          .WithScaleTranslation(0.8)
          .WithCubeTranslationControllerAxis()
          .WithAllianceRelativeControl();

// Heading-snap variant using Clone():
auto headingInput = driveInput.Clone()
    .WithControllerHeadingAxis(
        [this] { return m_controller.GetRightX(); },
        [this] { return m_controller.GetRightY(); })
    .WithHeadingControl([this] { return m_controller.GetRightStickButton(); });

// Pass to Drive():
frc2::CommandPtr driveCmd = m_drive->Drive([this] { return driveInput.Get(); });
```

---

## See Also

- [Java SwerveInputStream](../../java/swerve/swerve-input-stream.md)
- [C++ SwerveDrive\<N>](swerve-drive.md)
