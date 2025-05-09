# Stellar Cartography Data Storage Plan

## Overview

This document outlines the data storage architecture for the Stellar Cartography project, focusing on a SQLite-based approach optimized for git version control and project portability. The design prioritizes efficient querying, manageable database sizes, and meaningful version tracking.

## Architecture

### Hierarchical Sector-Based Design

We implement a hierarchical sector-based database structure following our galactic coordinate system:

- **1-digit sectors** (10,000 parsec cubes): e.g., 2:1:0 
- **2-digit sectors** (1,000 parsec cubes): e.g., 15:07:02
- **3-digit sectors** (100 parsec cubes): e.g., 153:072:033
- **4-digit sectors** (10 parsec cubes): e.g., 1533:0721:0334
- **5-digit sectors** (1 parsec cubes): e.g., 15334:07208:03336

### Database Structure

```
/data/
  ├── galaxy_metadata.db       (Small central metadata database)
  ├── sectors/
  │     ├── sector_15_07_02.db (Individual sector databases)
  │     ├── sector_15_07_02.sql (Text SQL dumps for git)
  │     ├── sector_15_07_03.db
  │     ├── sector_15_07_03.sql
  │     └── ...
  ├── derived/                 (Git-ignored derived data)
  │     ├── cache/             (Query cache)
  │     ├── visualization/     (Visualization data)
  │     └── ...
```

## Implementation Details

### 1. Core Database Schema

**galaxy_metadata.db**
```sql
CREATE TABLE sector_registry (
    sector_code TEXT PRIMARY KEY,  -- e.g., "15_07_02"
    digit_level INTEGER,           -- 1, 2, 3, 4, or 5
    parent_sector TEXT,            -- e.g., "1_5_7" for "15_07_02"
    x_min INTEGER, x_max INTEGER,  -- Boundaries in parsecs
    y_min INTEGER, y_max INTEGER,
    z_min INTEGER, z_max INTEGER,
    star_count INTEGER,            -- Statistics
    has_database BOOLEAN,          -- Whether a .db file exists
    last_updated TIMESTAMP
);

CREATE TABLE known_associations (
    real_star_id TEXT PRIMARY KEY,
    fasa_system_id TEXT,
    association_type TEXT,         -- "canonical", "computed", "manual"
    association_notes TEXT
);
```

**Individual sector databases (sector_XX_YY_ZZ.db)**
```sql
CREATE TABLE stars (
    id TEXT PRIMARY KEY,
    name TEXT,
    alternate_names TEXT,          -- JSON array of alternate designations
    
    -- Coordinates
    ra REAL,                       -- Right Ascension
    dec REAL,                      -- Declination
    distance REAL,                 -- Distance in light years
    x REAL, y REAL, z REAL,        -- Cartesian coordinates
    mgrs_code TEXT,                -- 3D-MGRS reference
    
    -- Physical characteristics
    spectral_class TEXT,
    visual_magnitude REAL,
    absolute_magnitude REAL,
    
    -- Additional data
    has_planets BOOLEAN,
    notes TEXT,
    
    -- For fictional assignments
    fasa_system_id TEXT,           -- NULL if not mapped
    faction TEXT                   -- Political affiliation in game setting
);

-- Spatial index
CREATE VIRTUAL TABLE stars_spatial USING rtree(
    id,
    x_min, x_max,
    y_min, y_max, 
    z_min, z_max
);
```

### 2. Git Integration

**.gitattributes**
```
*.db filter=lfs diff=sqlite -text
*.sql diff
```

**Pre-commit hook script (scripts/update_sql_dumps.sh)**
```bash
#!/bin/bash
# Generate SQL dumps for all sector databases
for db in data/sectors/*.db; do
  echo ".dump" | sqlite3 "$db" > "${db%.db}.sql"
done

# Add SQL dumps to staging
git add data/sectors/*.sql
```

### 3. Application Interface

The application layer will implement:

1. **Routing logic** to determine which sector database contains requested stars
2. **Dynamic attachment** of relevant sector databases
3. **Cross-sector queries** spanning multiple databases when needed
4. **Lazy loading** to minimize memory usage

## Workflows

### Data Processing Pipeline

1. **Raw Data Import**
   - Parse raw astronomical data CSV files
   - Convert to SQLite records
   - Assign sector codes based on coordinates

2. **Sector Database Creation**
   - Group stars by sector
   - Create individual sector databases
   - Generate SQL dumps for version control

3. **FASA System Mapping**
   - Update known associations
   - Apply mapping algorithms
   - Store results in sector databases

### Query Workflows

1. **Single Sector Query**
   ```python
   def get_stars_in_sector(sector_code):
       db_path = f"data/sectors/sector_{sector_code.replace(':', '_')}.db"
       conn = sqlite3.connect(db_path)
       return conn.execute("SELECT * FROM stars").fetchall()
   ```

2. **Cross-Sector Query**
   ```python
   def get_stars_in_region(x_range, y_range, z_range):
       # Identify affected sectors
       sectors = identify_sectors_in_region(x_range, y_range, z_range)
       
       # Query each sector and combine results
       results = []
       for sector in sectors:
           db_path = f"data/sectors/sector_{sector.replace(':', '_')}.db"
           conn = sqlite3.connect(db_path)
           results.extend(conn.execute(
               "SELECT * FROM stars WHERE x BETWEEN ? AND ? AND y BETWEEN ? AND ? AND z BETWEEN ? AND ?",
               (x_range[0], x_range[1], y_range[0], y_range[1], z_range[0], z_range[1])
           ).fetchall())
       
       return results
   ```

## Best Practices

1. **Size Management**
   - Aim to keep individual sector databases under 100MB
   - Split into smaller sectors if a single sector grows too large
   - Use appropriate compression for text data within SQLite

2. **Version Control**
   - Always update SQL dumps before committing changes
   - Consider using Git LFS for databases over 50MB
   - Include meaningful commit messages about data changes

3. **Performance Optimization**
   - Create proper indexes for common query patterns
   - Use prepared statements for repetitive queries
   - Implement query result caching for frequently accessed data

4. **Data Integrity**
   - Implement validation rules for all data imports
   - Maintain checksums for raw data files
   - Document data provenance for all sources

## Migration Path

This architecture provides a clear migration path to more robust database systems:

1. **Initial Phase**: SQLite with sector sharding
2. **Growth Phase**: Consider PostgreSQL with PostGIS for heavily used sectors
3. **Scale-Up Options**: Implement cross-database federation if needed

The SQL dumps maintained for version control will facilitate migration by providing clean, standard SQL that can be imported into any relational database system.

## Conclusion

This sector-based SQLite approach provides an optimal balance of:
- Query performance for astronomical data
- Git compatibility for version control
- Minimal infrastructure requirements
- Clear scaling pathways

By focusing on 2-digit sector databases initially, we achieve both manageable file sizes and query performance while maintaining compatibility with git-based workflows. 