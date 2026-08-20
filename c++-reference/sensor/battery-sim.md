# BatterySim

**Namespace:** `yams::motorcontrollers::simulation`\
**Header:** `yams/motorcontrollers/simulation/BatterySim.hpp`

`BatterySim` models a single, shared robot battery in simulation. Every simulated `SmartMotorController` — via its `ArmSimSupplier`, `ElevatorSimSupplier`, or `DCMotorSimSupplier`, or directly from `SparkWrapper`/`TalonFXWrapper`/`TalonFXSWrapper` — registers its own current draw here every loop. `BatterySim` combines the currently-registered draw of every mechanism into one loaded-voltage calculation, which is written to `frc::sim::RoboRioSim::SetVInVoltage(...)`, so voltage sag reflects the whole robot's load, not just one mechanism in isolation.

{% hint style="info" %}
This class is entirely static and requires no setup for basic voltage sag under combined load — every built-in sim supplier and hardware wrapper registers with it automatically.
{% endhint %}

## Static Fields

| Field                | Type              | Description                                                                              |
| -------------------- | ----------------- | ------------------------------------------------------------------------------------------- |
| `BatteryVoltage`     | `units::volt_t`   | Nominal open-circuit battery voltage used when discharge simulation is disabled. Default `12V`. |
| `BatteryResistance`  | `units::ohm_t`    | Nominal internal resistance used when discharge simulation is disabled. Default `0.020` ohm.    |

## Voltage Calculation

```cpp
static units::volt_t CalculateVoltage(const void* id, units::ampere_t current);
```

Registers `current` under `id` and returns the resulting loaded battery voltage across every registered id. `id` should be a stable identity for the calling mechanism — YAMS's built-in sim suppliers and hardware wrappers pass their own `SimSupplier` instance address (`this` / `m_simSupplier.get()`).

## Discharge Simulation

By default, `BatterySim` holds a constant nominal voltage and resistance — enough to model sag under instantaneous combined load, but not a battery weakening over a match. Enable discharge modeling to layer state-of-charge tracking on top: current draw is integrated into amp-hours consumed over time (using `frc::Timer::GetFPGATimestamp()`), and the open-circuit voltage droops along a fixed discharge curve (flat through most of the charge, sagging quickly near depletion) while internal resistance rises as the battery empties.

```cpp
static void EnableDischarge(double batteryCapacityAmpHours, units::volt_t nominalVoltage,
                            units::ohm_t nominalResistance);
static void DisableDischarge();
static void ResetDischarge();
static double GetStateOfCharge();
```

| Method               | Description                                                                                       |
| -------------------- | --------------------------------------------------------------------------------------------------- |
| `EnableDischarge`    | Enables discharge modeling with the given capacity and nominal (fully-charged) voltage/resistance.  |
| `DisableDischarge`   | Disables discharge modeling; reverts to a constant `BatteryVoltage`/`BatteryResistance`.             |
| `ResetDischarge`     | Resets the simulated battery back to a full charge and clears the discharge-integration timestamp.  |
| `GetStateOfCharge`   | Current state of charge, from `0` (empty) to `1` (full).                                            |

{% hint style="warning" %}
Discharge simulation only affects `CalculateVoltage(...)`. It has no effect on a real robot, where `frc::RobotController::GetBatteryVoltage()` reflects actual hardware.
{% endhint %}

## Custom Discharge Curves

`EnableDischarge(...)` sags voltage along a built-in curve that models a typical FRC sealed lead-acid battery — roughly flat through most of the charge, then dropping off quickly near depletion. Not every battery behaves that way. Call `ReplaceSOCInterpolation(...)` **before** `EnableDischarge(...)` to swap in a curve that matches the battery you're actually trying to model.

```cpp
static void ReplaceSOCInterpolation(const std::map<double, double>& socToVoltage);
```

Reach for this when:

* **You're modeling a well-used competition battery.** An old battery sags earlier and harder than a fresh one — a flatter, lower curve reproduces that instead of assuming every match starts with a fresh battery.
* **You measured a real curve.** If you've put a battery on a load tester and have actual voltage-vs-state-of-charge data, feeding that in directly gives the most accurate brownout predictions possible.

```cpp
#include <map>
#include <yams/motorcontrollers/simulation/BatterySim.hpp>

// Model a well-used competition battery that sags earlier and more severely than a new one.
std::map<double, double> wornBatteryCurve{
    {0.00, 8.0},  {0.05, 9.5},  {0.10, 10.5}, {0.20, 11.2}, {0.40, 11.6},
    {0.60, 11.9}, {0.80, 12.2}, {0.90, 12.4}, {1.00, 12.6},
};

yams::motorcontrollers::simulation::BatterySim::ReplaceSOCInterpolation(wornBatteryCurve);
// Pair with a reduced usable capacity and higher resistance to match a worn battery.
yams::motorcontrollers::simulation::BatterySim::EnableDischarge(
    15.0, units::volt_t{12.6}, units::ohm_t{0.028});
```

{% hint style="info" %}
Keys and values should span the full `[0, 1]` state-of-charge range — querying outside the range you defined returns the nearest endpoint's voltage instead of extrapolating, so a table missing the low or high end will not sag realistically there.
{% endhint %}

## Example

```cpp
#include <yams/motorcontrollers/simulation/BatterySim.hpp>

void Robot::RobotInit() {
  // Model an 18Ah battery starting at a full 12.9V, 20 mOhm nominal.
  yams::motorcontrollers::simulation::BatterySim::EnableDischarge(
      18.0, units::volt_t{12.9}, units::ohm_t{0.020});
}

void Robot::TestInit() {
  // Start every test run from a full charge.
  yams::motorcontrollers::simulation::BatterySim::ResetDischarge();
}
```

## Related Pages

* [SmartMotorController](../motor-controllers/smart-motor-controller.md)
* [BatterySim (Java)](../../java-reference/sensor/battery-sim.md)
* [Simulation](../../guides/simulation.md)
