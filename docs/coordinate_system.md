# 3D Military Grid Reference System (3D-MGRS)

This document details the 3D coordinate system used in the Stelar Cartography Project for referencing star systems in three-dimensional space.

## Design Goals

The coordinate system was designed with the following goals:

1. **Intuitive Navigation**: Easy for players and GMs to understand relative positions
2. **Astronomically Grounded**: Aligns with real galactic structure and coordinates
3. **Grid-Based**: Allows for systematic organization of space into sectors
4. **Scalable**: Works for both local stellar neighborhood and galactic-scale references
5. **Game-Friendly**: Supports RPG mechanics like travel times and communication ranges

## Core Components

### Origin Point

The system uses Sol (our solar system) as the origin point (0,0,0). This provides an intuitive reference for Earth-based players while aligning with most astronomical catalogs.

### Directional Convention

The system aligns with galactic coordinates:

- **X-axis**: "East/West" orientation
  - Positive X (East): Points toward the Galactic Center (galactic longitude 0°)
  - Negative X (West): Points away from the Galactic Center (galactic longitude 180°)

- **Y-axis**: "North/South" orientation
  - Positive Y (North): Points to 90° galactic longitude
  - Negative Y (South): Points to 270° galactic longitude

- **Z-axis**: "Up/Down" orientation
  - Positive Z (Up): Points toward the North Galactic Pole
  - Negative Z (Down): Points toward the South Galactic Pole

### Grid Structure

Space is divided into cubic sectors, each 10 light-years on a side. This size was chosen to:
- Encompass meaningful stellar neighborhoods
- Provide manageable numbers of stars per sector
- Align approximately with sublight travel capabilities in the game system

### Sector Designation

Each sector is designated by a three-letter code (XXX):

- First letter: X-coordinate sector (A-Z)
- Second letter: Y-coordinate sector (A-Z)
- Third letter: Z-coordinate sector (A-Z)

The alphabet is centered on M, making Sol reside in sector MMM:
- A = -12 sectors from center (-120 to -110 ly)
- M = center sector (-5 to +5 ly)
- Z = +13 sectors from center (+130 to +140 ly)

### Sub-grid Reference

Within each sector, a six-digit numeric code provides precise coordinates:
- First two digits: X position within sector (00-99)
- Middle two digits: Y position within sector (00-99)
- Last two digits: Z position within sector (00-99)

Each unit in this sub-grid represents 0.1 light-years of distance.

## Reference Format

The standard format for a 3D-MGRS reference is:

`XXX-NNNNNN`

Where:
- XXX is the three-letter sector code
- NNNNNN is the six-digit numeric sub-grid reference

Example: `MMM-505050` represents Sol (at the center of sector MMM)

## Extended Format

For references beyond the local region (>260 light-years), an extended format is used:

`NNNN-XXX-NNNNNN`

Where:
- NNNN is the galactic sector number
- XXX is the local sector within that galactic sector
- NNNNNN is the sub-grid reference

Example: `2700-MMM-000050` represents the Galactic Center

## Coordinate Conversion

### Converting Cartesian to 3D-MGRS

```
function CartesianTo3DMGRS(x, y, z):
    # Determine sector letters (A-Z)
    # M is the center (0) position, A is -12, Z is +13
    x_sector = chr(77 + floor(x / 10))  # 77 is ASCII 'M'
    y_sector = chr(77 + floor(y / 10))
    z_sector = chr(77 + floor(z / 10))
    
    # Calculate sub-grid references (00-99)
    x_sub = floor((x % 10) * 10) % 100
    y_sub = floor((y % 10) * 10) % 100
    z_sub = floor((z % 10) * 10) % 100
    
    # Format the result
    return f"{x_sector}{y_sector}{z_sector}-{x_sub:02d}{y_sub:02d}{z_sub:02d}"
```

### Converting 3D-MGRS to Cartesian

```
function MGRSToCartesian(mgrs):
    # Parse the MGRS code
    sector = mgrs.split('-')[0]
    subgrid = mgrs.split('-')[1]
    
    # Extract sector letters
    x_sector = ord(sector[0]) - 77  # 77 is ASCII 'M'
    y_sector = ord(sector[1]) - 77
    z_sector = ord(sector[2]) - 77
    
    # Extract subgrid numbers
    x_sub = int(subgrid[0:2]) / 10
    y_sub = int(subgrid[2:4]) / 10
    z_sub = int(subgrid[4:6]) / 10
    
    # Calculate cartesian coordinates
    x = (x_sector * 10) + x_sub
    y = (y_sector * 10) + y_sub
    z = (z_sector * 10) + z_sub
    
    return (x, y, z)
```

## Practical Examples

### Known Star Systems

| Star Name      | Distance (ly) | 3D-MGRS Code    |
|----------------|---------------|-----------------|
| Sol            | 0             | MMM-505050      |
| Alpha Centauri | 4.37          | MMM-505046      |
| Sirius         | 8.6           | MMM-483550      |
| Epsilon Eridani| 10.5          | MLM-455050      |
| Tau Ceti       | 11.9          | MNM-605050      |
| 40 Eridani     | 16.3          | MLK-455035      |
| Vega           | 25.3          | MNM-506075      |

### Star Trek Systems

| System Name    | Real Star           | 3D-MGRS Code    |
|----------------|--------------------|-----------------|
| Sol (Earth)    | Sol                | MMM-505050      |
| Vulcan         | 40 Eridani         | MLK-455035      |
| Andoria        | Epsilon Indi       | MLM-404550      |
| Tellar         | 61 Cygni           | MMN-504560      |
| Quo'nos        | Fictitious (~100ly)| JMM-355050      |
| Romulus        | Fictitious (~250ly)| GIM-204570      |

## Usage in Game

### Navigation
Players use 3D-MGRS codes to plot courses between systems. The difference between codes indicates distance and direction.

### Communication
Communication delays are calculated based on distance between 3D-MGRS coordinates.

### Territory Control
Political territories can be defined by sector (e.g., "The Federation controls sectors LLM through NNO").

### Exploration
Unknown regions are referenced by their sector codes, allowing for systematic exploration.

## Technical Implementation

The code for implementing 3D-MGRS is in `src/coordinate_conversion.py` and includes:

1. Functions to convert between coordinate systems
2. Validation checks for proper formatting
3. Distance calculations between 3D-MGRS coordinates
4. Helper functions for sector-based queries

## Future Enhancements

Potential improvements to the coordinate system:

1. Sector groupings for larger administrative regions
2. Visual mapping aids (sector coloring, boundary markers)
3. Extended reference system for intergalactic coordinates 