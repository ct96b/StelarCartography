# Star Trek System Mapping Algorithm

This document outlines the algorithm used to map fictional Star Trek systems from the FASA Star Trek universe to real astronomical data in the Stelar Cartography Project.

## Mapping Challenge

The core challenge is to assign approximately 700 fictional Star Trek systems to a subset of real astronomical systems from our 12,000-star catalog. The mapping must:

1. Honor canonical associations where they exist
2. Create plausible mappings for fictional systems
3. Maintain canon-consistent political groupings
4. Respect astronomical reality where possible

## Mapping Categories

The algorithm divides systems into three categories:

### 1. Canon-Confirmed Systems

Some Star Trek systems have established real-world counterparts in canon sources or authoritative references:

| Star Trek System | Real Star System   | Source            |
|------------------|--------------------|--------------------|
| Sol (Earth)      | Sun                | Canon              |
| Vulcan           | 40 Eridani A       | Canon (confirmed)  |
| Alpha Centauri   | Alpha Centauri     | Canon              |
| Wolf 359         | Wolf 359           | Canon              |
| Sirius           | Sirius             | Canon              |

These systems are directly mapped based on name or catalog designations.

### 2. Implied Systems

Some Star Trek systems strongly imply a real star but aren't definitively confirmed:

| Star Trek System | Likely Real Star    | Reasoning                          |
|------------------|--------------------|-----------------------------------|
| Andoria          | Epsilon Indi        | Distance from Earth matches canon  |
| Rigel            | Beta Orionis        | Traditional name implies connection|
| Denobula         | Chi¹ Orionis        | Star characteristics match lore    |

These systems are mapped with high confidence but might be overridden with manual adjustments.

### 3. Fictional Systems

Most Star Trek systems have no clear real-world counterpart (e.g., Quo'noS, Romulus, Risa) and must be assigned based on criteria:

- Distance constraints from canon
- Spectral class appropriate for habitability
- Political grouping considerations
- Spatial relationship to known systems

## Mapping Algorithm Steps

The algorithm proceeds in this order:

### Step 1: Direct Mapping

For canon-confirmed and implied systems:

```pseudocode
function MapKnownSystems(trek_systems, star_catalog):
    for each system in trek_systems:
        if system.has_real_counterpart():
            matches = star_catalog.find_by_name(system.real_counterpart)
            if matches:
                system.assign_to(matches[0])
                system.mapping_confidence = "High"
```

### Step 2: Distance-Constrained Mapping

For systems with canonical distances but no specific real counterpart:

```pseudocode
function MapDistanceConstrainedSystems(trek_systems, star_catalog):
    for each system in trek_systems:
        if system.has_distance_constraint() and not system.is_mapped():
            distance = system.distance_from_sol
            tolerance = min(distance * 0.2, 5.0)  # 20% or 5 ly tolerance
            
            candidate_stars = star_catalog.find_by_distance(
                distance - tolerance, 
                distance + tolerance
            )
            
            # Filter by habitability
            candidate_stars = filter_by_spectral_class(candidate_stars, ['F', 'G', 'K'])
            
            if candidate_stars:
                # Choose best candidate (e.g., most similar to Sol)
                system.assign_to(select_best_candidate(candidate_stars))
                system.mapping_confidence = "Medium"
```

### Step 3: Political Clustering

For remaining systems, group by political entity:

```pseudocode
function MapPoliticalSystems(trek_systems, star_catalog):
    # Group unmapped systems by faction
    faction_systems = group_by_faction(
        [s for s in trek_systems if not s.is_mapped()]
    )
    
    for faction, systems in faction_systems.items():
        # Find anchor points (already mapped systems of this faction)
        anchors = [s for s in trek_systems if s.faction == faction and s.is_mapped()]
        
        if anchors:
            # Create a spatial cluster around the anchors
            center = average_position(anchors)
            radius = estimate_faction_radius(faction)
            
            # Find suitable stars in this region
            candidates = star_catalog.find_in_sphere(center, radius)
            candidates = filter_by_spectral_class(candidates, ['F', 'G', 'K'])
            
            # Assign unmapped systems to candidates
            assign_systems_to_candidates(systems, candidates)
```

### Step 4: Spatial Distribution

For any remaining unmapped systems:

```pseudocode
function MapRemainingSystemsSpatially(trek_systems, star_catalog):
    # Create KDTree for efficient spatial queries
    from scipy.spatial import KDTree
    mapped_positions = [s.position for s in trek_systems if s.is_mapped()]
    kdtree = KDTree(mapped_positions)
    
    for system in trek_systems:
        if not system.is_mapped():
            # Estimate position based on FASA map coordinates
            estimated_position = convert_fasa_to_3d(system.fasa_coordinates)
            
            # Find nearest mapped system as reference
            distance, index = kdtree.query(estimated_position)
            reference_system = trek_systems[index]
            
            # Find real stars in estimated direction and distance
            candidates = find_stars_in_direction(
                reference_system.position,
                estimated_position - reference_system.position,
                tolerance=20.0  # degrees
            )
            
            if candidates:
                system.assign_to(select_best_candidate(candidates))
                system.mapping_confidence = "Low"
```

### Step 5: Manual Overrides

The algorithm provides a JSON file of manual overrides for specific systems:

```json
{
  "manual_overrides": [
    {
      "trek_system": "Quo'noS",
      "star_id": "HIP 88601",
      "reasoning": "K-type star with appropriate distance, manually selected to be more isolated from Federation space"
    },
    {
      "trek_system": "Romulus",
      "star_id": "HIP 112122",
      "reasoning": "Located in an isolated region appropriate for the Romulan Star Empire"
    }
  ]
}
```

These overrides are applied last and take precedence over algorithmic assignments.

## Selection Criteria for Fictional Systems

When choosing between candidate stars for a fictional system, the algorithm considers:

### Habitability Score

Based on:
- Spectral class (F5-K2 stars preferred)
- Multiple star systems (stable orbits less likely)
- Age of star (old enough for life to evolve)

```pseudocode
function calculate_habitability_score(star):
    score = 0
    
    # Spectral class (higher is better)
    if star.spectral_class.startswith('G'):
        score += 5
    elif star.spectral_class.startswith('K') and int(star.spectral_class[1]) < 5:
        score += 4
    elif star.spectral_class.startswith('F') and int(star.spectral_class[1]) > 5:
        score += 3
    
    # Multiple star systems (lower is better)
    if 'binary' in star.notes.lower() or 'multiple' in star.notes.lower():
        score -= 2
    
    return score
```

### Political Consistency

Based on:
- Proximity to other systems of same faction
- Appropriate borders with enemy factions
- Strategic value based on position

```pseudocode
function calculate_political_score(star, trek_system, mapped_systems):
    score = 0
    
    # Distance to other systems of same faction
    same_faction_systems = [s for s in mapped_systems if s.faction == trek_system.faction]
    avg_distance = average_distance(star.position, [s.position for s in same_faction_systems])
    
    # Closer is better (within reason)
    if 5 < avg_distance < 30:
        score += 3
    elif avg_distance <= 5 or avg_distance < 50:
        score += 1
    
    # Border systems get bonus for being near other factions
    if trek_system.is_border_system():
        enemy_systems = [s for s in mapped_systems if s.faction in trek_system.enemy_factions()]
        min_enemy_distance = min_distance(star.position, [s.position for s in enemy_systems])
        
        # Close but not too close
        if 5 < min_enemy_distance < 15:
            score += 2
    
    return score
```

### Canon Consistency

Based on:
- Match to canonical distance (if known)
- Consistency with established travel times
- Known directional references

## Evaluation Metrics

The mapping is evaluated based on:

1. **Canonical Accuracy**: Percentage of canon-compliant mappings
2. **Habitability Plausibility**: Percentage of systems mapped to stars of appropriate type
3. **Political Coherence**: Average distance between systems of the same faction
4. **Spatial Distribution**: How well the map preserves the relative positions from the FASA map

## Edge Cases and Solutions

### Handling Sparse Regions

Some Star Trek regions are densely populated but correspond to sparse areas in real space:

```pseudocode
function handle_sparse_region(region, trek_systems, star_catalog):
    # If not enough suitable stars in a region:
    if count_suitable_stars(region) < count_systems_in_region(trek_systems, region):
        # 1. Expand search radius
        expanded_region = expand_region(region, factor=1.5)
        candidates = find_stars_in_region(expanded_region)
        
        # 2. Relax spectral class restrictions
        if len(candidates) < required_count:
            candidates = find_stars_in_region(expanded_region, allow_extended_spectral=True)
        
        # 3. If still insufficient, mark some systems as "unconfirmed"
        if len(candidates) < required_count:
            mark_excess_systems_as_unconfirmed(trek_systems, region, len(candidates))
```

### Handling Dense Regions

Some areas have too many suitable stars for the Trek systems present:

```pseudocode
function handle_dense_region(region, trek_systems, star_catalog):
    candidates = find_stars_in_region(region)
    
    if len(candidates) > 2 * count_systems_in_region(trek_systems, region):
        # Filter to most appropriate stars
        candidates = filter_top_candidates(
            candidates, 
            count_systems_in_region(trek_systems, region) * 1.5,
            criteria=['habitability', 'brightness', 'isolation']
        )
```

## Implementation Details

The mapping algorithm is implemented in `src/system_mapping.py` and uses:

1. **KDTree** for efficient spatial queries
2. **Hierarchical Clustering** to identify natural star groupings
3. **A* Pathfinding** to validate travel routes between systems

## Future Enhancements

Potential improvements to the mapping algorithm:

1. **Machine Learning Approach**: Train a model on canon systems to identify suitable candidates for fictional systems
2. **User Feedback Loop**: Allow users to suggest alternative mappings and incorporate feedback
3. **Dynamic Recalculation**: Update mappings as new astronomical data becomes available
4. **Interactive Tuning**: Provide tools for manually adjusting system positions while maintaining constraints 