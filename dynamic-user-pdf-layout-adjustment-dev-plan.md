# Dynamic User PDF Layout Adjustment Development Plan

## Overview

This plan outlines the steps to enhance the PDF layout system by adding user control over pagination while maintaining the current functionality and styling. The key improvements include:

1. User-configurable items per page
2. Automatic resizing based on user settings
3. Visual separation of pages with shadowed card containers
4. Preservation of all current styling and functionality

## Current System Analysis

The current system already implements:
- Dynamic scaling based on line item count
- Smart pagination for large numbers of items
- Optimized PDF generation settings
- Enhanced CSS print styles
- Flexible layout containers
- Content overflow protection

We'll build upon this foundation to add user control while preserving these benefits.

## Step-by-Step Implementation Plan

### Step 1: Create User Controls for Pagination Settings

1. Add a pagination settings interface to the requisition preview page:
   - Create a settings panel with controls for items per page
   - Add a toggle for auto-sizing vs. manual pagination
   - Include visual preview of page breaks

2. Implement the settings UI component:
```tsx
interface PaginationSettings {
  itemsPerPage: number;
  autoSize: boolean;
}

const PaginationSettingsPanel: FC<{
  settings: PaginationSettings;
  onSettingsChange: (settings: PaginationSettings) => void;
  totalItems: number;
}> = ({ settings, onSettingsChange, totalItems }) => {
  return (
    <div className="bg-white p-4 rounded-lg shadow-sm border border-gray-200 mb-4">
      <h3 className="text-lg font-medium mb-3">Layout Settings</h3>
      
      <div className="space-y-4">
        <div className="flex items-center justify-between">
          <label htmlFor="auto-size" className="text-sm font-medium text-gray-700">
            Auto-size content
          </label>
          <Switch
            id="auto-size"
            checked={settings.autoSize}
            onCheckedChange={(checked) => 
              onSettingsChange({ ...settings, autoSize: checked })
            }
          />
        </div>
        
        <div className={settings.autoSize ? "opacity-50 pointer-events-none" : ""}>
          <label htmlFor="items-per-page" className="block text-sm font-medium text-gray-700 mb-1">
            Items per page
          </label>
          <div className="flex items-center gap-3">
            <Slider
              id="items-per-page"
              min={1}
              max={Math.min(20, totalItems)}
              step={1}
              value={[settings.itemsPerPage]}
              onValueChange={(value) => 
                onSettingsChange({ ...settings, itemsPerPage: value[0] })
              }
              disabled={settings.autoSize}
            />
            <span className="text-sm font-medium w-8 text-center">
              {settings.itemsPerPage}
            </span>
          </div>
          <p className="text-xs text-gray-500 mt-1">
            {Math.ceil(totalItems / settings.itemsPerPage)} page(s) total
          </p>
        </div>
      </div>
    </div>
  );
};
```

### Step 2: Modify the RequisitionForm Component to Support User-Defined Pagination

1. Update the `RequisitionForm` component to accept pagination settings:
```tsx
export interface RequisitionFormProps {
  // Existing props...
  paginationSettings?: {
    itemsPerPage: number;
    autoSize: boolean;
  };
}
```

2. Modify the pagination logic to respect user settings:
```typescript
// Split items into pages based on user settings
const pages = useMemo(() => {
  // If auto-size is enabled or no pagination settings provided, use the current logic
  if (!paginationSettings || paginationSettings.autoSize) {
    // If we have 20 or fewer items, try to fit them on one page with dynamic scaling
    if (line_items.length <= 20) {
      return [line_items];
    }
    
    // For more than 20 items, split into multiple pages with default itemsPerPage
    const itemsPerPage = 20;
    const result = [];
    
    for (let i = 0; i < line_items.length; i += itemsPerPage) {
      result.push(line_items.slice(i, i + itemsPerPage));
    }
    
    return result;
  }
  
  // If manual pagination is enabled, use the user-defined itemsPerPage
  const { itemsPerPage } = paginationSettings;
  const result = [];
  
  for (let i = 0; i < line_items.length; i += itemsPerPage) {
    result.push(line_items.slice(i, i + itemsPerPage));
  }
  
  return result;
}, [line_items, paginationSettings]);
```

### Step 3: Enhance Dynamic Scaling to Work with User-Defined Page Sizes

1. Update the dynamic scaling functions to consider items per page:
```typescript
// Calculate row height class based on items per page and total items
const getRowHeightClass = useMemo(() => {
  // Get the number of items on the current page
  const currentPageItemCount = pages.length > 0 ? pages[0].length : 0;
  
  // Base height for normal display (up to 5 items)
  if (currentPageItemCount <= 5) {
    return "py-1.5";
  }
  // Slightly reduced height for 6-7 items
  else if (currentPageItemCount <= 7) {
    return "py-1";
  }
  // More compact for 8-9 items
  else if (currentPageItemCount <= 9) {
    return "py-0.5 h-6";
  }
  // Very compact for 10-12 items
  else if (currentPageItemCount <= 12) {
    return "py-0.5";
  }
  // Extra compact for 13-15 items
  else if (currentPageItemCount <= 15) {
    return "py-0";
  }
  // Extremely compact for more than 15 items
  else {
    return "py-0 h-5";
  }
}, [pages]);

// Similar updates for other dynamic styling functions...
```

### Step 4: Implement Visual Page Separation with Shadowed Card Containers

1. Modify the page rendering to include visual separation:
```tsx
// Render a page of the form with visual separation
const renderPage = (pageItems: LineItem[], pageIndex: number, isLastPage: boolean) => {
  const isFirstPage = pageIndex === 0;
  
  return (
    <div 
      key={pageIndex}
      className={`mb-8 ${pageIndex > 0 ? 'mt-12' : 'mt-4'}`}
    >
      {/* Page number indicator for preview mode */}
      {pages.length > 1 && (
        <div className="text-center mb-2 text-sm font-medium text-gray-500 print:hidden">
          Page {pageIndex + 1} of {pages.length}
        </div>
      )}
      
      {/* Page container with shadow */}
      <div 
        data-testid={isFirstPage ? "requisition-form" : `requisition-form-page-${pageIndex}`}
        className={`w-[11in] min-h-[8.5in] bg-white p-4 font-['Helvetica'] text-black rounded-lg shadow-lg mx-auto ${pageIndex > 0 ? 'page-break-before' : ''}`}
      >
        {/* Main Border */}
        <div className="rounded-lg border border-gray-300 h-full flex flex-col bg-white overflow-hidden">
          {/* Existing content... */}
        </div>
      </div>
    </div>
  );
};
```

2. Add print-specific styles to handle the new page containers:
```css
@media print {
  /* Hide page number indicators in print */
  .print-controls, .page-number-indicator {
    display: none;
  }
  
  /* Remove shadows and spacing in print */
  .page-container {
    box-shadow: none !important;
    margin: 0 !important;
    padding: 0 !important;
  }
  
  /* Ensure proper page breaks */
  .page-break-before {
    page-break-before: always;
  }
}
```

### Step 5: Update the PDF Generation Settings to Account for User Pagination

1. Modify the PDF generation settings calculation:
```typescript
function calculatePdfSettings(data: RequisitionFormProps) {
  // Base settings
  const settings = {
    format: 'Letter' as const,
    landscape: true,
    printBackground: true,
    preferCSSPageSize: true,
    margin: {
      top: '0.1in',
      right: '0.1in',
      bottom: '0.1in',
      left: '0.1in'
    },
    scale: 0.98
  };

  // If using auto-sizing, adjust scale based on items per page
  if (!data.paginationSettings || data.paginationSettings.autoSize) {
    // Get the number of items on the first page
    const itemsOnFirstPage = data.line_items.length;
    
    // Apply existing scaling logic
    if (itemsOnFirstPage > 15) {
      settings.scale = 0.94;
      // Adjust margins...
    } else if (itemsOnFirstPage > 10) {
      settings.scale = 0.96;
      // Adjust margins...
    } else if (itemsOnFirstPage >= 8 && itemsOnFirstPage <= 9) {
      settings.scale = 0.97;
      // Adjust margins...
    }
  } else {
    // For manual pagination, calculate scale based on items per page
    const itemsPerPage = data.paginationSettings.itemsPerPage;
    
    if (itemsPerPage > 15) {
      settings.scale = 0.94;
      // Adjust margins...
    } else if (itemsPerPage > 10) {
      settings.scale = 0.96;
      // Adjust margins...
    } else if (itemsPerPage >= 8 && itemsPerPage <= 9) {
      settings.scale = 0.97;
      // Adjust margins...
    }
  }

  return settings;
}
```

### Step 6: Integrate Pagination Settings with the Requisition Preview Page

1. Add state management for pagination settings:
```tsx
export default function RequisitionPreview({ searchParams }: { searchParams: { data: string } }) {
  // Parse the form data from the search params
  const formData = searchParams.data ? JSON.parse(decodeURIComponent(searchParams.data)) : {};
  
  // Calculate line item count for conditional styling
  const lineItemCount = formData.line_items?.length || 0;
  
  // State for pagination settings
  const [paginationSettings, setPaginationSettings] = useState<PaginationSettings>({
    itemsPerPage: Math.min(10, lineItemCount),
    autoSize: true
  });
  
  // Pass settings to RequisitionForm
  return (
    <>
      <style jsx global>{/* Existing styles */}</style>
      <div className="flex flex-col items-center">
        <div className="print-controls sticky top-0 z-10 w-full bg-white border-b p-4 mb-4 flex justify-between">
          <h1 className="text-xl font-bold">Requisition Preview</h1>
          <div className="flex items-center gap-4">
            <Button variant="outline" onClick={() => window.print()}>Print</Button>
            <Button onClick={() => setShowSettings(!showSettings)}>
              {showSettings ? 'Hide Settings' : 'Show Settings'}
            </Button>
          </div>
        </div>
        
        {showSettings && (
          <div className="w-full max-w-2xl mb-6">
            <PaginationSettingsPanel
              settings={paginationSettings}
              onSettingsChange={setPaginationSettings}
              totalItems={lineItemCount}
            />
          </div>
        )}
        
        <div className="requisition-form-container">
          <RequisitionForm 
            {...formData} 
            paginationSettings={paginationSettings}
          />
        </div>
      </div>
    </>
  );
}
```

### Step 7: Update the API Route to Pass Pagination Settings

1. Modify the API route to accept pagination settings:
```typescript
export async function POST(req: NextRequest) {
  try {
    const data: RequisitionFormProps = await req.json();
    
    // Rest of the existing code...
    
    // Navigate to our form page with the data including pagination settings
    const url = `${baseUrl}/requisition-preview?data=${encodeURIComponent(JSON.stringify(data))}`;
    
    // Rest of the existing code...
  } catch (error) {
    // Error handling...
  }
}
```

### Step 8: Persist User Preferences

1. Add local storage for pagination settings:
```typescript
// Hook for persisting pagination settings
function usePaginationSettings(initialSettings: PaginationSettings) {
  // Initialize state from localStorage if available
  const [settings, setSettings] = useState<PaginationSettings>(() => {
    if (typeof window === 'undefined') return initialSettings;
    
    const savedSettings = localStorage.getItem('paginationSettings');
    return savedSettings ? JSON.parse(savedSettings) : initialSettings;
  });
  
  // Save to localStorage when settings change
  useEffect(() => {
    if (typeof window !== 'undefined') {
      localStorage.setItem('paginationSettings', JSON.stringify(settings));
    }
  }, [settings]);
  
  return [settings, setSettings] as const;
}
```

2. Use the hook in the Requisition Preview page:
```typescript
const [paginationSettings, setPaginationSettings] = usePaginationSettings({
  itemsPerPage: Math.min(10, lineItemCount),
  autoSize: true
});
```

### Step 9: Add Visual Feedback for Page Breaks

1. Implement a visual indicator for page breaks in the preview mode:
```tsx
{pages.length > 1 && pageIndex < pages.length - 1 && (
  <div className="page-break-indicator print:hidden">
    <div className="border-t-2 border-dashed border-gray-300 my-4 relative">
      <span className="absolute top-0 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-white px-2 text-xs text-gray-500">
        Page Break
      </span>
    </div>
  </div>
)}
```

### Step 10: Testing and Validation

1. Test with various configurations:
   - Different numbers of items per page
   - Auto-sizing vs. manual pagination
   - Different total numbers of line items
   - Print preview and actual PDF generation

2. Verify across different browsers and devices:
   - Chrome, Firefox, Safari
   - Desktop and mobile views

3. Validate PDF output:
   - Check that all content is visible
   - Ensure footer is properly positioned
   - Verify pagination works correctly
   - Confirm that user settings are respected

## Implementation Priority

1. Step 1: Create User Controls for Pagination Settings (High Priority)
2. Step 2: Modify the RequisitionForm Component to Support User-Defined Pagination (High Priority)
3. Step 4: Implement Visual Page Separation with Shadowed Card Containers (High Priority)
4. Step 3: Enhance Dynamic Scaling to Work with User-Defined Page Sizes (Medium Priority)
5. Step 5: Update the PDF Generation Settings to Account for User Pagination (Medium Priority)
6. Step 6: Integrate Pagination Settings with the Requisition Preview Page (Medium Priority)
7. Step 9: Add Visual Feedback for Page Breaks (Medium Priority)
8. Step 7: Update the API Route to Pass Pagination Settings (Low Priority)
9. Step 8: Persist User Preferences (Low Priority)
10. Step 10: Testing and Validation (Critical)

## Potential Challenges and Mitigations

### Challenge 1: Maintaining Print Quality with User Settings
**Mitigation**: Implement guardrails that prevent users from selecting configurations that would result in unreadable content. Add warnings when settings might lead to poor print quality.

### Challenge 2: Performance with Many Pages
**Mitigation**: Implement virtualization for preview mode when there are many pages, rendering only visible pages to improve performance.

### Challenge 3: Consistent Styling Across Different Page Configurations
**Mitigation**: Create a comprehensive set of styling rules that adapt to different page sizes and item counts, ensuring consistency regardless of user settings.

### Challenge 4: Browser Compatibility for Print Styles
**Mitigation**: Test extensively across browsers and implement browser-specific CSS where needed to ensure consistent printing behavior.

## Conclusion

This development plan provides a comprehensive approach to implementing user-controlled pagination with dynamic resizing and improved visual presentation. By following these steps, we can enhance the PDF layout system while preserving all current functionality and styling.

The implementation will give users greater control over how their requisition forms are paginated and displayed, while ensuring that the system continues to produce high-quality, professional-looking PDFs regardless of the chosen settings. 