# Star Catalog CSV Field Reference

This document describes each field in the star catalog CSV files used in the Stellar Cartography Project.

## Header Structure

Each constellation CSV file and the main star catalog follow this header structure:

```
USID,Constellation,Name,B,B->F->VAR->G,F,G.,Var,HD,HIP,RA,Dec,"vis. mag.","abs. mag.",Dist. (ly),Sp. class,Notes,RA angle,Dec Angle,Latin Constellation,Full Designation
```

## Field Descriptions

### Identification Fields

- **USID** - Unique Star ID
  - Sequential numeric identifier for each star system
  - Primary key for reference across the entire catalog
  - Values begin at 1 and are continuous across constellation files

- **Constellation** - Constellation Name
  - The name of the constellation where the star is located (e.g., "Andromeda", "Aquarius")
  - Used for traditional astronomical organization

- **Name** - Common Star Name
  - The common or traditional name of the star (e.g., "α And", "Alpheratz")
  - May include Greek letter designations for brighter stars

- **Full Designation** - Complete Star Designation
  - The full designation including Greek letter or other identifier and constellation

### Bayer/Flamsteed/Variable Designations

- **B** - [Bayer Designation](https://en.wikipedia.org/wiki/Bayer_designation)
  - Greek letter designation (e.g., "α", "β")
  - Used for identifying brighter stars within a constellation
  - Created by Johann Bayer in 1603 in his star atlas _Uranometria_
  - Generally assigned in order of brightness within each constellation

- **B->F->VAR->G** - Combined Designation Information
  - Extra information related to Bayer, Flamsteed, Variable, or General Catalog designation
  - Indicates alternative designation systems applicable to the star

- **F** - [Flamsteed Number](https://en.wikipedia.org/wiki/Flamsteed_designation)
  - Numeric designation within a constellation
  - Historical catalog identifier created by John Flamsteed
  - Stars numbered by their right ascension within each constellation

- **G.** - [Gould Designation](https://en.wikipedia.org/wiki/Gould_designation)
  - Alternative catalog designation
  - Created by Benjamin Apthorp Gould in his _Uranometria Argentina_ (1879)
  - Used particularly for southern hemisphere stars

- **Var** - [Variable Star Designation](https://en.wikipedia.org/wiki/Variable-star_designation)
  - Identifier for variable stars (e.g., "R", "S", "V428")
  - Standard system for naming stars that change in brightness
  - Uses capital letters and, when those are exhausted, double-letter designations

### Catalog Identifiers

- **HD** - [Henry Draper Catalog Number](https://en.wikipedia.org/wiki/Henry_Draper_Catalogue)
  - Reference number from the Henry Draper Catalog
  - Standard astronomical reference catalog published between 1918 and 1924
  - Covers over 225,000 stars and provides spectral classifications

- **HIP** - [Hipparcos Catalog Number](https://en.wikipedia.org/wiki/Hipparcos_Catalogue)
  - Reference number from the Hipparcos satellite mission catalog
  - Modern high-precision astrometric catalog released in 1997
  - Contains precise positions, motions, and distances for over 118,000 stars

### Positional and Distance Data

- **RA** - Right Ascension
  - Celestial longitude in hours, minutes, seconds format (e.g., "00h 08m 23.17s")
  - One of the two coordinates used to locate a star on the celestial sphere

- **Dec** - Declination
  - Celestial latitude in degrees, minutes, seconds format (e.g., "+29° 05′ 27.0″")
  - The second coordinate used to locate a star on the celestial sphere

- **RA angle** - Right Ascension in Decimal Degrees
  - RA converted to decimal degrees for easier computation (0-360°)

- **Dec Angle** - Declination in Decimal Degrees
  - Dec converted to decimal degrees for easier computation (-90° to +90°)

- **Dist. (ly)** - Distance in Light-Years
  - The distance from Earth to the star measured in light-years
  - Critical for 3D spatial calculations

### Physical Characteristics

- **vis. mag.** - Visual Magnitude
  - Apparent brightness as seen from Earth
  - Lower numbers indicate brighter stars (e.g., Sirius is -1.46)

- **abs. mag.** - Absolute Magnitude
  - Intrinsic brightness (how bright the star would appear at a standard distance of 10 parsecs)
  - Measure of the star's actual luminosity

- **Sp. class** - Spectral Classification
  - Classification based on spectral characteristics (e.g., "B9p", "M0IIIvar")
  - Indicates temperature, luminosity, and other properties
  - Format typically includes:
    - Temperature class (O, B, A, F, G, K, M from hottest to coolest)
    - Luminosity class (I-V, with I being supergiants and V being main sequence)
    - Special features (e.g., "p" for peculiar, "var" for variable)

### Additional Information

- **Notes** - Additional Information
  - Descriptive text with additional details about the star
  - May include variability information, multiplicity, special features
  - Sometimes includes a reference to source information

- **Latin Constellation** - Latin Genitive Form
  - The constellation name in Latin genitive form (e.g., "Andromedae", "Aquarii")
  - Used in formal star designations

## Usage Examples

### Locating a Star in 3D Space

To position a star in 3D space, use:
1. Convert RA and Dec to radians or use the pre-calculated RA angle and Dec Angle fields
2. Use the distance in light-years
3. Apply spherical-to-Cartesian conversion formulas

### Identifying a Star

Stars can be referenced by:
1. USID (e.g., "USID 42") - Most precise method
2. Catalog number (e.g., "HD 6114" or "HIP 4911")
3. Traditional name (e.g., "Alpheratz" or "α Andromedae") 