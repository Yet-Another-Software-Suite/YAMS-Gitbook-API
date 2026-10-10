# EasyCRT

Package: `yams.core.units`

`EasyCRT` (Easy Chinese Remainder Theorem) computes the absolute mechanism position from two absolute encoders with coprime gear ratios. This solves the problem of absolute encoders that only read 0–1 rotation: by using two encoders with slightly different gear reductions, the combined reading uniquely identifies position over a larger range.

***

## When to Use

Use `EasyCRT` when:

* Your mechanism travels more than one full revolution
* You need absolute position without homing
* You have two absolute encoders driven through coprime gear pairs from the mechanism

***

## EasyCRTConfig

Builder class for `EasyCRT` configuration (`yams.core.units.EasyCRTConfig`).

```java
EasyCRTConfig(Supplier<Angle> absoluteEncoder1AngleSupplier, Supplier<Angle> absoluteEncoder2AngleSupplier)
```

| Parameter                       | Description                                                           |
| ------------------------------- | --------------------------------------------------------------------- |
| `absoluteEncoder1AngleSupplier` | Supplier returning the current reading of the first absolute encoder  |
| `absoluteEncoder2AngleSupplier` | Supplier returning the current reading of the second absolute encoder |

### Builder Methods

All methods return `EasyCRTConfig` for chaining. Set the encoder ratios with one of `withEncoderRatios`, `withCommonDriveGear`, or the per-encoder gearing methods.

| Method | Description |
| ------ | ----------- |
| `withEncoderRatios(double encoder1RotPerMechanismRot, double encoder2RotPerMechanismRot)` | Sets each encoder's rotations per mechanism rotation directly. |
| `withCommonDriveGear(double commonRatio, int driveGearTeeth, int absoluteEncoder1PinionTeeth, int absoluteEncoder2PinionTeeth)` | Both encoders are driven by one drive gear after a shared reduction. Ratio per encoder is `commonRatio * driveGearTeeth / pinionTeeth`. The two pinion tooth counts should be coprime. Also seeds the gear recommendation inputs. |
| `withAbsoluteEncoder1Gearing(int... teethChain)` | Meshed gear chain for encoder 1, ordered from the mechanism gear to the encoder pinion. Not valid for compound (same-shaft) trains. |
| `withAbsoluteEncoder2Gearing(int... teethChain)` | Same as above, for encoder 2. |
| `withAbsoluteEncoder1GearingStages(int... driverDrivenPairs)` | Explicit `(driver, driven)` stage pairs for encoder 1. Use for compound trains. |
| `withAbsoluteEncoder2GearingStages(int... driverDrivenPairs)` | Same as above, for encoder 2. |
| `withAbsoluteEncoderOffsets(Angle encoder1Offset, Angle encoder2Offset)` | Offsets added to the raw readings before wrapping. Keep at zero if the encoders are zeroed on-device. |
| `withAbsoluteEncoder1Inverted(boolean inverted)` | Flips the sign of encoder 1's ratio (sensor mounted reversed). |
| `withAbsoluteEncoder2Inverted(boolean inverted)` | Flips the sign of encoder 2's ratio. |
| `withAbsoluteEncoderInversions(boolean encoder1Inverted, boolean encoder2Inverted)` | Sets both inversions in one call. |
| `withMechanismRange(Angle minAngle, Angle maxAngle)` | Valid mechanism range. Solutions outside this range are rejected. |
| `withMatchTolerance(Angle tolerance)` | Maximum modular error on encoder 2 for a candidate to be accepted. Tune with `getLastErrorRotations()`; often close to the backlash seen at the encoder. |
| `withCrtGearRecommendationInputs(int stage1GearTeeth, double stage2Ratio)` | Inputs for the gear pair recommender (drive gear teeth and common ratio). |
| `withCrtGearRecommendationConstraints(double coverageMargin, int minTeeth, int maxTeeth, int maxIterationsLimit)` | Search bounds for the gear pair recommender. Only runs in simulation; a no-op on the robot. |

### Helpers

| Method | Returns | Description |
| ------ | ------- | ----------- |
| `getUniqueCoverage()` | `Optional<Angle>` | Mechanism range over which the encoder pair gives a unique solution. |
| `coverageSatisfiesRange()` | `boolean` | `true` when the unique coverage covers the configured mechanism range. |
| `getRecommendedCrtGearPair()` | `Optional<CrtGearPair>` | Suggested coprime pinion pair from the recommendation inputs. |
| `static isCoprime(int a, int b)` | `boolean` | `true` when `gcd(a, b) == 1`. |
| `static ratioFromChain(int... teethChain)` | `double` | Ratio of a meshed gear chain. |
| `static ratioFromDriverDrivenPairs(int... driverDrivenPairs)` | `double` | Ratio of explicit stage pairs. |
| `static ratioFromCommonDrive(double commonRatio, int driveGearTeeth, int encoderTeeth)` | `double` | Ratio for one encoder on a common drive gear. |

Getters for the configured values are also available (`getEncoder1RotationsPerMechanismRotation()`, `getAbsoluteEncoder1Offset()`, `getMinMechanismAngle()`, `getMatchTolerance()`, and so on).

***

## EasyCRT

```java
EasyCRT(EasyCRTConfig config)
```

### Methods

| Method                    | Returns           | Description                                                         |
| ------------------------- | ----------------- | ------------------------------------------------------------------- |
| `getAngleOptional()`      | `Optional<Angle>` | Returns the resolved mechanism angle, or empty if the solve failed. |
| `getLastStatus()`         | `CRTStatus`       | The result of the most recent solve attempt.                        |
| `getLastErrorRotations()` | `double`          | The combined encoder residual error from the last solve.            |
| `getLastIterations()`     | `int`             | Number of candidates evaluated in the last solve.                   |

***

## Enum: CRTStatus

| Value            | Description                                                                    |
| ---------------- | ------------------------------------------------------------------------------ |
| `OK`             | Solve succeeded; `getAngleOptional()` returns a valid result                   |
| `NO_SOLUTION`    | No candidate was within the valid range and tolerance                          |
| `AMBIGUOUS`      | Two or more candidates tied; result is suppressed                              |
| `NOT_ATTEMPTED`  | No solve has been run since construction                                       |
| `INVALID_CONFIG` | A solve was not attempted because the configuration is incomplete or invalid |

***

## Example

```java
EasyCRTConfig config = new EasyCRTConfig(
        () -> Rotations.of(encoder1.getAbsolutePosition()),
        () -> Rotations.of(encoder2.getAbsolutePosition()))
    .withCommonDriveGear(1.0, 50, 19, 21)   // 50T drive gear; 19T and 21T encoder pinions are coprime
    .withMechanismRange(Rotations.of(0), Rotations.of(10))
    .withMatchTolerance(Degrees.of(2));

EasyCRT crt = new EasyCRT(config);

// In periodic():
Optional<Angle> position = crt.getAngleOptional();
if (position.isPresent()) {
    double mechanismRotations = position.get().in(Rotations);
}
```

{% hint style="warning" %}
The two encoder pinion tooth counts (`absoluteEncoder1PinionTeeth` and `absoluteEncoder2PinionTeeth`) should be coprime: their greatest common divisor must be 1. Common coprime pairs: 19 & 21, 17 & 19, 18 & 19. Using non-coprime pairs causes `AMBIGUOUS` or `NO_SOLUTION` results.
{% endhint %}

***

See also: [C++ EasyCRT](../../c++-reference/units/easy-crt.md)
