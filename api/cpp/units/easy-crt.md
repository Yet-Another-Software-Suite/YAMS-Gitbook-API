# EasyCRT

**Namespace:** `yams::units`
**Java equivalent:** [Java EasyCRT](../../java/units/easy-crt.md)

`EasyCRT` computes absolute mechanism position from two absolute encoders with coprime gear ratios using the Chinese Remainder Theorem. The C++ API uses a struct-based config rather than Java's builder class.

---

## EasyCRTConfig Struct

```cpp
struct EasyCRTConfig {
    std::function<units::turn_t()> enc1;   // First absolute encoder supplier
    std::function<units::turn_t()> enc2;   // Second absolute encoder supplier
    int enc1Teeth = 19;                    // Must be coprime with enc2Teeth
    int enc2Teeth = 21;
    double commonK = 1.0;                  // Mechanism-to-encoder scale factor
    double offset1 = 0.0;                  // Software offset for enc1 (turns)
    double offset2 = 0.0;                  // Software offset for enc2 (turns)
    double minRot = 0.0;                   // Valid mechanism range minimum (turns)
    double maxRot = 1.0;                   // Valid mechanism range maximum (turns)
    double tolerance = 0.005;             // Max combined encoder error to accept
    bool inv1 = false;                    // Invert enc1
    bool inv2 = false;                    // Invert enc2

    EasyCRTConfig& WithTeeth(int t1, int t2, double k);
    EasyCRTConfig& WithOffsets(double o1, double o2);
    EasyCRTConfig& WithRange(double minR, double maxR);
    EasyCRTConfig& WithTolerance(double tol);
    EasyCRTConfig& WithInversions(bool i1, bool i2);
};
```

`enc1Teeth` and `enc2Teeth` must be coprime (GCD = 1). The usable range of the mechanism in turns is `[minRot, maxRot]`.

---

## Helper Functions

Free functions in `yams::units`:

```cpp
inline int CrtGcd(int a, int b)
inline int CrtLcm(int a, int b)
inline bool CrtIsCoprime(int a, int b)
inline double CrtRatioFromChain(std::initializer_list<int> teeth)
inline double CrtRatioFromStages(std::initializer_list<int> pairs)
inline double CrtCommonK(double commonRatio, int driveTeeth)
```

| Function | Description |
|----------|-------------|
| `CrtGcd` | Greatest common divisor of two integers |
| `CrtLcm` | Least common multiple of two integers |
| `CrtIsCoprime` | Returns `true` if `CrtGcd(a, b) == 1` |
| `CrtRatioFromChain` | Encoder rotations per mechanism rotation from a gear chain (tooth counts in mesh order) |
| `CrtRatioFromStages` | Ratio from alternating (driver, driven) tooth-count pairs |
| `CrtCommonK` | Computes the scale factor K from a common gear ratio and driving sprocket tooth count |

---

## EasyCRT Class

### Status Enum

```cpp
enum class Status {
    NotAttempted,  // No solve run since construction
    Ok,            // Solve succeeded
    NoSolution,    // No candidate within range/tolerance
    Ambiguous,     // Two tied candidates — result suppressed
    InvalidConfig  // Inconsistent configuration (e.g., non-coprime teeth)
};
```

### Constructor

```cpp
explicit EasyCRT(EasyCRTConfig cfg)
```

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `GetAngle()` | `std::optional<units::turn_t>` | Solves and returns mechanism position. Returns `std::nullopt` on failure. |
| `GetStatus() const` | `Status` | Status of the most recent solve attempt |
| `GetLastError() const` | `double` | Combined encoder residual from the most recent solve |
| `GetLastIterations() const` | `int` | Number of CRT candidates evaluated in the most recent solve |

---

## Example

```cpp
units::EasyCRTConfig cfg;
cfg.enc1 = [this] { return units::turn_t{encoder1.GetAbsolutePosition()}; };
cfg.enc2 = [this] { return units::turn_t{encoder2.GetAbsolutePosition()}; };
cfg.WithTeeth(19, 21, 1.0)
   .WithRange(0.0, 10.0)
   .WithTolerance(0.005);

units::EasyCRT crt{cfg};

// In periodic:
auto pos = crt.GetAngle();
if (pos.has_value()) {
    double turns = pos.value().value();
}
```

{% hint style="info" %}
`enc1Teeth` and `enc2Teeth` must be coprime. Call `CrtIsCoprime(t1, t2)` to verify before constructing. The solve range `[minRot, maxRot]` must be narrower than the CRT period (`LCM(enc1Teeth, enc2Teeth) / commonK`) for the solution to be unambiguous.
{% endhint %}
