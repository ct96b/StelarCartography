# FASA Star Trek Systems CSV Field Reference

This document describes the fields in the FASA Star Trek systems CSV file used in the Stellar Cartography Project.

## Header Structure

The FASA systems CSV file has the following header structure:

```
System,NS,EW,Status
```

## Field Descriptions

### Identification Field

- **System** - Star System Name
  - The name of the fictional star system from the FASA Star Trek universe
  - May include special prefixes (Alpha, Beta, Delta, etc.) or suffixes (numbers)
  - May include special characters such as apostrophes and hyphens
  - Some entries may begin with a # symbol, indicating special notation or provisional designations

### Positional Data

- **NS** - North/South Coordinate
  - Represents the North-South position in the FASA coordinate system
  - Positive values generally indicate positions "north" of the galactic plane
  - Negative values generally indicate positions "south" of the galactic plane
  - Values typically range from approximately -12 to +7
  - Some entries may be blank if coordinates are unknown or undefined

- **EW** - East/West Coordinate
  - Represents the East-West position in the FASA coordinate system
  - Positive values generally indicate positions "east" in the galaxy
  - Negative values generally indicate positions "west" in the galaxy
  - Values typically range from approximately -9 to +8
  - Some entries may be blank if coordinates are unknown or undefined

### Political Information

- **Status** - Political Affiliation
  - Indicates which political entity controls or is associated with the system
  - Common values include:
    - **UFP Member** - Full member of the United Federation of Planets
    - **UFP Associate** - Associate member or protectorate of the Federation
    - **UFP Colony** - Federation colony world
    - **Klingon Empire** - System controlled by the Klingon Empire
    - **Romulan Star Empire** - System controlled by the Romulan Star Empire
    - **Orion** - System associated with the Orion pirates/syndicates
    - **Indep** - Independent system not affiliated with major powers
    - **AoOW** - Associated Outworlds (coalition of independent worlds)
    - **PDir** - Prime Directive applies (pre-warp civilization)
    - **Quar** - Quarantined system
    - **MCA** - Mutual Cooperation Area (shared jurisdiction)
    - **OFMA** - Orion Free Merchant Association
    - **IKS** - Imperial Klingon States

## Usage Notes

### Mapping to Real Star Systems

When mapping FASA systems to real star systems in our catalog:

1. The NS/EW coordinates provide a relative positioning guide but are not directly convertible to astronomical coordinates
2. Political affiliations can be used to ensure logical clustering of faction territories
3. Named systems that correspond to real stars (e.g., "Alpha Tauri") should be mapped to their actual counterparts where possible
4. Systems without coordinates should be assigned positions based on narrative context and political boundaries

### Coordinate System Conversion

The FASA coordinate system is a simplified 2D grid created for gameplay purposes and doesn't directly correspond to actual astronomical coordinates. The process of mapping involves:

1. Creating a correlation between the FASA NS/EW grid and our 3D Cartesian coordinate system
2. Adjusting for the fact that FASA coordinates compress vast astronomical distances
3. Maintaining relative positioning between politically affiliated systems
4. Adding the Z-axis component that is not represented in the FASA system

## Special System Notes

- Systems with empty coordinate fields may be extragalactic or have deliberately obscured locations in the source material
- Some systems like "Bon" or "Corill" appear in the FASA materials but lack detailed information
- Systems with "###" after their names may be duplicate designations or alternate versions of similarly named systems 