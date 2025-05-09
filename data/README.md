# Data Directory

This directory contains all the raw, processed, and derived data used in the Stellar Cartography Project.

## Directory Structure

- **constellations/** - Contains the original star catalog data split by constellation for easier management and validation
- **Star Systems - StellarCartography.csv** - The combined source catalog of all real star systems
- **FASA Systems - Systems.csv** - The catalog of fictional star systems from the FASA Star Trek universe

## Data Flow

1. Raw astronomical data is stored in the main CSV files
2. For easier management, the data is split into separate files by constellation in the `constellations/` directory
3. During processing, this data will be transformed into a 3D coordinate system and organized by sectors

## File Formats

### Star Systems CSV Format
The main star systems CSV contains the following key fields:
- USID - Unique Star ID, a sequential identifier for each star system
- Constellation - The constellation where the star is located
- Name - Common or catalog name of the star
- Spectral class - The spectral classification
- Distance (ly) - Distance from Sol in light-years
- Coordinates - Right Ascension and Declination

### FASA Systems CSV Format
The FASA systems file contains fictional star systems from the Star Trek universe with fields that allow mapping to real systems:
- System name
- Political affiliation
- Estimated distance
- Notable planets
- References to canonical sources

## Usage Notes

- When adding new star systems, ensure the USID sequence is maintained
- When processing data, use the scripts in the `scripts/` directory
- Processed data will be output to appropriate subdirectories based on the organization scheme 