# GAD 27 Exterior Lighting (Wig-Wag / Alternating Flash)

## Overview

The Garmin GAD 27 Electronic Adapter Unit provides a solid-state (no relays) exterior
lighting control function that alternates power between two lighting circuits -- commonly
referred to as "wig-wag." This is designed to improve aircraft visibility by creating an
alternating flash pattern between two lights (typically left and right landing lights, or
taxi vs. landing).

The GAD 27 can flash landing and/or taxi lights in an alternating fashion. This is the
**only** approved GAD 27 function for 28-volt aircraft.

## Hardware Inputs

### J271 Connector (Discrete Inputs)

| Pin | Name                    | Direction | Description                                   |
|-----|-------------------------|-----------|-----------------------------------------------|
| 34  | LIGHT 1 SWITCH IN       | Input     | Light circuit 1 enable (N877RV: right landing)|
| 35  | LIGHT 2 SWITCH IN       | Input     | Light circuit 2 enable (N877RV: left landing) |
| 36  | ALTERNATING FLASH ON IN | Input     | Enables alternating flash mode                |

All three are discrete inputs, active when grounded.

### TB273 Terminal Block (Power)

| Terminal | Name               | Direction | Description                         |
|----------|--------------------|-----------|-------------------------------------|
| 3        | LIGHT 1 POWER IN   | Input     | Unswitched power feed for circuit 1 |
| 4        | LIGHT 2 POWER IN   | Input     | Unswitched power feed for circuit 2 |
| 9        | LIGHT 1 POWER OUT  | Output    | Switched power to light circuit 1   |
| 10       | LIGHT 2 POWER OUT  | Output    | Switched power to light circuit 2   |

Power flows in on terminals 3/4 and is switched out on terminals 9/10 by the GAD 27
based on the state of the discrete inputs and the software configuration.

## N877RV Switch Wiring

Per the electrical schematic (`multifunction.kicad_sch`), N877RV uses:

- **LANDING_SW** (SPST) -- wired to **both** J271 pin 34 (LIGHT 1 SWITCH IN) and
  pin 35 (LIGHT 2 SWITCH IN). A single switch enables both landing light circuits.
- **WAG_SW** (SPST) -- wired to J271 pin 36 (ALTERNATING FLASH ON IN)

The GAD 27 light outputs map to:
- LIGHT 1 OUTPUT (TB273-9) → right landing lights (LANDING_WING_RT, LANDING_TIP_RT)
- LIGHT 2 OUTPUT (TB273-10) → left landing lights (LANDING_WING_LT, LANDING_TIP_LT)

> **Note:** Taxi lights are handled outside the GAD 27 and are not covered in this document.

## G3X Touch Software Configuration

The wig-wag feature is configured through the G3X Touch display in Configuration Mode
on the **External Lights Configuration Page** (Section 30.4.33 of the G3X/G3X Touch
Installation Manual, 190-01115-01 Rev AZ).

> **Note:** Electrical Control System must be enabled on the LRU Configuration Page in
> order to display the External Lights Configuration Page.

### Flash Selection

Controls which light outputs participate in flashing:

| Setting            | Description                                                        |
|--------------------|--------------------------------------------------------------------|
| Light 1            | Only flash lighting output 1                                      |
| Light 2            | Only flash lighting output 2                                      |
| Flash Both Lights  | Flash both outputs, alternating (light 1 on → light 2 off, etc.)  |

For N877RV: **Flash Both Lights** (alternating wig-wag between left and right landing).

### Automatic Flashing

The lighting outputs may be flashed by:
- An **external switch** (WAG_SW / pin 36) -- not controlled through software
- An **automatic airspeed trigger** -- enabled in software with a configurable IAS threshold
- **Both** -- either source activates flash

When automatic flashing is enabled, the airspeed value should be set greater than the
aircraft's final approach speed so the lights do not flash during landing.

The GAD 27 receives IAS from the G3X Touch system over the CAN bus and automatically
activates the alternating flash when indicated airspeed meets or exceeds the configured
threshold. This operates **independently** of the hardware switch (pin 36).

### Light Priority

Controls the interaction between light flashing behavior and external light switches.
This is a critical configuration setting that determines what happens when the flash state
and the switch state conflict. Three options:

| Setting       | Description (per Garmin Installation Manual)                                     |
|---------------|----------------------------------------------------------------------------------|
| **Flash**     | If flash is active, the light flashes **regardless** of its switch state         |
| **Switch (On)**  | Light is steady ON whenever its switch is on; can be off or flashing when off |
| **Switch (Off)** | Light is steady OFF whenever its switch is off; can be on or flashing when on |

Detailed behavior for each setting:

| Light Priority   | Switch ON, No Flash | Switch ON, Flash Active | Switch OFF, No Flash | Switch OFF, Flash Active |
|------------------|---------------------|-------------------------|----------------------|--------------------------|
| **Flash**        | ON (steady)         | **FLASH**               | OFF                  | **FLASH**                |
| **Switch (On)**  | **ON (steady)**     | **ON (steady)**         | OFF                  | FLASH                    |
| **Switch (Off)** | ON (steady)         | **FLASH**               | **OFF**              | **OFF**                  |

The key differences:
- **Flash** -- flash overrides the switch in both directions; lights flash even with switch OFF
- **Switch (On)** -- switch-on overrides flash; lights cannot flash while the switch is on
- **Switch (Off)** -- switch-off overrides flash; lights cannot flash while the switch is off

### Warm-Up Time

A configurable delay before the flash pattern begins. Intended to protect HID and plasma
lamps that require a warm-up period before being rapidly cycled. Select the value closest
to the recommended warm-up time without being smaller. LED installations can set this to
zero.

## Logic Table

The following table describes the state of each light output based on switch positions,
airspeed conditions, and the Light Priority configuration. Because N877RV wires
LANDING_SW to both pin 34 and pin 35 simultaneously, both landing light circuits are
always enabled or disabled together.

"Flash Active" means the alternating flash mode is engaged, either by the hardware
switch (WAG_SW / pin 36) or by the airspeed auto-enable exceeding the threshold.

```
Flash_Active = WAG_SW_ON  OR  (Airspeed_Auto_Enabled  AND  IAS >= Threshold)
```

### N877RV Recommended Configuration

- **Flash Selection:** Flash Both Lights
- **Light Priority: Switch (Off)**

With **Switch (Off)**, the LANDING_SW controls whether the lights are available at all.
When LANDING_SW is off, both outputs stay off regardless of flash state -- no surprises.
When LANDING_SW is on, the lights operate normally (steady or flashing depending on
flash conditions).

This is the correct choice for N877RV because LANDING_SW is wired to both pin 34 and
pin 35. The other Light Priority settings produce undesirable behavior:

- **Flash** would cause the lights to flash even with LANDING_SW off (if WAG_SW is on
  or airspeed threshold is met). The pilot loses the ability to turn off landing lights.
- **Switch (On)** would make the lights stay steady-on whenever LANDING_SW is on,
  preventing wig-wag entirely. Flash would only work with LANDING_SW off, which is
  the opposite of useful.

### N877RV State Table (Light Priority = Switch (Off))

Since LANDING_SW drives both pin 34 and pin 35 together:

| # | LANDING_SW | WAG_SW | IAS >= Threshold | Flash Active | Right Landing (L1) | Left Landing (L2) | Source        |
|---|------------|--------|------------------|--------------|--------------------|--------------------|---------------|
| 1 | OFF        | OFF    | No               | No           | OFF                | OFF                | Documentation |
| 2 | OFF        | OFF    | Yes              | Yes          | OFF                | OFF                | Documentation |
| 3 | OFF        | ON     | --               | Yes          | OFF                | OFF                | Documentation |
| 4 | ON         | OFF    | No               | No           | **ON (steady)**    | **ON (steady)**    | Documentation |
| 5 | ON         | ON     | --               | Yes          | **FLASH (alt)**    | **FLASH (alt)**    | Documentation |
| 6 | ON         | OFF    | Yes              | Yes          | **FLASH (alt)**    | **FLASH (alt)**    | Documentation |
| 7 | ON         | ON     | Yes              | Yes          | **FLASH (alt)**    | **FLASH (alt)**    | Documentation |

Rows 1-3: LANDING_SW off → Switch (Off) priority keeps lights off regardless of flash state.
Rows 4: No flash active → lights steady on.
Rows 5-7: Flash active with switch on → Switch (Off) allows flashing.

**Legend:**
- **ON (steady)** -- light is on continuously at full brightness
- **FLASH (alt)** -- lights alternate: when right landing is on, left landing is off and vice versa

### N877RV State Table with Flash Priority (for comparison)

If Light Priority were set to **Flash** instead (NOT recommended for N877RV):

| # | LANDING_SW | WAG_SW | IAS >= Threshold | Flash Active | Right Landing (L1) | Left Landing (L2) |
|---|------------|--------|------------------|--------------|--------------------|--------------------|
| 1 | OFF        | OFF    | No               | No           | OFF                | OFF                |
| 2 | OFF        | OFF    | Yes              | Yes          | **FLASH (alt)**    | **FLASH (alt)**    |
| 3 | OFF        | ON     | --               | Yes          | **FLASH (alt)**    | **FLASH (alt)**    |
| 4 | ON         | OFF    | No               | No           | **ON (steady)**    | **ON (steady)**    |
| 5 | ON         | ON     | --               | Yes          | **FLASH (alt)**    | **FLASH (alt)**    |
| 6 | ON         | OFF    | Yes              | Yes          | **FLASH (alt)**    | **FLASH (alt)**    |
| 7 | ON         | ON     | Yes              | Yes          | **FLASH (alt)**    | **FLASH (alt)**    |

Rows 2-3 show the problem: lights flash even though LANDING_SW is off.

### Generic GAD 27 State Table (Independent Light Switches)

For reference, installations where pin 34 and pin 35 are controlled by separate switches.
Assumes Light Priority = **Switch (Off)** and Flash Selection = **Flash Both Lights**:

| # | Light 1 SW (Pin 34) | Light 2 SW (Pin 35) | Flash Active | Light 1 Output  | Light 2 Output  | Source        |
|---|---------------------|---------------------|--------------|-----------------|-----------------|---------------|
| 1 | OFF                 | OFF                 | No           | OFF             | OFF             | Documentation |
| 2 | OFF                 | OFF                 | Yes          | OFF             | OFF             | Documentation |
| 3 | ON                  | OFF                 | No           | **ON (steady)** | OFF             | Documentation |
| 4 | OFF                 | ON                  | No           | OFF             | **ON (steady)** | Documentation |
| 5 | ON                  | ON                  | No           | **ON (steady)** | **ON (steady)** | Documentation |
| 6 | ON                  | ON                  | Yes          | **FLASH (alt)** | **FLASH (alt)** | Documentation |
| 7 | ON                  | OFF                 | Yes          | **FLASH (alt)** | OFF             | Documentation |
| 8 | OFF                 | ON                  | Yes          | OFF             | **FLASH (alt)** | Documentation |

Rows 7-8: With Switch (Off) priority, a light whose switch is off stays off. The
single active light flashes solo (on/off rather than alternating with the other light).

## Post-Installation Check Procedure

From the G3X Touch EFIS Part 23 AML STC Maintenance Manual (190-02472-02),
Section 8.19:

> **8.19 Wig-Wag Landing/Taxi Light Check (if configured)**
>
> 1. Apply power to the aircraft.
> 2. Locate the Landing and Taxi Light switches and turn both to Flash.
>    - Verify the landing and taxi light alternately flash.
> 3. Turn both switches to on and verify they are on continuously.
> 4. Remove power from the aircraft.

For the N877RV switch arrangement, step 2 means: turn on LANDING_SW and WAG_SW.
Step 3 means: keep LANDING_SW on, turn WAG_SW off.

## Common Wiring Patterns

### Single Landing Switch + Flash Switch (N877RV)

N877RV wires LANDING_SW to both pin 34 and pin 35, so both landing light circuits
(left and right) are enabled together by one switch. WAG_SW enables alternating flash
between left and right.

### Two Separate Light Switches + Flash Switch

A common Garmin-documented pattern where one switch controls taxi (pin 34), another
controls landing (pin 35), and a third enables flash (pin 36). Wig-wag alternates between
the taxi and landing lights.

### Three-Position Progressive Switch

Per the FlyLEDs wiring guide, a single three-position progressive switch can replace the
separate taxi/landing switches:
- Position 1 (OFF): No lights
- Position 2 (TAXI): Pin 34 grounded (taxi on)
- Position 3 (+LANDING): Pins 34 and 35 both grounded (taxi and landing on)

With this arrangement, the wig-wag is typically handled by the airspeed auto-enable
rather than a separate flash switch.

## Operational Notes

### Suppressing Flash in IMC

If airspeed auto-enable is configured, `WAG_SW` (pin 36) is an "ON" input only -- it
can activate flash but cannot inhibit it when the airspeed condition is already met.
The flash logic is an OR:

```
Flash_Active = WAG_SW_ON  OR  (Airspeed_Auto_Enabled  AND  IAS >= Threshold)
```

With **Light Priority = Switch (Off)** (recommended), turning off `LANDING_SW` will
turn off the lights entirely, but this loses all landing light capability. There is no way
to have steady-on landing lights while above the airspeed threshold if auto-enable is
configured.

With **Light Priority = Flash**, the situation is even worse: the lights would flash even
with `LANDING_SW` off if the airspeed threshold is met.

**Recommended approach for N877RV:** Do not use airspeed auto-enable. Rely on
`WAG_SW` as the sole flash control. This gives full pilot authority:

- **WAG_SW ON** → landing lights alternate (wig-wag)
- **WAG_SW OFF** → landing lights steady

This allows wig-wag in VMC for visibility while keeping steady landing lights available
in IMC or any other time flashing is undesirable, without having to turn the lights off.

### Alternative: Strategic Airspeed Threshold

If airspeed auto-enable is desired, setting the threshold above typical approach speeds
(but below cruise) would limit auto-flash to cruise flight. During approach in IMC at
lower speeds, the lights would remain steady. This is fragile and depends on consistent
approach speed profiles.

## Sources

- Garmin G3X/G3X Touch Avionics Installation Manual (190-01115-01, Rev AZ, Sep 2025),
  Section 3 (GAD 27 Installation), Section 30.4.33 (External Lights Configuration Page)
- Garmin G3X Touch EFIS Part 23 AML STC Maintenance Manual (190-02472-02, Rev 4)
- FlyLEDs GAD27 Wiring Guide (flyleds.com)
- N877RV electrical schematic (`multifunction.kicad_sch`, `lights.kicad_sch`)
