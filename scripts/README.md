# Scripts Directory

This directory contains utility scripts for processing, validating, and transforming the star system data used in the Stellar Cartography Project.

## Purpose

These scripts handle various data management and processing tasks:
1. Data splitting and organization
2. Coordinate conversion and transformation
3. Validation and quality checks
4. Sector-based organization
5. System mapping between real and fictional star systems

## Available Scripts

Currently, this directory may contain:

- **check_constellation_usids.py** - Validates the continuity of USID sequence numbers across constellation files
- **split_by_constellation.py** - Splits the main star catalog into separate files by constellation

## Usage

Scripts should be run from the project root directory, for example:

```bash
python scripts/check_constellation_usids.py
```

## Adding New Scripts

When adding new scripts to this directory:

1. Maintain a consistent naming convention
2. Document the script's purpose in a header comment
3. Include command-line argument handling where appropriate
4. Add error handling for data processing edge cases
5. Update this README with information about the new script

## Planned Scripts

Future scripts to be developed:

- **convert_coordinates.py** - Convert astronomical coordinates to 3D Cartesian coordinates
- **generate_sectors.py** - Organize star systems by 3D-MGRS sectors
- **map_systems.py** - Map FASA fictional systems to real star systems
- **export_visualization.py** - Generate data for 3D visualization 