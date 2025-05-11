# Stelar Cartography Project

## Project Context
**Type**: Programming, Research

## Overview
This project aims to create a comprehensive 3D mapping system that integrates approximately 12,000 real star systems with fictional star systems from the FASA Star Trek universe. The goal is to build a standardized set of star systems for a sci-fi RPG setting that maintains scientific accuracy while supporting game mechanics.

## Project Goals
- Create a catalog of real star systems from raw astronomical data
- Map FASA Star Trek systems onto real star systems
- Develop a consistent 3D coordinate system for navigation
- Visualize the star mapping in an interactive 3D environment
- Generate reference materials for RPG gameplay

## Documentation
- [Implementation Guide](docs/implementation_guide.md) - Technical implementation details
- [CSV Fields Reference](docs/csv_fields.md) - Star catalog field descriptions
- [Mapping Algorithm](docs/mapping_algorithm.md) - System mapping methodology
- [Data Storage Plan](docs/data_storage_plan.md) - Database architecture
- [Potential Catalogs](docs/potential_catalogs.md) - Additional star catalogs for future integration

## Data Organization

### Storage Approach
The project uses a multi-tiered approach to data storage:

1. **Raw Data Files**
   - Original star system data in CSV/TSV format
   - FASA systems data in separate CSV format
   - All data stored in plain text for git compatibility

2. **Spatial Organization**
   - Stars organized by actual 3D proximity
   - Multiple access methods (by distance, by grid sector, etc.)
   - Avoids Earth-centric constellation grouping

3. **Derived Data**
   - Calculated Cartesian coordinates
   - Star system relationships
   - Political faction assignments

## 3D Military Grid Reference System (3D-MGRS)

We've designed a specialized coordinate system for referencing star systems:

### Core Components

1. **Origin**: Sol (0,0,0)

2. **Grid Size**: 10 light-years per grid cube

3. **Directional Convention**:
   - X-axis: "East" (positive) is toward the Galactic Center, "West" (negative) is away
   - Y-axis: "North" (positive) is 90° galactic longitude, "South" (negative) is 270°
   - Z-axis: "Up" (positive) is toward galactic north pole, "Down" (negative) is toward south pole

4. **Sector Designation**:
   - X-axis: Letters A-Z (east/west)
   - Y-axis: Letters A-Z (north/south) 
   - Z-axis: Letters A-Z (up/down)
   - Sol is at sector MMM (assuming M is middle of alphabet)

5. **Sub-Grid Reference**: 
   - 2-digit numbers for each axis (00-99)
   - Allows precision to 0.1 ly

### Reference Format
`XXX-NNNNNN`
- XXX: Three-letter sector (e.g., KLM)
- NNNNNN: Six-digit numeric reference (two digits per axis)

### Examples
- Sol: `MMM-505050` (center of sector MMM)
- Alpha Centauri: `MMM-505046` (4.3 ly from Sol)
- Sirius: `MMM-483550` (8.6 ly from Sol)

### Galactic Scale Extension
For references beyond the local area, the system extends to:
`NNNN-XXX-NNNNNN`
- NNNN: Galactic sector number
- XXX: Local sector letters
- NNNNNN: Precise coordinates

This allows referencing locations like the Galactic Center (~27,000 ly away) while maintaining compatibility with local navigation.

## Technical Implementation

### Tools and Libraries
- Python for data processing and coordinate calculations
- Pandas for data manipulation
- NumPy and Astropy for astronomical calculations
- SciPy for spatial queries and nearest-neighbor searches
- Visualization with Matplotlib/Plotly (2D/3D)

### File Structure
```
/
├── data/
│   ├── raw/
│   │   ├── real_stars.csv          # Original star catalog
│   │   └── fasa_systems.csv        # FASA Star Trek systems
│   ├── processed/
│   │   ├── by_sector/              # Organized by 3D-MGRS sectors
│   │   │   ├── JJK.csv
│   │   │   ├── JKK.csv
│   │   │   ├── KJK.csv
│   │   │   ├── MMM.csv             # Sol sector
│   │   │   └── ...
│   │   └── by_distance/            # Organized by distance from Sol
│   │       ├── 0-10ly.csv
│   │       ├── 10-25ly.csv
│   │       └── ...
│   └── mappings/
│       ├── known_associations.csv  # Established mappings
│       ├── computed_mappings.csv   # Algorithm-generated mappings
│       └── manual_overrides.csv    # Custom adjustments
├── src/
│   ├── data_processing.py
│   ├── coordinate_conversion.py
│   ├── system_mapping.py
│   └── visualization.py
└── docs/
    ├── coordinate_system.md
    ├── mapping_algorithm.md
    └── usage_guide.md
```

## Mapping Approach

### Known Associations
- Systems with established real-world counterparts (e.g., Vulcan → 40 Eridani)
- Direct mapping based on canonical information

### Fictional Systems
- For systems without real-world counterparts (e.g., Qo'noS, Romulus)
- Assigned based on:
  - Distance from known systems
  - Spectral class compatibility (F, G, K preferred for habitable planets)
  - Political boundaries from FASA materials
  - Spatial clustering analysis

## Next Steps

1. **Data Preparation**
   - Load and clean star catalog data
   - Convert to 3D Cartesian coordinates
   - Organize by 3D-MGRS sectors

2. **System Mapping**
   - Implement algorithm for mapping FASA systems to real stars
   - Handle known systems and fictional systems differently
   - Create visualization of initial mapping

3. **Refinement**
   - Manual adjustments for story consistency
   - Validation against Star Trek lore
   - Documentation of final mapping decisions

4. **Output Generation**
   - 3D interactive visualization
   - Sector maps and reference guides
   - Data export in formats suitable for RPG use 