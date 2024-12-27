# O9101 Helical Bore Cycle - User Manual

## Overview

The O9101 subprogram is designed to perform a helical bore cycle on a CNC milling machine, specifically targeting compatibility with older pre-NGC Haas controllers. This program allows for the efficient machining of circular bores with control over various parameters, including optional radial depth reduction and a final spring pass for improved surface finish.

## Parameters

The subprogram accepts the following parameters:

| Parameter | Description                                                                                                                                                                                          | Required | Default Value                               |
| :-------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------- | :------------------------------------------ |
| X         | X-coordinate of the bore center.                                                                                                                                                                     | No       | Current X position (#5001)                  |
| Y         | Y-coordinate of the bore center.                                                                                                                                                                     | No       | Current Y position (#5002)                  |
| R         | Rapid plane (R-level) above the workpiece surface.                                                                                                                                                   | No       | 0.1                                         |
| Z         | Final depth of the bore.                                                                                                                                                                             | Yes      | N/A                                         |
| D         | Diameter of the bore.                                                                                                                                                                                | Yes      | N/A                                         |
| P         | Pitch of the helix (axial distance per revolution).                                                                                                                                                  | No       | 20% of tool diameter                        |
| F         | Feed rate.                                                                                                                                                                                           | Yes      | N/A                                         |
| W         | Radial depth of cut per pass.                                                                                                                                                                        | No       | 20% of tool diameter                        |
| T         | Tool number to use.                                                                                                                                                                                  | No       | Current tool number (#5020)                 |
| A         | Depth reduction flag. 0 = Off, 1 = On. When enabled, the radial depth of cut (W) is reduced based on the current depth using a polynomial function defined by N.                                      | No       | 0 (Off)                                     |
| N         | Polynomial exponent for depth reduction. Determines the shape of the depth reduction curve. Valid values are 1 (linear), 2 (quadratic), 3 (cubic), and 4 (quartic).                                  | No       | 2 (Quadratic)                               |
| M         | Message bypass flag. 0 = Off, 1 = On. When set to 0, the program displays an operator message with cycle parameters and estimated time before starting. Setting to 1 bypasses this message. | No       | 0 (Off)                                     |

## Operation

1. **Parameter Setup:** The user must provide the required parameters (Z, D, F) when calling the subprogram. Optional parameters can be omitted, and the program will use the default values.
2. **Tool Handling:** If a tool number (T) is specified, and it's different from the current tool, the program will perform a tool change (M6).
3. **Calculations:** The program calculates the tool radius, the number of radial and Z passes required, and the estimated cycle time.
4. **Rapid to Start:** The machine rapids to the specified X, Y coordinates and the R-level. Tool length offset (G43) and cutter compensation (G41) are applied.
5. **Operator Message:** If the message bypass flag (M) is 0, an operator message is displayed, showing the cycle parameters and estimated time. The operator must press "Cycle Start" to proceed.
6. **Radial Pass Loop:** The program performs a series of radial passes, stepping in towards the center of the bore.
    *   **Depth Reduction:** If the depth reduction flag (A) is 1, the radial depth of cut is reduced based on the current Z depth using the specified polynomial exponent (N).
    *   **Z Pass Loop:** For each radial pass, the program performs a helical interpolation (G3) to the final Z depth, with a pitch defined by P.
    *   **Chip Evacuation:** Every third Z pass, the tool fully retracts to the R-level to allow for chip evacuation. A dwell (G4 P0.2) is added at the R-level to ensure chips are cleared.
7. **Spring Pass:** After the radial passes, a spring pass is performed at the final bore diameter to improve surface finish. This involves a lead-in arc, a full circular interpolation (G3) at the bore radius, and a lead-out arc.
8. **Retract:** Cutter compensation (G40) is canceled, and the tool retracts to the R-level.
9. **Return:** The subprogram ends with an M99, returning control to the main program.

## Error Handling

The program includes error checks for missing or invalid parameters:

| Error Code | Description                               |
| :--------- | :---------------------------------------- |
| N9000      | Diameter missing (D parameter not provided) |
| N9002      | Feed rate missing (F parameter not provided) |
| N9004      | Final Z depth missing (Z parameter not provided) |
| N9006      | N must be 1-4                             |
| N9007      | W must be positive                        |
| N9008      | Pitch must be positive                    |

If an error is detected, the program will halt and display the corresponding error message.

## Example Usage

```gcode
G0 X0 Y0 Z1 (MOVE TO A SAFE POSITION)
M6 T1 (SELECT TOOL 1)
G43 H1 (APPLY TOOL LENGTH OFFSET FOR TOOL 1)
G0 X1 Y1 (MOVE TO THE BORE CENTER LOCATION)
G65 P9101 X1 Y1 Z-0.5 D1 F50 W0.05 T1 A1 N2 (CALL THE HELICAL BORE SUBPROGRAM)
G0 Z1 (RETRACT TO A SAFE HEIGHT)
M30 (END OF PROGRAM)
```

This example calls the O9101 subprogram to machine a bore with a diameter of 1 inch, a final depth of -0.5 inches, and a feed rate of 50 units per minute. The radial depth of cut is set to 0.05 inches, and depth reduction is enabled with a quadratic reduction curve.

## Notes

*   This program is designed for older pre-NGC Haas controllers and may not be fully compatible with newer control systems.
*   The default values for pitch (P) and radial depth of cut (W) are set to 20% of the tool diameter. Adjust these values based on your specific tooling and material.
*   The depth reduction feature (A, N) can be used to reduce the load on the tool at greater depths, potentially improving tool life and surface finish.
*   The estimated cycle time displayed in the operator message is an approximation and may vary depending on the specific machine and cutting conditions.
*   Always verify the program's output using a simulation or dry run before machining actual parts.


