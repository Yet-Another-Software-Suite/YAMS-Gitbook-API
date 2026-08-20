# BatterySim

**Package:** `yams.motorcontrollers.simulation`

`BatterySim` models a single, shared robot battery in simulation. Every simulated `SmartMotorController` — via its `ArmSimSupplier`, `ElevatorSimSupplier`, or `DCMotorSimSupplier`, or directly from `SparkWrapper`/`TalonFXWrapper`/`TalonFXSWrapper` — registers its own current draw here every loop. `BatterySim` combines the currently-registered draw of every mechanism into one loaded-voltage calculation and is written to `RoboRioSim.setVInVoltage(...)`, so voltage sag reflects the whole robot's load, not just one mechanism in isolation.

{% hint style="info" %}
This class is entirely static and requires no setup for basic voltage sag under combined load — every built-in sim supplier and hardware wrapper registers with it automatically.
{% endhint %}

By default, before `enableDischarge(...)` is ever called, `BatterySim` holds a constant nominal open-circuit voltage of `12V` and internal resistance of `20 mΩ` — enough to model voltage sag under instantaneous combined load, but not a battery weakening over a match.

## Voltage Calculation

| Method                                    | Returns  | Description                                                                                                     |
| ------------------------------------------ | -------- | ------------------------------------------------------------------------------------------------------------------ |
| `calculateVoltage(UUID id, double current)` | `double` | Registers `current` (amps) under `id` and returns the resulting loaded battery voltage across every registered id. |
| `calculateVoltage(UUID id, Current current)` | `double` | `Current`-typed overload of the above.                                                                             |

`id` should be a stable identity for the calling mechanism — YAMS uses each `SmartMotorController`'s `m_batterySimUUID` field internally, generated once per controller instance.

## Discharge Simulation

Enable discharge modeling to layer state-of-charge tracking on top of the constant nominal voltage/resistance: current draw is integrated into amp-hours consumed over time, and the open-circuit voltage droops along an interpolation table (flat through most of the charge, sagging quickly near depletion — see [Custom Discharge Curves](#custom-discharge-curves) below) while internal resistance rises as the battery empties.

| Method                                                                     | Returns  | Description                                                                                              |
| ----------------------------------------------------------------------------- | -------- | -------------------------------------------------------------------------------------------------------- |
| `enableDischarge(double batteryCapacityAmpHours, Voltage nomVoltage, Resistance nomResistance)` | `void`   | Enables discharge modeling with the given capacity and nominal (fully-charged) voltage/resistance.        |
| `disableDischarge()`                                                       | `void`   | Disables discharge modeling; reverts to a constant nominal voltage/resistance.                  |
| `resetDischarge()`                                                         | `void`   | Resets the simulated battery back to a full charge and clears the discharge-integration timestamp.        |
| `getStateOfCharge()`                                                       | `double` | Current state of charge, from `0` (empty) to `1` (full).                                                  |

{% hint style="warning" %}
Discharge simulation only affects `calculateVoltage(...)`. It has no effect on a real robot, where `RobotController.getBatteryVoltage()` reflects actual hardware.
{% endhint %}

## Custom Discharge Curves

`enableDischarge(...)` sags voltage along a built-in curve that models a typical FRC sealed lead-acid battery — roughly flat through most of the charge, then dropping off quickly near depletion. Not every battery behaves that way. Call `replaceSOCInterpolation(...)` **before** `enableDischarge(...)` to swap in a curve that matches the battery you're actually trying to model.

| Method                                                       | Returns | Description                                                                                                |
| ------------------------------------------------------------- | ------- | -------------------------------------------------------------------------------------------------------------- |
| `replaceSOCInterpolation(InterpolatingDoubleTreeMap socToVoltage)` | `void`  | Replaces the state-of-charge `[0, 1]` &rarr; open-circuit-voltage table used by discharge simulation.        |

Reach for this when:

* **You're modeling a well-used competition battery.** An old battery sags earlier and harder than a fresh one — a flatter, lower curve reproduces that instead of assuming every match starts with a fresh battery.
* **You're using a different chemistry.** Lithium chemistries like LiFePO4 hold a much flatter voltage curve than lead-acid until they're nearly empty, then fall off a cliff — a shape the default curve doesn't capture.
* **You measured a real curve.** If you've put a battery on a load tester and have actual voltage-vs-state-of-charge data, feeding that in directly gives the most accurate brownout predictions for *your* battery.

```java
import edu.wpi.first.math.interpolation.InterpolatingDoubleTreeMap;
import yams.motorcontrollers.simulation.BatterySim;
import static edu.wpi.first.units.Units.Volts;
import static edu.wpi.first.units.Units.Milliohms;

// Model a well-used competition battery that sags earlier and more severely than a new one.
InterpolatingDoubleTreeMap wornBatteryCurve = new InterpolatingDoubleTreeMap();
wornBatteryCurve.put(0.00, 8.0);
wornBatteryCurve.put(0.05, 9.5);
wornBatteryCurve.put(0.10, 10.5);
wornBatteryCurve.put(0.20, 11.2);
wornBatteryCurve.put(0.40, 11.6);
wornBatteryCurve.put(0.60, 11.9);
wornBatteryCurve.put(0.80, 12.2);
wornBatteryCurve.put(0.90, 12.4);
wornBatteryCurve.put(1.00, 12.6);

BatterySim.replaceSOCInterpolation(wornBatteryCurve);
// Pair with a reduced usable capacity and higher resistance to match a worn battery.
BatterySim.enableDischarge(15.0, Volts.of(12.6), Milliohms.of(28));
```

{% hint style="info" %}
Keys and values should span the full `[0, 1]` state-of-charge range — `InterpolatingDoubleTreeMap` clamps to the nearest defined endpoint outside that range, so a table missing the low or high end will not sag realistically there.
{% endhint %}

## Example

```java
import yams.motorcontrollers.simulation.BatterySim;
import static edu.wpi.first.units.Units.Volts;
import static edu.wpi.first.units.Units.Milliohms;

@Override
public void robotInit() {
  // Model an 18Ah battery starting at a full 12.9V, 20 mOhm nominal.
  BatterySim.enableDischarge(18.0, Volts.of(12.9), Milliohms.of(20));
}

@Override
public void testInit() {
  // Start every test run from a full charge.
  BatterySim.resetDischarge();
}
```

## Related Pages

* [SmartMotorController](../motor-controllers/smart-motor-controller.md)
* [BatterySim (C++)](../../c++-reference/sensor/battery-sim.md)
* [Simulation](../../guides/simulation.md)
