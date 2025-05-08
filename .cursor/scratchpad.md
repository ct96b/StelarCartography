# Stelar Cartography Project - Programming Planner

## Background and Motivation
This project aims to map approximately 12,000 real star systems to fictional systems from the FASA Star Trek universe to create a scientifically grounded 3D star map for a sci-fi RPG setting. Previous attempts using spreadsheets ran into memory/performance limitations, requiring a programmatic approach.

## Key Challenges and Analysis

### Data Processing Challenges
- **Scale**: Processing 12,000+ stars efficiently
- **Coordinate Conversion**: Converting astronomical coordinates (RA, Dec, Distance) to 3D Cartesian
- **Spatial Queries**: Finding nearest neighbors and performing spatial analysis

### Mapping Challenges
- **Known Systems**: Some Star Trek systems have canonical real-world counterparts
- **Fictional Systems**: Most systems are fictional and need plausible assignments
- **Political Boundaries**: Need to maintain faction territories in 3D space

### Visualization Challenges
- **3D Rendering**: Displaying thousands of stars efficiently
- **Interactive Exploration**: Allowing users to navigate the 3D map
- **Information Display**: Showing relevant star data on selection

## High-level Task Breakdown

### 1. Data Infrastructure
- [ ] Design data model for star systems
- [ ] Create file structure for organizing by 3D-MGRS sectors
- [ ] Define conversion process for astronomical coordinates
- [ ] Implement spatial indexing for efficient queries

### 2. Core Processing
- [ ] Load and clean star catalog data
- [ ] Convert to 3D Cartesian coordinates
- [ ] Generate 3D-MGRS sector assignments
- [ ] Implement KDTree for nearest-neighbor search

### 3. FASA System Mapping
- [ ] Create data structure for FASA systems
- [ ] Implement mapping for known associations
- [ ] Develop algorithm for fictional system assignment
- [ ] Create validation process for mapping quality

### 4. Visualization
- [ ] Select visualization library (Plotly vs. Matplotlib vs. custom)
- [ ] Implement basic 3D scatter plot of stars
- [ ] Add faction coloring and highlighting
- [ ] Create interactive controls for exploration

### 5. Documentation & Export
- [ ] Generate sector reference sheets
- [ ] Create star system catalog with 3D-MGRS coordinates
- [ ] Document mapping decisions and algorithms
- [ ] Export data in RPG-friendly formats

## Project Status Board

| Task | Status | Priority | Notes |
|------|--------|----------|-------|
| Design data model | Not Started | High | Define star system attributes and relationships |
| Create file structure | Not Started | Medium | Implement directory organization |
| Coordinate conversion | Not Started | High | RA/Dec/Distance → X/Y/Z |
| 3D-MGRS implementation | Not Started | High | Create sector designation system |
| Data loading | Not Started | High | Parse TSV catalog efficiently |
| FASA system processing | Not Started | Medium | Define mapping criteria |
| Nearest-neighbor algorithm | Not Started | Medium | Implement KDTree spatial index |
| Basic visualization | Not Started | Medium | Initial 3D plot |
| Interactive controls | Not Started | Low | Add after basic visualization works |
| Documentation | Not Started | Low | Generate after implementation |

## Implementation Details

### Data Model 
```python
# Star system class outline
class StarSystem:
    # Basic attributes
    id: str                 # Unique identifier
    name: str               # Common name
    coordinates: dict       # Various coordinate representations
        celestial: dict     # RA, Dec, Distance
        cartesian: dict     # X, Y, Z (light-years)
        mgrs: str           # 3D-MGRS reference (e.g., MMM-505050)
    
    # Astronomical attributes
    spectral_class: str     # E.g., "G2V"
    magnitude: dict         # Visual and absolute magnitudes
    
    # Game attributes
    faction: str            # Political affiliation
    notes: str              # Additional information
```

### 3D-MGRS Algorithm
The 3D Military Grid Reference System needs to efficiently:
1. Convert between celestial and Cartesian coordinates
2. Generate sector designations (XXX)
3. Calculate sub-grid references (NNNNNN)
4. Handle edge cases (e.g., negative coordinates)

### Mapping Algorithm Criteria
For fictional systems without real-world counterparts:
1. Match on spectral class (habitable planet preference)
2. Consider distance constraints from lore
3. Maintain political clustering
4. Check for astronomical plausibility

## Lessons
*This section will be populated as we implement and learn*

## Next Action Items
1. Set up project structure
2. Implement basic data loading
3. Create coordinate conversion system
4. Build 3D-MGRS implementation
5. Create simple visualization proof of concept 