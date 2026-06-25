# SwerveInputStream

**Package:** `yams.mechanisms.swerve.utility`

Converts raw joystick axis values into `ChassisSpeeds` for swerve drive control. Handles deadbands, cubic input curves, axis scaling, alliance-relative flipping, and multiple drive modes (angular-velocity, heading-snap, translation-only, aim-at-target, drive-to-pose). Implements `Supplier<ChassisSpeeds>` so it can be passed directly wherever a `ChassisSpeeds` supplier is expected.

---

## Constructors

Three ways to construct a `SwerveInputStream`:

```java
// Variant 1: with a rotation axis (angular-velocity mode)
new SwerveInputStream(SwerveDrive drive, DoubleSupplier x, DoubleSupplier y, DoubleSupplier rot)

// Variant 2: with heading axes (heading-angle mode)
new SwerveInputStream(SwerveDrive drive, DoubleSupplier x, DoubleSupplier y,
                      DoubleSupplier headingX, DoubleSupplier headingY)

// Static factory: translation only — add rotation later with withControllerRotationAxis()
// or withControllerHeadingAxis()
SwerveInputStream.of(SwerveDrive drive, DoubleSupplier x, DoubleSupplier y)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `drive` | `SwerveDrive` | The drivetrain instance. Used for heading reference, pose estimation, and config. |
| `x` | `DoubleSupplier` | Forward/back axis in range `[-1, 1]`. Positive is forward. |
| `y` | `DoubleSupplier` | Left/right axis in range `[-1, 1]`. Positive is left. |
| `rot` | `DoubleSupplier` | Rotation axis in range `[-1, 1]`. Used for angular-velocity control. |
| `headingX` | `DoubleSupplier` | Heading X component in range `[-1, 1]`. Used for heading-angle control. |
| `headingY` | `DoubleSupplier` | Heading Y component in range `[-1, 1]`. Used for heading-angle control. |

---

## Builder Methods

All builder methods return `SwerveInputStream` for chaining.

### Velocity Limits

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withMaximumLinearVelocity(LinearVelocity velocity)` | `velocity` | Maximum chassis translational speed. Default is 4 m/s. |
| `withMaximumAngularVelocity(AngularVelocity velocity)` | `velocity` | Maximum chassis rotational speed. Default is 1 rotation/s. |

### Axis Configuration

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withControllerRotationAxis(DoubleSupplier rot)` | `rot` — range `[-1, 1]` | Set or replace the rotation axis supplier after construction. Enables angular-velocity mode. |
| `withControllerHeadingAxis(DoubleSupplier headingX, DoubleSupplier headingY)` | `headingX`, `headingY` — range `[-1, 1]` | Set or replace the heading axis suppliers after construction. Enables heading mode when `withHeadingControl()` is active. |
| `withDeadband(double deadband)` | `deadband` — 0.0 to 1.0 | Deadband applied to all axes before scaling. Inputs below this magnitude are treated as zero. |
| `withScaleTranslation(double scale)` | `scale` — range `(0, 1]` | Multiplies the translation output magnitude by `scale`. |
| `withScaleRotation(double scale)` | `scale` — range `(0, 1]` | Multiplies the rotation axis value by `scale`. |

### Input Curves

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withCubeTranslationControllerAxis()` | — | Applies a cubic curve to translation magnitude for finer low-speed control. Always enabled. |
| `withCubeTranslationControllerAxis(BooleanSupplier enabled)` | `enabled` | Applies a cubic curve to translation magnitude when supplier returns `true`. |
| `withCubeRotationControllerAxis()` | — | Applies a cubic curve to the rotation axis. Always enabled. |
| `withCubeRotationControllerAxis(BooleanSupplier enabled)` | `enabled` | Applies a cubic curve to the rotation axis when supplier returns `true`. |

### Drive Modes

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withHeadingControl(BooleanSupplier trigger)` | `trigger` | Enables heading-angle control mode while trigger returns `true`. Requires `withControllerHeadingAxis()`. |
| `withAim(Supplier<Pose2d> target, BooleanSupplier trigger)` | `target`, `trigger` | Rotates the robot to face the target pose while translating. Enables AIM mode while trigger is `true`. |
| `withTranslationOnly(BooleanSupplier trigger)` | `trigger` | Suppresses rotation and locks the current heading while trigger returns `true`. |

### Field Orientation

| Method | Parameters | Description |
|--------|-----------|-------------|
| `withAllianceRelativeControl()` | — | Flips the X axis when on the Red alliance so "forward" always points toward the opponent wall. Always enabled. |
| `withAllianceRelativeControl(BooleanSupplier enabled)` | `enabled` | Alliance-relative control when supplier returns `true`. |
| `withRobotRelative()` | — | Outputs robot-relative `ChassisSpeeds` instead of field-relative. Always enabled. |
| `withRobotRelative(BooleanSupplier enabled)` | `enabled` | Robot-relative output when supplier returns `true`. |
| `withTranslationHeadingOffset(Rotation2d angle)` | `angle` | Rotates the translation direction by `angle`. Always enabled. |
| `withTranslationHeadingOffset(Rotation2d angle, BooleanSupplier enabled)` | `angle`, `enabled` | Rotates the translation direction by `angle` when supplier returns `true`. |

### Utility

| Method | Parameters | Description |
|--------|-----------|-------------|
| `clone()` | — | Returns a copy of this stream. Subsequent changes to the copy do not affect the original. |

---

## Output Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `get()` | `ChassisSpeeds` | Computes and returns the current `ChassisSpeeds` based on all configured inputs and the active drive mode. Call once per robot loop. |
| `atTargetPose(double toleranceMeters)` | `boolean` | Returns `true` when the robot is within `toleranceMeters` of the drive-to-pose target. Only meaningful in `DRIVE_TO_POSE` mode; logs an error and returns `true` if called in another mode. |

---

## SwerveInputMode Enum

`SwerveInputStream` automatically selects the active drive mode each loop based on which suppliers are configured and active.

| Value | Description |
|-------|-------------|
| `ANGULAR_VELOCITY` | Default. Rotates using the angular-velocity axis (`withControllerRotationAxis`). |
| `HEADING` | Controls robot heading via right-stick X/Y angle. Active when `withHeadingControl()` trigger is `true` and heading axes are configured. |
| `AIM` | Rotates to face a target `Pose2d`. Active when `withAim()` trigger is `true` and a target is set. |
| `TRANSLATION_ONLY` | Suppresses rotation and holds the current heading. Active when `withTranslationOnly()` trigger is `true`, or when no rotation source is configured. |
| `DRIVE_TO_POSE` | Autonomously drives to a target `Pose2d`. Active when `withDriveToPose()` trigger is `true` and PID controllers are configured. |

Mode priority (highest to lowest): `DRIVE_TO_POSE` > `TRANSLATION_ONLY` > `AIM` > `HEADING` > `ANGULAR_VELOCITY`.

---

## Example

```java
XboxController driver = new XboxController(0);

// Angular-velocity stream: left stick translates, right stick X rotates
SwerveInputStream driveInput = SwerveInputStream.of(swerveDrive,
        () -> -driver.getLeftY(),
        () -> -driver.getLeftX())
    .withControllerRotationAxis(driver::getRightX)
    .withDeadband(0.05)
    .withScaleTranslation(0.8)
    .withScaleRotation(0.6)
    .withAllianceRelativeControl()
    .withCubeTranslationControllerAxis();

// Heading-snap variant: clone so changes don't affect the original
SwerveInputStream headingInput = driveInput.clone()
    .withControllerHeadingAxis(driver::getRightX, driver::getRightY)
    .withHeadingControl(() -> driver.getRightStickButton());

// In a command or periodic, call get() to obtain ChassisSpeeds:
swerveDrive.drive(driveInput.get());
```

---

## See Also

- [SwerveDrive](swerve-drive.md)
- [C++ SwerveInputStream](../../cpp/swerve/swerve-input-stream.md)
