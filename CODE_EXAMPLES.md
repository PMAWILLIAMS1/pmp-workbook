# Code Examples: Key Features Demonstrated

## Form Field Examples

### Basic Input Fields
```javascript
// Text input example
<input
  id="author"
  type="text"
  name="author"
  value={formData.author}
  onChange={handleInputChange}
  className="w-full p-3 rounded-lg border-2 border-gray-200 focus:border-pmi-blue"
  placeholder="Your full name"
/>

// Checkbox example
<input
  id="eduSecondaryChecked"
  type="checkbox"
  name="eduSecondaryChecked"
  checked={!!formData.eduSecondaryChecked}
  onChange={handleInputChange}
  className="h-4 w-4 text-pmi-gold border-gray-300 rounded focus:ring-pmi-blue"
/>

// Textarea example
<textarea
  name="eligibilityNotes"
  value={formData.eligibilityNotes}
  onChange={handleInputChange}
  rows="4"
  className="w-full p-3 rounded-lg border-2 border-gray-200 focus:border-pmi-blue"
  placeholder="Additional notes about your eligibility..."
/>
```

### Radio Button Groups
```javascript
{["8-week", "12-week", "self-paced"].map((option) => (
  <div key={option} className="flex items-center">
    <input
      id={`study-${option}`}
      type="radio"
      name="studyPlanOption"
      value={option}
      checked={formData.studyPlanOption === option}
      onChange={handleInputChange}
      className="h-4 w-4 text-pmi-gold border-gray-300 focus:ring-pmi-blue"
    />
    <label htmlFor={`study-${option}`} className="ml-2 text-gray-700">
      {option.charAt(0).toUpperCase() + option.slice(1)} Plan
    </label>
  </div>
))}
```

## Navigation System Examples

### Section Navigation
```javascript
const NavLink = ({ id, title, IconComp }) => (
  <button
    onClick={() => setCurrentPage(id)}
    className={`w-full flex items-center p-3 rounded-lg text-left transition-colors ${
      currentPage === id
        ? "bg-pmi-blue text-white"
        : "text-gray-700 hover:bg-gray-100"
    }`}
    aria-current={currentPage === id ? "page" : undefined}
  >
    <IconComp className="mr-3 flex-shrink-0" />
    <span className="flex-1">{title}</span>
    {hasContent(id) && (
      <span className="ml-2 w-2 h-2 bg-pmi-gold rounded-full flex-shrink-0" />
    )}
  </button>
);
```

### Page Wrapper with Scroll Persistence
```javascript
const PageWrapper = ({ id, children }) => {
  const isActive = currentPage === id;
  const scrollerRef = useRef(null);
  
  useEffect(() => {
    const el = scrollerRef.current;
    if (!el) return;
    
    // Restore scroll position
    el.scrollTop = scrollPositions.current[id] || 0;
    
    const onScroll = () => {
      scrollPositions.current[id] = el.scrollTop;
    };
    
    el.addEventListener("scroll", onScroll);
    return () => el.removeEventListener("scroll", onScroll);
  }, [id, currentPage]);

  return (
    <section
      aria-labelledby={`${id}-heading`}
      className={`absolute inset-0 transition-opacity duration-300 ${
        isActive ? "opacity-100 pointer-events-auto" : "opacity-0 pointer-events-none"
      }`}
    >
      <div
        ref={scrollerRef}
        className="h-full overflow-y-auto bg-white p-8 md:p-12"
      >
        {children}
      </div>
    </section>
  );
};
```

## Data Management Examples

### localStorage Integration
```javascript
// Load saved data on component mount
useEffect(() => {
  const saved = localStorage.getItem("pmpWorkbook");
  if (saved) {
    try {
      setFormData(JSON.parse(saved));
    } catch (error) {
      console.error("Failed to load saved data:", error);
    }
  }
}, []);

// Save data whenever form changes
useEffect(() => {
  localStorage.setItem("pmpWorkbook", JSON.stringify(formData));
}, [formData]);
```

### Progress Tracking
```javascript
function hasContent(id) {
  const keys = {
    eligibility: [
      "eduSecondaryChecked", "eduFourYearChecked", "exp60MonthsChecked",
      "exp36MonthsChecked", "edu35HoursChecked", "capmChecked"
    ],
    audits: [
      "auditSupervisorFormsChecked", "auditEducationCertificatesChecked",
      "audit35HourCertificateChecked", "auditCapmCertificateChecked"
    ],
    "non-traditional": [
      "nTProjectTitle1", "nTOrganization1", "nTYourRole1", "nTReflection1"
    ],
    "exam-prep": [
      "studyPlanOption", "studyPlanReflection", "studyWeek1", "studyWeek2"
    ],
    "exam-day": [
      "examDayOnlineChecked", "examDayInPersonChecked", 
      "examDayValidIdChecked", "examDayReflection"
    ],
    bonus: ["bonusReflection"],
    appendix: [],
    cover: ["author"],
  };
  
  const list = keys[id] || [];
  return list.some((k) => !!formData[k]);
}
```

## Export/Import Examples

### JSON Export Functionality
```javascript
function exportJSON() {
  const blob = new Blob([JSON.stringify(formData, null, 2)], {
    type: "application/json",
  });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "pmp-workbook.json";
  a.click();
  URL.revokeObjectURL(url);
}
```

### JSON Import Functionality
```javascript
function importJSON(file) {
  if (!file) return;
  
  file.text().then((text) => {
    try {
      setFormData(JSON.parse(text));
    } catch (error) {
      alert("Invalid JSON file");
    }
  });
}

// File input handling
<input
  type="file"
  accept=".json"
  onChange={(e) => importJSON(e.target.files[0])}
  className="hidden"
  id="import-file"
/>
```

## Print Functionality Examples

### Section-Specific Printing
```javascript
function printSection(id) {
  setCurrentPage(id);
  setTimeout(() => window.print(), 50);
}

// Print buttons
{sections.map((s) => (
  <button
    key={`print-${s.id}`}
    onClick={() => printSection(s.id)}
    className="rounded-lg border px-2 py-1 text-sm hover:bg-gray-50"
    title={`Print ${s.title}`}
  >
    Print {s.id === "toc" ? "TOC" : s.title.split(".")[0]}
  </button>
))}
```

### Print-Friendly CSS
```css
@media print {
  .no-print {
    display: none !important;
  }
  
  .shadow-xl,
  .shadow-lg,
  .shadow-md {
    box-shadow: none !important;
  }
  
  .rounded-2xl,
  .rounded-xl,
  .rounded-lg {
    border-radius: 0 !important;
  }
  
  .bg-gradient-to-br {
    background: white !important;
  }
}
```

## Icon System Examples

### Base Icon Component
```javascript
const Icon = ({ children, className, size = 22, style }) => (
  <svg
    xmlns="http://www.w3.org/2000/svg"
    width={size}
    height={size}
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    strokeWidth="2"
    strokeLinecap="round"
    strokeLinejoin="round"
    className={className}
    style={style}
  >
    {children}
  </svg>
);
```

### Specific Icon Implementations
```javascript
const BookOpen = (p) => (
  <Icon {...p}>
    <path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"></path>
    <path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"></path>
  </Icon>
);

const ClipboardList = (p) => (
  <Icon {...p}>
    <rect x="8" y="2" width="8" height="4" rx="1" ry="1"></rect>
    <path d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"></path>
    <path d="M12 11h4"></path>
    <path d="M12 16h4"></path>
    <path d="M8 11h.01"></path>
    <path d="M8 16h.01"></path>
  </Icon>
);
```

## Responsive Design Examples

### Mobile-First Sidebar
```javascript
<aside
  className={`no-print bg-white rounded-2xl shadow-lg p-6 w-full md:w-72 md:block ${
    isSidebarOpen ? "block" : "hidden md:block"
  }`}
>
  {/* Sidebar content */}
</aside>
```

### Mobile Navigation Header
```javascript
<div className="md:hidden sticky top-0 z-50 bg-white rounded-xl shadow p-3 flex items-center justify-between no-print">
  <button
    onClick={() => setIsSidebarOpen((s) => !s)}
    className="p-2 rounded-lg"
    aria-label="Open menu"
  >
    <Menu className="text-gray-600" />
  </button>
  <span className="text-lg font-bold text-pmi-blue">
    PMP Blueprint™
  </span>
  <span className="w-8" />
</div>
```

## Form Validation Examples

### Input Change Handler
```javascript
function handleInputChange(e) {
  const { name, value, type, checked } = e.target;
  setFormData((prev) => ({
    ...prev,
    [name]: type === "checkbox" ? checked : value,
  }));
}
```

### Conditional Field Rendering
```javascript
{formData.eduSecondaryChecked && (
  <div className="ml-6 mt-2">
    <label htmlFor="eduSecondaryType" className="block text-sm font-medium text-gray-700 mb-1">
      Type of secondary education:
    </label>
    <input
      id="eduSecondaryType"
      type="text"
      name="eduSecondaryType"
      value={formData.eduSecondaryType}
      onChange={handleInputChange}
      className="w-full p-2 rounded border-gray-300 focus:border-pmi-blue"
      placeholder="e.g., High School Diploma, GED"
    />
  </div>
)}
```

These examples demonstrate the practical implementation of a modern, user-friendly form application using React hooks and modern JavaScript without a build process.