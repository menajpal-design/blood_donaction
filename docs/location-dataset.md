# Bangladesh Location Dataset Documentation

## Overview

The Location Dataset is a comprehensive, offline-queryable JSON file containing all Bangladesh administrative divisions organized hierarchically:
- **8 Divisions**
- **64 Districts**
- **473 Upazilas**
- **4,554 Unions/Pouroshava**

Each union is fully enriched with `divisionId`, `districtId`, and `upazilaId` for fast queries without database lookups.

## Files

| File | Purpose |
|------|---------|
| `data/location-dataset.json` | Complete dataset with three representations |
| `src/services/location-dataset.service.js` | Query service for in-memory lookups |
| `src/scripts/generate-location-dataset.js` | Script to regenerate the dataset |

## Dataset Structure

### 1. **Flat Array** (`flat` property)
Optimized for union-level queries with all parent references.

```json
{
  "flat": [
    {
      "id": 1,
      "name": "Sadarganj",
      "bnName": "সদরগঞ্জ",
      "code": "UNION-0001",
      "upazilaId": 1,
      "upazilaName": "Dhaka Sadar",
      "upazilaBnName": "ঢাকা সদর",
      "districtId": 1,
      "districtName": "Dhaka",
      "districtBnName": "ঢাকা",
      "divisionId": 1,
      "divisionName": "Dhaka",
      "divisionBnName": "ঢাকা"
    },
    // ... 4,553 more unions
  ]
}
```

**Use cases:**
- Searching for unions by name or code
- Building autocomplete suggestions
- Exporting union lists with full location context

### 2. **Hierarchical Structure** (`hierarchical` property)
Organized from divisions down to unions for UI navigation.

```json
{
  "hierarchical": [
    {
      "id": 1,
      "name": "Dhaka",
      "bnName": "ঢাকা",
      "code": "DIV-0001",
      "districts": [
        {
          "id": 1,
          "name": "Dhaka",
          "bnName": "ঢাকা",
          "code": "DIS-0001",
          "divisionId": 1,
          "upazilas": [
            {
              "id": 1,
              "name": "Dhaka Sadar",
              "bnName": "ঢাকা সদর",
              "code": "UPAZILA-0001",
              "districtId": 1,
              "divisionId": 1,
              "unions": [
                {
                  "id": 1,
                  "name": "Sadarganj",
                  "bnName": "সদরগঞ্জ",
                  "code": "UNION-0001",
                  "upazilaId": 1,
                  "districtId": 1,
                  "divisionId": 1
                }
                // ... more unions
              ]
            }
            // ... more upazilas
          ]
        }
        // ... more districts
      ]
    }
    // ... more divisions
  ]
}
```

**Use cases:**
- Rendering cascading dropdown menus
- Frontend tree structures
- Hierarchical navigation UI

### 3. **Optimized Indexes** (`indexes` property)
Four maps (object keys) for O(1) lookups by location ID.

```json
{
  "indexes": {
    "divisions": { "1": {...}, "2": {...}, ... },
    "districts": { "1": {...}, "2": {...}, ... },
    "upazilas": { "1": {...}, "2": {...}, ... },
    "unions": { "1": {...}, "2": {...}, ... }
  }
}
```

**Use cases:**
- Fast ID → location lookups
- Validating hierarchies
- Reference resolution

## Query Service

### Import
```javascript
import { locationDatasetService } from '../services/location-dataset.service.js';
```

### Common Operations

#### 1. Find a Location by ID (O(1))
```javascript
// Get a union
const union = locationDatasetService.findUnion(1);
// { id: 1, name: "Sadarganj", divisionId: 1, districtId: 1, upazilaId: 1, ... }

// Get an upazila
const upazila = locationDatasetService.findUpazila(1);

// Get a district
const district = locationDatasetService.findDistrict(1);

// Get a division
const division = locationDatasetService.findDivision(1);
```

#### 2. Get Cascading Data
```javascript
// Get unions in an upazila
const unions = locationDatasetService.getUnionsByUpazila(1);

// Get upazilas in a district
const upazilas = locationDatasetService.getUpazilasByDistrict(1);

// Get districts in a division
const districts = locationDatasetService.getDistrictsByDivision(1);
```

#### 3. Search by Name
```javascript
// Search unions (case-insensitive, partial match)
const results = locationDatasetService.searchUnions('Sadarganj', 50);
// Returns up to 50 matching unions
```

#### 4. Get Hierarchical Path
```javascript
// Trace from union up to division
const path = locationDatasetService.getLocationPath(1);
// {
//   union: {...},
//   upazila: {...},
//   district: {...},
//   division: {...},
//   path: "Dhaka > Dhaka > Dhaka Sadar > Sadarganj",
//   pathBn: "ঢাকা > ঢাকা > ঢাকা সদর > সদরগঞ্জ"
// }
```

#### 5. Validate Location Hierarchy
```javascript
// Verify union belongs to this hierarchy
const isValid = locationDatasetService.validateHierarchy(
  unionId,
  upazilaId,
  districtId,
  divisionId
);
```

#### 6. Get All Locations at a Level
```javascript
// All divisions
const divisions = locationDatasetService.getAllDivisions();

// All unions
const unions = locationDatasetService.getAllUnions();

// Complete hierarchy
const hierarchy = locationDatasetService.getFullHierarchy();
```

## Performance Characteristics

| Operation | Time Complexity | Notes |
|-----------|-----------------|-------|
| `findUnion()` | O(1) | Index lookup |
| `getUnionsByUpazila()` | O(n) | n = unions in upazila (~10-20) |
| `searchUnions()` | O(m) | m = total unions (4,554) |
| `getLocationPath()` | O(1) | Two index lookups |
| `validateHierarchy()` | O(1) | Index lookup + comparison |

**Dataset Size:** ~4.6 MB (JSON) → ~1 MB (gzipped)

## Use Cases

### 1. **Client-Side Autocomplete**
```javascript
// Download once on app load
const allUnions = locationDatasetService.getAllUnions();

// Search as user types
const filtered = allUnions.filter(union => 
  union.name.toLowerCase().includes(searchTerm)
);
```

### 2. **Cascading Form Fields** (React)
```jsx
export const LocationSelector = () => {
  const [district, setDistrict] = useState('');
  
  const upazilas = locationDatasetService.getUpazilasByDistrict(district);
  
  const handleDistrictChange = (districtId) => {
    setDistrict(districtId);
    // No API call needed - instant update!
  };
  
  return (
    <>
      <select onChange={(e) => handleDistrictChange(e.target.value)}>
        {/* districts */}
      </select>
      <select>
        {upazilas.map(u => <option key={u.id}>{u.name}</option>)}
      </select>
    </>
  );
};
```

### 3. **Location Validation** (Backend)
```javascript
// When user submits form
const isValid = locationDatasetService.validateHierarchy(
  req.body.unionId,
  req.body.upazilaId,
  req.body.districtId,
  req.body.divisionId
);

if (!isValid) {
  throw new ApiError(400, 'Invalid location hierarchy');
}
```

### 4. **Display Full Address Path**
```javascript
const locPath = locationDatasetService.getLocationPath(unionId);
console.log(locPath.pathBn);
// Output: "ঢাকা > ঢাকা > ঢাকা সদর > সদরগঞ্জ"
```

## Regenerating the Dataset

If the `bd-address-pro` package is updated with new location data:

```bash
# Server directory
npm run generate:location-data

# Or directly
node src/scripts/generate-location-dataset.js
```

This will:
1. Fetch fresh data from `bd-address-pro`
2. Validate structure and counts
3. Generate all three representations
4. Save to `data/location-dataset.json`

## Integration Points

### 1. **Frontend (React)**
```javascript
// Import client-side
import { locationDatasetService } from '../services/location-dataset.service.js';

// Or embed as JSON
import locationData from '../data/location-dataset.json';
```

### 2. **Backend (Express)**
```javascript
// Server-side queries
import { locationDatasetService } from '../services/location-dataset.service.js';

app.get('/api/search/unions', (req, res) => {
  const results = locationDatasetService.searchUnions(req.query.q);
  res.json(results);
});
```

### 3. **Database Population**
Keep using the database for authoritative records, but use the dataset service for:
- Quick validation checks
- Search suggestions
- Reference lookups

## Best Practices

✅ **Do:**
- Use dataset service for read-only queries
- Cache the dataset once on app load
- Use for client-side form validation
- Validate submitted hierarchies server-side

❌ **Don't:**
- Modify the dataset at runtime (it will reload from disk)
- Use for user-generated location data
- Replace database with dataset service for critical operations

## Troubleshooting

### "Location dataset not found"
```bash
# Regenerate the dataset
npm run generate:location-data
```

### Dataset vs Database Mismatch
If database locations differ from dataset:
```bash
# Regenerate dataset to match db
npm run generate:location-data

# Or reseed database
npm run seed:locations
```

## API Reference (Location Dataset Service)

```typescript
// Lookups
findUnion(unionId: number): LocationEntity | null
findUpazila(upazilaId: number): LocationEntity | null
findDistrict(districtId: number): LocationEntity | null
findDivision(divisionId: number): LocationEntity | null

// Get related
getUnionsByUpazila(upazilaId: number): LocationEntity[]
getUpazilasByDistrict(districtId: number): LocationEntity[]
getDistrictsByDivision(divisionId: number): LocationEntity[]

// Search
searchUnions(term: string, limit?: number): LocationEntity[]

// Hierarchy
getHierarchyByDivision(divisionId: number): HierarchyNode | null
getFullHierarchy(): HierarchyNode[]
getLocationPath(unionId: number): LocationPath | null

// Validate
validateHierarchy(unionId, upazilaId, districtId, divisionId): boolean

// Data
getAllDivisions(): LocationEntity[]
getAllUnions(): LocationEntity[]
getDataset(): Dataset
getMetadata(): Metadata
getStats(): Stats
```

## License

This dataset is derived from `bd-address-pro` package following Bangladesh administrative boundaries.
