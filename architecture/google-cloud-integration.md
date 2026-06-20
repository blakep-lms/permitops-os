# Google Cloud Integration — Proximity Exhibits

PermitOps OS generates court-ready "Points of Consideration" maps — branded radius exhibits required for California ABC (Alcohol Beverage Control) and TTB (Alcohol and Tobacco Tax and Trade Bureau) applications.

## The Regulatory Requirement

California ABC alcohol permit applications require proof that the proposed premises are not within **500-600 feet** of "sensitive uses":

- Schools and educational institutions
- Churches and religious institutions
- Public playgrounds and parks
- Residential dwellings (in certain zoning configurations)
- Hospitals and healthcare facilities

The applicant must submit a map showing these proximity points relative to the proposed location. This map is a legal exhibit — it must be accurate, current, and defensible.

## The Problem

Historically, permit consultants create these maps manually:
1. Look up the address on Google Maps
2. Manually search for nearby sensitive uses
3. Measure distances with the ruler tool
4. Screenshot, annotate, and format
5. Convert to PDF with branding

This takes **30-60 minutes per location**, is error-prone, and varies in quality depending on who does it. For a consultant managing hundreds of cases, it's a significant time sink.

## The Solution: Automated Proximity Exhibits

PermitOps OS automates the entire pipeline:

```
┌──────────────────────────────────────────────────────────┐
│                  PROXIMITY EXHIBIT PIPELINE                │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  1. ADDRESS INPUT                                          │
│     └─ Case premises address                               │
│                                                            │
│  2. GOOGLE GEOCODING API                                   │
│     └─ Precise lat/lng for the premises                    │
│                                                            │
│  3. GOOGLE ADDRESS VALIDATION API                          │
│     └─ Verify the address is real + component-level parse  │
│                                                            │
│  4. SENSITIVE-USE QUERY                                    │
│     ├─ OSM/Overpass API                                    │
│     │  Query: schools, churches, parks, playgrounds        │
│     │  within configurable radius (500ft / 600ft)          │
│     ├─ U.S. Census Geocoder                                │
│     │  Corroboration for residential classification        │
│     └─ Results: list of consideration points with distance │
│                                                            │
│  5. MAP GENERATION                                         │
│     ├─ Google Static Maps API                              │
│     │  Branded map image with markers                      │
│     └─ Overlay: radius circle, labeled points, legend      │
│                                                            │
│  6. EXHIBIT ASSEMBLY                                       │
│     ├─ HTML template (branded, professional)               │
│     ├─ Data table: each consideration point + distance     │
│     ├─ Certifications and disclaimers                      │
│     └─ Convert: HTML → PDF → PNG (for embedding)           │
│                                                            │
│  7. APPROVAL GATE                                          │
│     └─ the permit consultant reviews before the exhibit enters the       │
│        packet as a Final Deliverable                       │
│                                                            │
└──────────────────────────────────────────────────────────┘
```

## Google Cloud APIs Used

| API | Purpose | Why Google |
|---|---|---|
| **Geocoding API** | Convert address → precise coordinates | Best-in-class accuracy for California addresses |
| **Address Validation API** | Verify address exists + component validation | Catches typos, suite/unit issues, and wrong cities |
| **Static Maps API** | Generate branded map images | Custom styling, markers, and paths |

## Dedication: Dedicated Hermes Skill

The proximity exhibit system runs as a dedicated Hermes Agent skill (`pnm-proximity-exhibits` v1.0.0) on Pearl's hardware. The skill:

- Accepts a premises address and case context
- Runs the full pipeline above
- Produces a branded HTML exhibit
- Converts to PDF for filing
- Preserves the approval gate (the permit consultant reviews before it enters the packet)

## Technical Details

### Radius Configuration

```python
# Default radii by permit type
RADII = {
    "abc_on_sale": 600,    # feet — ABC on-sale general
    "abc_off_sale": 600,   # feet — ABC off-sale general
    "abc_beer_wine": 500,  # feet — beer/wine only (some cities)
    "ttb_federal": 300,    # feet — TTB federal permit
}
```

### Sensitive-Use Categories (OSM Tags)

```python
SENSITIVE_USES = {
    "school": ["amenity=school", "amenity=college", "amenity=university"],
    "church": ["amenity=place_of_worship"],
    "park": ["leisure=park", "leisure=playground", "leisure=recreation_ground"],
    "hospital": ["amenity=hospital", "amenity=clinic"],
    "residential": ["building=residential", "building=apartments"],
}
```

### Output Format

Each exhibit includes:

1. **Cover header** — PNM branding, case reference, address
2. **Map image** — Google Static Maps with markers + radius circle
3. **Data table** — every consideration point with:
   - Name/type (e.g., "Lincoln Elementary School")
   - Distance from premises (in feet)
   - Within radius? (Yes/No)
4. **Certifications** — methodology, data sources, generation date
5. **Page footer** — exhibit number, case number, prepared by

### First Successful Exhibit

The first operational exhibit was generated for a Huntington Beach location (18971 Beach Blvd), producing a complete branded HTML → PDF → PNG exhibit with geocoded consideration points. This proved the full pipeline end-to-end.

## Why This Is a Differentiator

| Manual Process | PermitOps OS |
|---|---|
| 30-60 minutes per location | ~2 minutes automated |
| Quality varies by preparer | Consistent, branded, defensible |
| Manual distance measurement | API-calculated, verifiable |
| Separate document formatting | HTML→PDF pipeline, filing-ready |
| No audit trail | Full provenance: when, what sources, who approved |

For a consultant filing 50+ ABC applications per year, this saves **25-50 hours of manual work** annually while improving quality and consistency.

## Future Enhancements

- **Hearing exhibit mode** — maps optimized for planning commission presentations
- **Batch processing** — generate exhibits for multiple premises in one run
- **Historical comparison** — overlay previous exhibit for the same location
- **Henry agent integration** — GM# (store ID) geocoding and cross-referencing across multi-site operators

## Related

- [Approval Gates](approval-gates.md) — exhibits require the permit consultant approval before entering packets
- [Workflows](../workflows/README.md) — ABC-TTB Proximity Exhibit Workflow
- [Data Model](data-model.md) — `source_evidence` records for each exhibit
