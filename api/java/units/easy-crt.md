# EasyCRT

Package: `yams.units`

`EasyCRT` (Easy Chinese Remainder Theorem) computes the absolute mechanism position from two absolute encoders with coprime gear ratios. This solves the problem of absolute encoders that only read 0–1 rotation — by using two encoders with slightly different gear reductions, the combined reading uniquely identifies position over a larger range.

---

## When to Use

Use `EasyCRT` when:

- Your mechanism travels more than one full revolution
- You need absolute position without homing
- You have two absolute encoders driven by coprime sprocket pairs from a common input shaft

---

## EasyCRTConfig

Builder class for `EasyCRT` configuration.

```java
EasyCRTConfig(Supplier<Angle> encoder1, Supplier<Angle> encoder2)
```

| Parameter | Description |
|-----------|-------------|
| `encoder1` | Supplier returning the current reading of the first absolute encoder |
| `encoder2` | Supplier returning the current reading of the second absolute encoder |

### Builder Methods

All methods return `EasyCRTConfig` for chaining.

| Method | Description |
|--------|-------------|
| `withCommonDriveGear(double reduction, int inputTeeth, int outputTeeth1, int outputTeeth2)` | Configures a common input gear driving two output sprockets. `inputTeeth` drives both; `outputTeeth1` and `outputTeeth2` must be coprime. |
| `withEncoder1Gearing(double ratio)` | Override gearing for encoder 1 independently. |
| `withEncoder2Gearing(double ratio)` | Override gearing for encoder 2 independently. |
| `withMechanismRange(Angle min, Angle max)` | Valid mechanism range. Solutions outside this range are rejected. |
| `withMatchTolerance(Angle tolerance)` | Maximum combined encoder error for a solution to be accepted. |
| `withEncoder1Offset(Angle offset)` | Software offset added to encoder 1 readings. |
| `withEncoder2Offset(Angle offset)` | Software offset added to encoder 2 readings. |

---

## EasyCRT

```java
EasyCRT(EasyCRTConfig config)
```

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `getAngleOptional()` | `Optional<Angle>` | Returns the resolved mechanism angle, or empty if the solve failed. |
| `getLastStatus()` | `CRTStatus` | The result of the most recent solve attempt. |
| `getLastErrorRotations()` | `double` | The combined encoder residual error from the last solve. |
| `getLastIterations()` | `int` | Number of candidates evaluated in the last solve. |

---

## Enum: CRTStatus

| Value | Description |
|-------|-------------|
| `OK` | Solve succeeded; `getAngleOptional()` returns a valid result |
| `NO_SOLUTION` | No candidate was within the valid range and tolerance |
| `AMBIGUOUS` | Two or more candidates tied; result is suppressed |
| `NOT_ATTEMPTED` | No solve has been run since construction |
| `INVALID_CONFIG` | The configuration has inconsistent parameters (e.g., non-coprime tooth counts) |

---

## Example

```java
EasyCRTConfig config = new EasyCRTConfig(
        () -> Rotations.of(encoder1.getAbsolutePosition()),
        () -> Rotations.of(encoder2.getAbsolutePosition()))
    .withCommonDriveGear(1.0, 18, 19, 21)   // 19T and 21T are coprime
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
The two output sprocket tooth counts (`outputTeeth1` and `outputTeeth2`) must be coprime — their greatest common divisor must be 1. Common coprime pairs: 19 & 21, 17 & 19, 18 & 19. Using non-coprime pairs causes `AMBIGUOUS` or `NO_SOLUTION` results.
{% endhint %}

---

See also: [C++ EasyCRT](../../cpp/units/easy-crt.md)
