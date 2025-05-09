# Constellations Directory

This directory contains the real star systems data split by constellation for more manageable processing and validation.

## Purpose

The constellation-based organization serves several purposes:
1. **Data Management** - Breaking the large dataset into smaller, logical chunks
2. **Validation** - Easier to verify completeness of data within established astronomical groupings
3. **Processing Efficiency** - Allows parallel or incremental processing of star data
4. **Traditional Organization** - Maintains connection to historical astronomical organization

## File Naming Convention

Each file is named after the constellation it represents:
- `Andromeda.csv` - Stars in the Andromeda constellation
- `Aquarius.csv` - Stars in the Aquarius constellation
- etc.

## Important Notes

1. Each constellation file maintains the same CSV structure as the parent star catalog
2. USID sequence numbers are preserved across all files
3. Files are intended for raw data management only and not for the final spatial organization
4. The final data structure will reorganize these stars by sector based on actual 3D proximity

## USID Continuity

USID sequence numbers are continuous across constellation files. Each star has a unique USID that does not restart for each constellation. This ensures data integrity when the files are recombined or processed.

## Processing Workflow

When processing these files:
1. Read each constellation CSV file
2. Process the star data (coordinate conversion, etc.)
3. Output to the appropriate sector-based organization
4. Maintain references to the original constellation for informational purposes 