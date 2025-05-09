# Source Code Directory

This directory contains the core application code for the Stellar Cartography Project.

## Purpose

The `src` directory houses the primary source code that powers the data processing, coordinate systems, mapping algorithms, and visualization components of the project. Unlike the `scripts` directory which contains standalone utility scripts, the code here is organized as importable modules that work together to form the application's core functionality.

## Module Structure

The source code is organized into several key modules:

- **data_processing.py** - Functions for reading, parsing, and transforming star system data
- **coordinate_conversion.py** - Conversion between astronomical and Cartesian coordinates
- **sector_management.py** - Implementation of the 3D-MGRS sector system
- **system_mapping.py** - Algorithms for mapping fictional systems to real stars
- **visualization.py** - Code for generating 2D and 3D visualizations of star systems

## Design Principles

The code in this directory follows these design principles:

1. **Modular Design** - Clear separation of concerns between different aspects of functionality
2. **Reusability** - Functions and classes designed to be reused across the project
3. **Testability** - Code structured to facilitate unit testing
4. **Documentation** - Comprehensive docstrings and comments

## Usage

The modules in this directory are intended to be imported and used by both the application and scripts:

```python
# Example usage
from src.coordinate_conversion import equatorial_to_cartesian
from src.sector_management import get_sector_code

# Convert star coordinates
x, y, z = equatorial_to_cartesian(ra, dec, distance)

# Get the sector code for a position
sector_code = get_sector_code(x, y, z)
```

## Development Guidelines

When developing code for this directory:

1. Add thorough docstrings to all functions and classes
2. Follow PEP 8 style guidelines
3. Write unit tests for new functionality
4. Keep modules focused on a single responsibility
5. Update this README when adding new modules 