# Location Cascading Dropdown System

## Overview

The registration form now includes a cascading dropdown system for selecting Bangladesh locations (Division → District → Upazila → Union/Pouroshava). Each dropdown is disabled until the previous level is selected, and data is fetched dynamically from the backend APIs.

## Architecture

### Components

#### 1. **useLocationCascade Hook** (`client/src/features/auth/hooks/useLocationCascade.js`)
A custom React hook that manages the cascading dropdown state and data fetching logic.

**Features:**
- Loads all divisions on component mount
- Automatically fetches districts when a division is selected
- Automatically fetches upazilas when a district is selected
- Automatically fetches unions when an upazila is selected
- Resets dependent dropdowns when parent selection changes
- Manages loading states for each level
- Handles errors gracefully

**State Management:**
```javascript
{
  divisions,       // Array of division objects
  districts,       // Array of district objects (filtered by division)
  upazilas,        // Array of upazila objects (filtered by district)
  unions,          // Array of union objects (filtered by upazila)
  
  selectedDivision,   // Currently selected division ID
  selectedDistrict,   // Currently selected district ID
  selectedUpazilaId,  // Currently selected upazila ID
  selectedUnion,      // Currently selected union ID
  
  isLoadingDivisions,  // Boolean: divisions loading state
  isLoadingDistricts,  // Boolean: districts loading state
  isLoadingUpazilas,   // Boolean: upazilas loading state
  isLoadingUnions,     // Boolean: unions loading state
  
  error            // Error message if any API call fails
}
```

#### 2. **LocationCascadeSelect Component** (`client/src/features/auth/components/LocationCascadeSelect.jsx`)
A React component that renders the four cascading dropdowns with proper disable states and loading indicators.

**Features:**
- Displays four select dropdowns in a 2x2 grid
- Disables each dropdown until previous is selected
- Shows loading indicators while fetching data
- Renders placeholder text when disabled (e.g., "Select Division first")
- Calls parent callback on each selection change
- Displays error messages if loading fails

**Props:**
```javascript
onLocationChange: (locationData) => void
  // Called when any dropdown value changes
  // Passes: { divisionId, districtId, upazilaId, unionId }
```

#### 3. **RegisterPage Component** (`client/src/features/auth/pages/RegisterPage.jsx`)
Updated registration form that integrates the LocationCascadeSelect component.

**Integration:**
```javascript
const [formData, setFormData] = useState({
  // ... other fields
  divisionId: '',
  districtId: '',
  upazilaId: '',
  unionId: '',
  // ... other fields
});

const handleLocationChange = (locationData) => {
  setFormData((previous) => ({
    ...previous,
    ...locationData,
  }));
};

// In JSX:
<LocationCascadeSelect onLocationChange={handleLocationChange} />
```

## API Integration

The component uses the following backend endpoints:

1. **GET /api/v1/locations/divisions**
   - Fetches all divisions on mount
   - Returns: `{ success: true, data: [{ id, name, bnName, code, externalId }, ...] }`

2. **GET /api/v1/locations/divisions/:divisionId/districts**
   - Fetches districts for selected division
   - Returns: `{ success: true, data: [{ id, name, bnName, code, divisionId, externalId }, ...] }`

3. **GET /api/v1/locations/districts/:districtId/upazilas**
   - Fetches upazilas for selected district
   - Returns: `{ success: true, data: [{ id, name, bnName, code, districtId, divisionId, externalId }, ...] }`

4. **GET /api/v1/locations/upazilas/:upazilaId/unions**
   - Fetches unions for selected upazila
   - Returns: `{ success: true, data: [{ id, name, bnName, code, upazilaId, districtId, divisionId, externalId }, ...] }`

## User Flow

1. **Page Load**
   - All divisions are fetched and displayed in the first dropdown
   - Districts, Upazilas, and Unions dropdowns are disabled

2. **User selects a Division**
   - Districts are fetched and the dropdown is enabled
   - Upazilas and Unions dropdowns are reset and disabled

3. **User selects a District**
   - Upazilas are fetched and the dropdown is enabled
   - Unions dropdown is reset and disabled

4. **User selects an Upazila**
   - Unions are fetched and the dropdown is enabled

5. **User selects a Union**
   - Form is ready for submission with all location data populated

## Styling

CSS classes for the cascading dropdown system:

- `.location-cascade-container` - Main container (2-column grid on desktop, 1-column on mobile)
- `.location-select-wrapper` - Individual dropdown wrapper
- `.location-select-wrapper label` - Dropdown label with loading indicator
- `.location-select-wrapper select` - Dropdown element with states
- `.location-select-wrapper select:disabled` - Disabled state styling
- `.loading-indicator` - Animated loading text with pulsing dot
- `.error-message` - Error message container

### Responsive Design

- **Desktop (>768px)**: 2x2 grid layout
- **Mobile (≤768px)**: Full-width single column layout

## Error Handling

If an API call fails:
- An error message is displayed at the top of the cascade container
- The failed dropdown retains its previous value
- Other dropdowns remain disabled until the error is resolved
- User can try again by re-selecting the parent dropdown

## Performance Considerations

1. **API Caching**: The `apiClient` uses a request cache with 30-second TTL
2. **Lazy Loading**: Data is only fetched when needed (on parent selection)
3. **Indexed Queries**: Backend queries use indexes on `divisionId`, `districtId`, and `upazilaId`
4. **Lean Projections**: Only necessary fields are returned from the API
5. **Cascade Reset**: When a parent changes, dependent dropdowns are already reset before loading new data

## Testing Checklist

- [ ] Divisions load on page mount
- [ ] Selecting a division enables district dropdown
- [ ] Selecting a district enables upazila dropdown
- [ ] Selecting an upazila enables union dropdown
- [ ] Changing a parent dropdown resets children
- [ ] Loading indicators show during API calls
- [ ] Error messages display if API fails
- [ ] Form submission includes all location IDs
- [ ] Mobile layout is responsive
- [ ] Dropdowns have proper accessible labels and IDs

## Future Enhancements

1. Add a search/filter feature to large dropdown lists
2. Add keyboard navigation support
3. Implement location abbreviations display
4. Add location preview or validation
5. Cache location data in local storage for offline support
6. Add ability to clear selections and start over
