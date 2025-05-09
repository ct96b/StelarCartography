# Stellar Cartography Implementation Guide

This guide provides practical steps to implement the data storage architecture outlined in `data_storage_plan.md`. It includes scripts, code examples, and implementation details for developers.

## Setup Steps

### 1. Directory Structure Setup

Create the required directory structure:

```bash
# Create main directory structure
mkdir -p data/sectors
mkdir -p data/derived/cache
mkdir -p data/derived/visualization
mkdir -p scripts

# Create .gitignore file
cat > .gitignore << 'EOF'
# Ignore derived data
data/derived/

# Ignore large DB files but keep their SQL dumps
# data/sectors/*.db

# Python cache
__pycache__/
*.py[cod]
EOF

# Create .gitattributes file
cat > .gitattributes << 'EOF'
*.db filter=lfs diff=sqlite -text
*.sql diff
EOF
```

### 2. Setup Git LFS (Optional)

If needed for larger database files:

```bash
# Install Git LFS
git lfs install

# Track database files
git lfs track "*.db"
```

### 3. Create Initial Metadata Database

```python
# scripts/create_metadata_db.py
import sqlite3
import os

def initialize_metadata_db():
    """Create and initialize the galaxy metadata database"""
    if not os.path.exists('data'):
        os.makedirs('data')
    
    conn = sqlite3.connect('data/galaxy_metadata.db')
    cursor = conn.cursor()
    
    # Create sector registry table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS sector_registry (
        sector_code TEXT PRIMARY KEY,
        digit_level INTEGER,
        parent_sector TEXT,
        x_min INTEGER, x_max INTEGER,
        y_min INTEGER, y_max INTEGER,
        z_min INTEGER, z_max INTEGER,
        star_count INTEGER,
        has_database BOOLEAN,
        last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )
    ''')
    
    # Create known associations table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS known_associations (
        real_star_id TEXT PRIMARY KEY,
        fasa_system_id TEXT,
        association_type TEXT,
        association_notes TEXT
    )
    ''')
    
    # Create initial known associations
    cursor.execute('''
    INSERT OR IGNORE INTO known_associations (real_star_id, fasa_system_id, association_type, association_notes)
    VALUES 
        ('HD_26965', '40 Eridani', 'canonical', 'Vulcan homeworld'),
        ('HD_131511', '61 Cygni', 'canonical', 'Tellar Prime (Tellarite homeworld)')
    ''')
    
    conn.commit()
    conn.close()
    
    print("Initialized galaxy_metadata.db successfully")

if __name__ == "__main__":
    initialize_metadata_db()
```

### 4. Create Data Processing Scripts

#### Coordinate Conversion

```python
# src/coordinate_conversion.py
import math
import numpy as np

def celestial_to_cartesian(ra_hours, ra_minutes, ra_seconds, dec_degrees, dec_minutes, dec_seconds, distance):
    """
    Convert celestial coordinates (RA, Dec, Distance) to Cartesian (X, Y, Z)
    
    Args:
        ra_hours, ra_minutes, ra_seconds: Right Ascension
        dec_degrees, dec_minutes, dec_seconds: Declination
        distance: Distance in light years
        
    Returns:
        (x, y, z): Cartesian coordinates with origin at Sol
    """
    # Convert RA to decimal degrees
    ra_decimal = 15 * (ra_hours + ra_minutes/60 + ra_seconds/3600)
    
    # Convert Dec to decimal degrees
    dec_sign = -1 if dec_degrees < 0 else 1
    dec_decimal = abs(dec_degrees) + dec_minutes/60 + dec_seconds/3600
    dec_decimal *= dec_sign
    
    # Convert to radians
    ra_rad = math.radians(ra_decimal)
    dec_rad = math.radians(dec_decimal)
    
    # Calculate Cartesian coordinates
    x = distance * math.cos(dec_rad) * math.cos(ra_rad)
    y = distance * math.cos(dec_rad) * math.sin(ra_rad)
    z = distance * math.sin(dec_rad)
    
    return (x, y, z)

def parse_celestial_coords(ra_str, dec_str):
    """
    Parse RA and Dec from string format
    
    Args:
        ra_str: Right Ascension string (e.g., "00h 08m 23.17s")
        dec_str: Declination string (e.g., "+29° 05′ 27.0″")
        
    Returns:
        (ra_h, ra_m, ra_s, dec_d, dec_m, dec_s)
    """
    # Parse RA
    ra_parts = ra_str.replace('h', ' ').replace('m', ' ').replace('s', ' ').strip().split()
    ra_h = float(ra_parts[0])
    ra_m = float(ra_parts[1])
    ra_s = float(ra_parts[2])
    
    # Parse Dec
    dec_str = dec_str.replace('°', ' ').replace('′', ' ').replace('″', ' ').strip()
    if dec_str.startswith('-'):
        dec_sign = -1
        dec_str = dec_str[1:]
    elif dec_str.startswith('+'):
        dec_sign = 1
        dec_str = dec_str[1:]
    else:
        dec_sign = 1
    
    dec_parts = dec_str.split()
    dec_d = float(dec_parts[0]) * dec_sign
    dec_m = float(dec_parts[1])
    dec_s = float(dec_parts[2])
    
    return (ra_h, ra_m, ra_s, dec_d, dec_m, dec_s)

def calculate_mgrs_code(x, y, z):
    """
    Calculate the 3D-MGRS code for a given set of Cartesian coordinates
    
    Args:
        x, y, z: Cartesian coordinates in light years
        
    Returns:
        mgrs_code: The 3D-MGRS code (e.g., "MMM-505050")
    """
    # Apply the directional convention:
    # X-axis: "East" (positive) is toward the Galactic Center, "West" (negative) is away
    # Y-axis: "North" (positive) is 90° galactic longitude, "South" (negative) is 270°
    # Z-axis: "Up" (positive) is toward galactic north pole, "Down" (negative) is toward south pole
    
    # Convert to grid sectors (each 10 ly)
    sector_x = int(x / 10)
    sector_y = int(y / 10)
    sector_z = int(z / 10)
    
    # Sol is at sector MMM (assuming M is middle of alphabet)
    # A-Z corresponds to -12 to +13 (26 letters, with M at 0)
    base_ord = ord('A')
    middle = ord('M') - base_ord  # 12
    
    # Calculate sector letters
    x_letter = chr(base_ord + (sector_x + middle) % 26)
    y_letter = chr(base_ord + (sector_y + middle) % 26)
    z_letter = chr(base_ord + (sector_z + middle) % 26)
    
    # Calculate sub-grid reference (two digits per axis, 0.1 ly precision)
    x_sub = int((x - sector_x * 10) * 10) % 100
    y_sub = int((y - sector_y * 10) * 10) % 100
    z_sub = int((z - sector_z * 10) * 10) % 100
    
    # Format the MGRS code
    mgrs_code = f"{x_letter}{y_letter}{z_letter}-{x_sub:02d}{y_sub:02d}{z_sub:02d}"
    
    return mgrs_code

def calculate_sector_code(x, y, z, digit_level=2):
    """
    Calculate the sector code at the specified digit level
    
    Args:
        x, y, z: Cartesian coordinates in light years
        digit_level: Sector precision (1-5)
        
    Returns:
        sector_code: The sector code (e.g., "15:07:02" for digit_level=2)
    """
    # Convert light years to parsecs (1 ly = 0.306601 parsecs)
    x_pc = x * 0.306601
    y_pc = y * 0.306601
    z_pc = z * 0.306601
    
    # Scale factors for different digit levels
    scale_factors = {
        1: 10000,  # 1-digit sectors (10,000 parsec cubes)
        2: 1000,   # 2-digit sectors (1,000 parsec cubes)
        3: 100,    # 3-digit sectors (100 parsec cubes)
        4: 10,     # 4-digit sectors (10 parsec cubes)
        5: 1       # 5-digit sectors (1 parsec cubes)
    }
    
    scale = scale_factors.get(digit_level, 1000)  # Default to 2-digit sectors
    
    # Calculate sector numbers
    x_sector = int(x_pc // scale)
    y_sector = int(y_pc // scale)
    z_sector = int(z_pc // scale)
    
    # Format the sector code based on digit level
    format_strings = {
        1: "{:01d}:{:01d}:{:01d}",
        2: "{:02d}:{:02d}:{:02d}",
        3: "{:03d}:{:03d}:{:03d}",
        4: "{:04d}:{:04d}:{:04d}",
        5: "{:05d}:{:05d}:{:05d}"
    }
    
    format_string = format_strings.get(digit_level, "{:02d}:{:02d}:{:02d}")
    sector_code = format_string.format(x_sector, y_sector, z_sector)
    
    return sector_code
```

#### Data Processing

```python
# src/data_processing.py
import os
import sqlite3
import csv
import json
from datetime import datetime
from coordinate_conversion import (
    parse_celestial_coords, 
    celestial_to_cartesian, 
    calculate_mgrs_code,
    calculate_sector_code
)

def create_sector_db(sector_code):
    """
    Create a new sector database
    
    Args:
        sector_code: The sector code (e.g., "15:07:02")
        
    Returns:
        conn: SQLite connection to the new database
    """
    db_path = f"data/sectors/sector_{sector_code.replace(':', '_')}.db"
    
    # Ensure directory exists
    os.makedirs(os.path.dirname(db_path), exist_ok=True)
    
    # Create database
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    
    # Create stars table
    cursor.execute('''
    CREATE TABLE IF NOT EXISTS stars (
        id TEXT PRIMARY KEY,
        name TEXT,
        alternate_names TEXT,
        
        ra REAL,
        dec REAL,
        distance REAL,
        x REAL, y REAL, z REAL,
        mgrs_code TEXT,
        
        spectral_class TEXT,
        visual_magnitude REAL,
        absolute_magnitude REAL,
        
        has_planets BOOLEAN DEFAULT 0,
        notes TEXT,
        
        fasa_system_id TEXT,
        faction TEXT
    )
    ''')
    
    # Create spatial index
    cursor.execute('''
    CREATE VIRTUAL TABLE IF NOT EXISTS stars_spatial USING rtree(
        id,
        x_min, x_max,
        y_min, y_max, 
        z_min, z_max
    )
    ''')
    
    conn.commit()
    return conn

def update_metadata_db(sector_code, star_count, digit_level=2, parent_sector=None):
    """
    Update the metadata database with sector information
    
    Args:
        sector_code: The sector code (e.g., "15:07:02")
        star_count: Number of stars in the sector
        digit_level: Sector precision (1-5)
        parent_sector: Parent sector code (e.g., "1:5:7" for "15:07:02")
    """
    # Parse sector boundaries from code
    parts = sector_code.split(':')
    x_sector = int(parts[0])
    y_sector = int(parts[1])
    z_sector = int(parts[2])
    
    # Scale factors for different digit levels
    scale_factors = {
        1: 10000,  # 1-digit sectors (10,000 parsec cubes)
        2: 1000,   # 2-digit sectors (1,000 parsec cubes)
        3: 100,    # 3-digit sectors (100 parsec cubes)
        4: 10,     # 4-digit sectors (10 parsec cubes)
        5: 1       # 5-digit sectors (1 parsec cubes)
    }
    
    scale = scale_factors.get(digit_level, 1000)  # Default to 2-digit sectors
    
    # Calculate sector boundaries
    x_min = x_sector * scale
    x_max = (x_sector + 1) * scale
    y_min = y_sector * scale
    y_max = (y_sector + 1) * scale
    z_min = z_sector * scale
    z_max = (z_sector + 1) * scale
    
    # Connect to metadata database
    conn = sqlite3.connect('data/galaxy_metadata.db')
    cursor = conn.cursor()
    
    # Update sector registry
    cursor.execute('''
    INSERT OR REPLACE INTO sector_registry 
        (sector_code, digit_level, parent_sector, x_min, x_max, y_min, y_max, z_min, z_max, star_count, has_database, last_updated)
    VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, 1, ?)
    ''', (
        sector_code, digit_level, parent_sector, 
        x_min, x_max, y_min, y_max, z_min, z_max, 
        star_count, datetime.now().isoformat()
    ))
    
    conn.commit()
    conn.close()

def process_star_catalog(csv_path):
    """
    Process a star catalog CSV file into sector databases
    
    Args:
        csv_path: Path to the star catalog CSV
    """
    # Dictionary to collect stars by sector
    sectors = {}
    
    # Read CSV file
    with open(csv_path, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        for row in reader:
            try:
                # Extract celestial coordinates
                ra_str = row.get('RA', '')
                dec_str = row.get('Dec', '')
                distance_str = row.get('Dist. (ly)', '')
                
                if not ra_str or not dec_str or not distance_str:
                    continue
                
                distance = float(distance_str)
                
                # Parse coordinates
                ra_h, ra_m, ra_s, dec_d, dec_m, dec_s = parse_celestial_coords(ra_str, dec_str)
                
                # Convert to Cartesian
                x, y, z = celestial_to_cartesian(ra_h, ra_m, ra_s, dec_d, dec_m, dec_s, distance)
                
                # Calculate MGRS code
                mgrs_code = calculate_mgrs_code(x, y, z)
                
                # Calculate sector code (2-digit level by default)
                sector_code = calculate_sector_code(x, y, z, digit_level=2)
                
                # Create star data
                star = {
                    'id': row.get('USID', '') or f"HD_{row.get('HD', '')}",
                    'name': row.get('Name', ''),
                    'alternate_names': json.dumps([
                        n for n in [
                            row.get('Full Designation', ''),
                            f"HD {row.get('HD', '')}" if row.get('HD') else None,
                            f"HIP {row.get('HIP', '')}" if row.get('HIP') else None
                        ] if n
                    ]),
                    'ra': float(ra_h) + float(ra_m)/60 + float(ra_s)/3600,
                    'dec': float(dec_d) + float(dec_m)/60 * (1 if dec_d >= 0 else -1) + float(dec_s)/3600 * (1 if dec_d >= 0 else -1),
                    'distance': distance,
                    'x': x,
                    'y': y,
                    'z': z,
                    'mgrs_code': mgrs_code,
                    'spectral_class': row.get('Sp. class', ''),
                    'visual_magnitude': float(row.get('vis. mag.', 0)) if row.get('vis. mag.', '') else None,
                    'absolute_magnitude': float(row.get('abs. mag.', 0)) if row.get('abs. mag.', '') else None,
                    'notes': row.get('Notes', '')
                }
                
                # Add to sector
                if sector_code not in sectors:
                    sectors[sector_code] = []
                
                sectors[sector_code].append(star)
                
            except Exception as e:
                print(f"Error processing row: {e}")
                continue
    
    # Process sectors and create databases
    for sector_code, stars in sectors.items():
        try:
            # Create sector database
            conn = create_sector_db(sector_code)
            cursor = conn.cursor()
            
            # Insert stars
            for star in stars:
                cursor.execute('''
                INSERT INTO stars (
                    id, name, alternate_names, 
                    ra, dec, distance, x, y, z, mgrs_code,
                    spectral_class, visual_magnitude, absolute_magnitude,
                    notes
                ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
                ''', (
                    star['id'], star['name'], star['alternate_names'],
                    star['ra'], star['dec'], star['distance'], star['x'], star['y'], star['z'], star['mgrs_code'],
                    star['spectral_class'], star['visual_magnitude'], star['absolute_magnitude'],
                    star['notes']
                ))
                
                # Add to spatial index
                cursor.execute('''
                INSERT INTO stars_spatial (
                    id, x_min, x_max, y_min, y_max, z_min, z_max
                ) VALUES (?, ?, ?, ?, ?, ?, ?)
                ''', (
                    star['id'], star['x'], star['x'], star['y'], star['y'], star['z'], star['z']
                ))
            
            conn.commit()
            conn.close()
            
            # Generate SQL dump
            os.system(f'echo ".dump" | sqlite3 "data/sectors/sector_{sector_code.replace(":", "_")}.db" > "data/sectors/sector_{sector_code.replace(":", "_")}.sql"')
            
            # Update metadata
            parent_sector = f"{sector_code[0]}:{sector_code[2]}:{sector_code[4]}" if len(sector_code) >= 5 else None
            update_metadata_db(sector_code, len(stars), digit_level=2, parent_sector=parent_sector)
            
            print(f"Processed sector {sector_code} with {len(stars)} stars")
            
        except Exception as e:
            print(f"Error processing sector {sector_code}: {e}")
            continue

def main():
    """Main processing function"""
    # Ensure output directories exist
    os.makedirs('data/sectors', exist_ok=True)
    
    # Process raw star catalog
    process_star_catalog('Star Systems - StellarCartography.csv')
    
    print("Data processing complete")

if __name__ == "__main__":
    main()
```

### 5. Create Scripts for Git Integration

```bash
# scripts/update_sql_dumps.sh
#!/bin/bash
# Generate SQL dumps for all sector databases

echo "Generating SQL dumps for sector databases..."

for db in data/sectors/*.db; do
  if [ -f "$db" ]; then
    echo "Processing $db..."
    sqlite3 "$db" .dump > "${db%.db}.sql"
  fi
done

echo "Adding SQL dumps to git staging..."
git add data/sectors/*.sql

echo "SQL dumps updated successfully"
```

```bash
# scripts/setup_git_hooks.sh
#!/bin/bash
# Setup git hooks for the project

# Create pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

# Run the SQL dump update script
./scripts/update_sql_dumps.sh

# Exit with the status of the script
exit $?
EOF

# Make hook executable
chmod +x .git/hooks/pre-commit

echo "Git hooks installed successfully"
```

### 6. Create Query Interface Module

```python
# src/query_interface.py
import os
import sqlite3
import json

class StellarCartography:
    """Main query interface for the Stellar Cartography database"""
    
    def __init__(self, base_path="data"):
        """Initialize the interface"""
        self.base_path = base_path
        self.metadata_path = os.path.join(base_path, "galaxy_metadata.db")
        self.sectors_path = os.path.join(base_path, "sectors")
        self.connections = {}  # Cache for database connections
        
    def _get_metadata_connection(self):
        """Get connection to metadata database"""
        if "metadata" not in self.connections:
            self.connections["metadata"] = sqlite3.connect(self.metadata_path)
        return self.connections["metadata"]
    
    def _get_sector_connection(self, sector_code):
        """Get connection to a sector database"""
        if sector_code not in self.connections:
            db_path = os.path.join(self.sectors_path, f"sector_{sector_code.replace(':', '_')}.db")
            if os.path.exists(db_path):
                self.connections[sector_code] = sqlite3.connect(db_path)
            else:
                return None
        return self.connections[sector_code]
    
    def get_sectors(self):
        """Get list of all available sectors"""
        conn = self._get_metadata_connection()
        cursor = conn.cursor()
        cursor.execute("SELECT sector_code, star_count FROM sector_registry ORDER BY sector_code")
        return cursor.fetchall()
    
    def get_sectors_in_region(self, x_range, y_range, z_range):
        """Get sectors that overlap with the given spatial region"""
        conn = self._get_metadata_connection()
        cursor = conn.cursor()
        cursor.execute("""
        SELECT sector_code FROM sector_registry 
        WHERE 
            (x_min <= ? AND x_max >= ?) AND
            (y_min <= ? AND y_max >= ?) AND
            (z_min <= ? AND z_max >= ?)
        """, (x_range[1], x_range[0], y_range[1], y_range[0], z_range[1], z_range[0]))
        return [row[0] for row in cursor.fetchall()]
    
    def get_stars_in_sector(self, sector_code):
        """Get all stars in a specific sector"""
        conn = self._get_sector_connection(sector_code)
        if not conn:
            return []
        
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM stars")
        
        # Get column names
        columns = [description[0] for description in cursor.description]
        
        # Convert rows to dictionaries
        results = []
        for row in cursor.fetchall():
            star_dict = dict(zip(columns, row))
            
            # Parse JSON array
            if 'alternate_names' in star_dict and star_dict['alternate_names']:
                star_dict['alternate_names'] = json.loads(star_dict['alternate_names'])
            
            results.append(star_dict)
            
        return results
    
    def get_stars_in_region(self, x_range, y_range, z_range):
        """Get stars in the specified spatial region"""
        # Find relevant sectors
        sectors = self.get_sectors_in_region(x_range, y_range, z_range)
        
        # Query each sector
        all_stars = []
        for sector_code in sectors:
            conn = self._get_sector_connection(sector_code)
            if not conn:
                continue
                
            cursor = conn.cursor()
            cursor.execute("""
            SELECT * FROM stars 
            WHERE 
                x BETWEEN ? AND ? AND 
                y BETWEEN ? AND ? AND 
                z BETWEEN ? AND ?
            """, (x_range[0], x_range[1], y_range[0], y_range[1], z_range[0], z_range[1]))
            
            # Get column names
            columns = [description[0] for description in cursor.description]
            
            # Convert rows to dictionaries
            for row in cursor.fetchall():
                star_dict = dict(zip(columns, row))
                
                # Parse JSON array
                if 'alternate_names' in star_dict and star_dict['alternate_names']:
                    star_dict['alternate_names'] = json.loads(star_dict['alternate_names'])
                
                all_stars.append(star_dict)
        
        return all_stars
    
    def get_stars_by_name(self, name_pattern):
        """Search for stars by name pattern"""
        results = []
        
        # Get all sectors
        sectors = self.get_sectors()
        
        for sector_code, _ in sectors:
            conn = self._get_sector_connection(sector_code)
            if not conn:
                continue
                
            cursor = conn.cursor()
            cursor.execute("""
            SELECT * FROM stars 
            WHERE 
                name LIKE ? OR
                alternate_names LIKE ?
            """, (f"%{name_pattern}%", f"%{name_pattern}%"))
            
            # Get column names
            columns = [description[0] for description in cursor.description]
            
            # Convert rows to dictionaries
            for row in cursor.fetchall():
                star_dict = dict(zip(columns, row))
                
                # Parse JSON array
                if 'alternate_names' in star_dict and star_dict['alternate_names']:
                    star_dict['alternate_names'] = json.loads(star_dict['alternate_names'])
                
                results.append(star_dict)
        
        return results
    
    def get_nearest_stars(self, x, y, z, limit=10):
        """Find the nearest stars to a 3D point"""
        results = []
        
        # Get sectors that might contain nearby stars
        # Create a search cube around the point
        search_radius = 50  # light years
        sectors = self.get_sectors_in_region(
            (x - search_radius, x + search_radius),
            (y - search_radius, y + search_radius),
            (z - search_radius, z + search_radius)
        )
        
        # Collect stars and their distances
        stars_with_distance = []
        for sector_code in sectors:
            conn = self._get_sector_connection(sector_code)
            if not conn:
                continue
                
            cursor = conn.cursor()
            cursor.execute("SELECT * FROM stars")
            
            # Get column names
            columns = [description[0] for description in cursor.description]
            
            # Convert rows to dictionaries and calculate distance
            for row in cursor.fetchall():
                star_dict = dict(zip(columns, row))
                
                # Calculate distance to this point
                dist = ((star_dict['x'] - x) ** 2 + 
                        (star_dict['y'] - y) ** 2 + 
                        (star_dict['z'] - z) ** 2) ** 0.5
                
                # Parse JSON array
                if 'alternate_names' in star_dict and star_dict['alternate_names']:
                    star_dict['alternate_names'] = json.loads(star_dict['alternate_names'])
                
                stars_with_distance.append((dist, star_dict))
        
        # Sort by distance and take the top 'limit'
        stars_with_distance.sort(key=lambda x: x[0])
        results = [(dist, star) for dist, star in stars_with_distance[:limit]]
        
        return results
    
    def close(self):
        """Close all database connections"""
        for conn in self.connections.values():
            conn.close()
        self.connections = {}
```

## Example Usage

### Processing Raw Data

```python
# Example script to process raw data
from src.data_processing import main as process_data

# Initialize metadata database
from scripts.create_metadata_db import initialize_metadata_db
initialize_metadata_db()

# Process the star catalog
process_data()
```

### Querying the Database

```python
# Example script for querying
from src.query_interface import StellarCartography

# Initialize interface
cartography = StellarCartography()

# Find all sectors
sectors = cartography.get_sectors()
print(f"Found {len(sectors)} sectors")

# Get stars in a sector
stars = cartography.get_stars_in_sector("15:07:02")
print(f"Found {len(stars)} stars in sector 15:07:02")

# Find stars near Sol
sol_neighbors = cartography.get_nearest_stars(0, 0, 0, limit=10)
print("Nearest stars to Sol:")
for dist, star in sol_neighbors:
    print(f"  {star['name']} - {dist:.2f} ly")

# Search for stars by name
alpha_cen = cartography.get_stars_by_name("Centauri")
print(f"Found {len(alpha_cen)} stars matching 'Centauri'")

# Clean up when done
cartography.close()
```

## Implementation Roadmap

1. **Setup Phase**
   - Initialize directory structure
   - Set up git hooks and configuration
   - Create metadata database

2. **Data Processing Phase**
   - Implement coordinate conversion
   - Process raw star data into sector databases
   - Generate SQL dumps

3. **Query Implementation Phase**
   - Implement basic sector queries
   - Add spatial search capabilities
   - Create integration with FASA systems

4. **Optional Enhancements**
   - Add web API for data access
   - Implement visualization tools
   - Create export utilities for RPG use

## Best Practices for Implementation

1. **Run in separate stages**
   - Process small subsets of data during development
   - Validate outputs after each processing stage
   - Track performance metrics for large operations

2. **Use transactions**
   - Wrap database operations in transactions
   - Implement proper error handling and rollbacks
   - Add validation steps during imports

3. **Test with actual queries**
   - Validate the system with realistic query patterns
   - Ensure cross-sector queries work correctly
   - Verify performance with different dataset sizes

4. **Document as you go**
   - Add inline comments to explain complex operations
   - Update the implementation guide with lessons learned
   - Create a user guide for querying the database 