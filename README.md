# Widget Development Process

This document outlines the step-by-step process for developing a new widget in the Poper application.
# Widget System Developer Guide

## 📚 Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [File Structure](#file-structure)
4. [Creating a New Widget Type](#creating-a-new-widget-type)
5. [Creating a New Template](#creating-a-new-template)
6. [Component System](#component-system)
7. [Preview System](#preview-system)
8. [Backend Integration](#backend-integration)
9. [Embed System](#embed-system)
10. [Styling Guide](#styling-guide)
11. [Best Practices](#best-practices)
12. [Case Study: Instagram Feed Profile Widget](#case-study-instagram-feed-profile-widget)

---

## Overview

The widget system is a flexible, modular architecture that allows developers to create customizable widgets that can be embedded on external websites. Each widget consists of:

- **Widget Type**: The category (e.g., `instagram-feed`, `google-reviews`)
- **Template**: A specific variation of a widget type (e.g., `profile-widget`, `grid`)
- **Settings**: User-configurable options stored as a flat JSON object
- **Preview**: Live preview component used in the editor
- **Front Component**: Production component used on embed sites

---

## Architecture

### High-Level Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    Widget Templates                         │
│              (widget-templates.js)                           │
│  Defines: Options, Menus, Default Values, Dependencies      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                    Widget Editor                            │
│                  (editor/index.js)                          │
│  - Loads template                                           │
│  - Renders MenuInputRenderer for each option/menu          │
│  - Manages widgetSettings state (flat object)               │
│  - Auto-saves to backend                                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              MenuInputRenderer                               │
│          (components/MenuInputRenderer.js)                   │
│  - Routes to appropriate input component based on menu.type │
│  - Handles dependencies (dependsOn)                          │
│  - Supports basic & complex field types                      │
└────────────────────┬────────────────────────────────────────┘
                     │
         ┌───────────┴───────────┐
         │                       │
         ▼                       ▼
┌─────────────────┐    ┌──────────────────────┐
│  Basic Fields   │    │  Complex Fields      │
│  (text, toggle, │    │  (instafields/,       │
│   color, etc.)  │    │   googlefields/)      │
└─────────────────┘    └──────────────────────┘
         │                       │
         └───────────┬───────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                    Widget Preview                            │
│              (preview/index.js)                              │
│  - Receives widgetSettings                                   │
│  - Renders appropriate PreviewComponent                      │
│  - Updates in real-time as settings change                   │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              Preview Component                               │
│    (preview/{widget-type}/{Template}Preview.js)             │
│  - Renders actual widget UI                                  │
│  - Uses settings to customize appearance                     │
│  - May use helper components (e.g., InstagramPost)          │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **Template Definition** → Defines available options and menus
2. **Editor** → User interacts with inputs, updates `widgetSettings`
3. **Preview** → Receives `widgetSettings`, renders live preview
4. **Backend** → Saves `widgetSettings` as JSON in `widgets.settings` column
5. **Embed** → Front component loads settings from backend and renders

---

## File Structure

```
app/src/pages/widgets/
├── data/
│   └── widget-templates.js          # Widget type & template definitions
├── components/
│   ├── MenuInputRenderer.js          # Routes to input components
│   ├── instafields/                  # Instagram-specific fields
│   │   ├── StyleCarousel.js
│   │   ├── LayoutSelector.js
│   │   ├── SourceTypeInput.js
│   │   └── ...
│   ├── googlefields/                 # Google Reviews-specific fields
│   └── ...
├── editor/
│   └── index.js                       # Main editor component
├── preview/
│   ├── index.js                       # Preview router
│   ├── instagram-feed/
│   │   ├── ProfileWidgetPreview.js   # Template preview component
│   │   ├── GridWidgetPreview.js
│   │   ├── components/                # Reusable preview components
│   │   │   ├── InstagramPost.js
│   │   │   ├── InstagramHeader.js
│   │   │   └── ...
│   │   ├── utils/
│   │   │   └── helpers.js            # Helper functions
│   │   ├── data/
│   │   │   └── staticData.js         # Static data for preview
│   │   ├── front.js                  # Front component router
│   │   └── index.scss                # Styles
│   └── ...
├      
└── templates/
    └── index.js                       # Template selection UI

backend/api/
├── routes/widgets/
│   └── index.js                       # Widget API endpoints
├── lib/postgres.js                    # Database functions
└── routes/
    └── ...                            # API routes
```

---

## Creating a New Widget Type

### Step 1: Define Widget in `widget-templates.js`

Add your widget type to the `WIDGET_TEMPLATES` object:

```javascript
export const WIDGET_TEMPLATES = {
  "my-widget": {
    id: "my-widget",
    name: "My Widget",
    icon: "🎨", // or path to icon image
    description: "Description of your widget",
    templates: [
      // Templates go here (see next section)
    ],
  },
};
```

### Step 2: Create Preview Component

Create `preview/my-widget/MyTemplatePreview.js`:

```javascript
import React from "react";

function MyTemplatePreview({ settings, viewMode = "desktop", data }) {
  // settings: Flat object with all user settings
  // viewMode: "desktop" | "mobile"
  // data: Optional data from backend (e.g., posts, profile)
  
  return (
    <div className="widget-preview-wrapper">
      <div className="widget-preview-container">
        {/* Your widget UI here */}
        <h1>{settings?.title || "My Widget"}</h1>
      </div>
    </div>
  );
}

export default MyTemplatePreview;
```

### Step 3: Register Preview in `preview/index.js`

```javascript
import MyTemplatePreview from "./my-widget/MyTemplatePreview";

const WIDGET_PREVIEWS = {
  "my-widget": {
    "my-template": MyTemplatePreview,
  },
};
```

### Step 4: Create Front Component (for embed)

Create `preview/my-widget/front.js`:

```javascript
import React from "react";
import MyTemplatePreview from "./MyTemplatePreview";

export default function Template({ template, ...props }) {
  switch (template) {
    case "my-template":
      return <MyTemplatePreview {...props} />;
    default:
      return <MyTemplatePreview {...props} />;
  }
}
```

### Step 5: Add Styles

Create `preview/my-widget/index.scss`:

```scss
.widget-preview-wrapper {
  // Your styles
}
```

Import in `preview/index.js`:

```javascript
import "./my-widget/index.scss";
```

---

## Creating a New Template

### Step 1: Add Template Definition in settings.js

In `widget-templates.js`, add a template to your widget type:

```javascript
"my-widget": {
  // ... widget definition
  templates: [
    {
      id: "my-template",
      name: "My Template",
      description: "Description of this template",
      isRecommended: true, // Optional: shows "Recommended" badge
      previewImage: "/widgets/my-template.png", // Optional: preview image
      options: [
        // Options define the settings structure
        {
          id: "content",
          name: "Content",
          menus: [
            {
              id: "title",
              label: "Title",
              type: "text",
              defaultValue: "My Widget Title",
            },
            {
              id: "showSubtitle",
              label: "Show Subtitle",
              type: "toggle",
              defaultValue: true,
            },
          ],
        },
        {
          id: "style",
          name: "Style",
          menus: [
            {
              id: "backgroundColor",
              label: "Background Color",
              type: "color",
              defaultValue: "#ffffff",
            },
          ],
        },
      ],
    },
  ],
},
```

### Step 2: Create Preview Component

See "Creating a New Widget Type" → Step 2.

### Step 3: Register Preview

See "Creating a New Widget Type" → Step 3.

---

## Component System

### Menu Types

The `MenuInputRenderer` supports these menu types:

#### Basic Types (Built-in)

- `text` - Text input
- `textarea` - Multi-line text
- `number` - Number input
- `toggle` - Boolean switch
- `select` - Dropdown
- `color` - Color picker
- `slider` - Range slider
- `segmented-control` - Segmented button group
- `radio` - Radio button group
- `checkbox-group` - Multiple checkboxes
- `datetime` - Date/time picker

#### Complex Types (Custom Components)

- `style-carousel` - Style selector with previews
- `layout-selector` - Layout selection
- `layout-grid` - Grid layout selector
- `source-type-input` - Source type selector
- `expandable-section` - Collapsible section with sub-menus
- `mobile-breakpoints` - Responsive breakpoint editor
- `search-input` - Search input with button
- `source-list` - List of sources with add/remove
- `rich-text` - Rich text editor
- `google-color-scheme` - Color scheme selector

 `:

```javascript
import MyCustomField from "./myfields/MyCustomField";

// In the switch statement:
case "my-custom-field":
  return <MyCustomField {...props} />;
```

### Dependencies

Use `dependsOn` to conditionally show/hide menus:

```javascript
{
  id: "subtitle",
  label: "Subtitle",
  type: "text",
  defaultValue: "",
  dependsOn: { menuId: "showSubtitle", value: true },
}
```

The menu will only render if `widgetSettings.showSubtitle === true`.

### Expandable Sections

Create nested menus:

```javascript
{
  id: "headerStyle",
  label: "Header",
  type: "expandable-section",
  subMenus: [
    {
      id: "headerBackground",
      label: "Background Color",
      type: "color",
      defaultValue: "#ffffff",
    },
    {
      id: "headerTextColor",
      label: "Text Color",
      type: "color",
      defaultValue: "#000000",
    },
  ],
}
```

---

## Preview System

### Preview Component Structure

```javascript
function MyTemplatePreview({ settings, viewMode, data }) {
  // Access settings (flat object)
  const title = settings?.title || "Default";
  const backgroundColor = settings?.backgroundColor || "#ffffff";
  
  // Handle view mode
  const containerClass = viewMode === "mobile" 
    ? "widget-preview-container-mobile"
    : "widget-preview-container-desktop";
  
  return (
    <div className="widget-preview-wrapper">
      <div className={containerClass}>
        {/* Your widget UI */}
      </div>
    </div>
  );
}
```

  
```

---

## Backend Integration

### Database Schema

Widgets are stored in the `widgets` table:

```sql
CREATE TABLE widgets (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255),
  settings JSONB,              -- Flat object with all settings
  status BOOLEAN DEFAULT false,
  type VARCHAR(100),           -- Widget type (e.g., "instagram-feed")
  domain_id INTEGER,
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);
```

### API Endpoints

#### Get Widget Instance

```javascript
// Frontend
const widget = await getWidgetInstanceById(widgetId);

// Backend: backend/api/lib/postgres.js
async function getWidgetById(widgetId) {
  const response = await pool.query(
    "SELECT * FROM widgets WHERE id = $1",
    [widgetId]
  );
  return response.rows?.[0] || null;
}
```

#### Create Widget Instance

```javascript
// Frontend
const widget = await createWidgetInstance({
  title: "My Widget",
  settings: widgetSettings, // Flat object
  type: "my-widget",
  domain_id: currentDomain.id,
});

// Backend
async function createWidget(widgetData) {
  const { title, settings, type, domain_id } = widgetData;
  const response = await pool.query(
    `INSERT INTO widgets (title, settings, type, domain_id, created_at, updated_at)
     VALUES ($1, $2::jsonb, $3, $4, NOW(), NOW())
     RETURNING *`,
    [title, JSON.stringify(settings), type, domain_id]
  );
  return response.rows?.[0];
}
```

#### Update Widget Instance

```javascript
// Frontend
await updateWidgetInstance(widgetId, {
  title: "Updated Title",
  settings: updatedSettings,
});

// Backend
async function updateWidget(widgetId, updates) {
  const { title, settings, status } = updates;
  const response = await pool.query(
    `UPDATE widgets 
     SET title = COALESCE($1, title),
         settings = COALESCE($2::jsonb, settings),
         status = COALESCE($3, status),
         updated_at = NOW()
     WHERE id = $4
     RETURNING *`,
    [title, settings ? JSON.stringify(settings) : null, status, widgetId]
  );
  return response.rows?.[0];
}
```

### Widget-Specific Data

For widgets that need external data (e.g., Instagram posts), use `wigets_data` table:

```sql
CREATE TABLE wigets_data (
  id SERIAL PRIMARY KEY,
  widget_id INTEGER REFERENCES widgets(id),
  key VARCHAR(255),           -- e.g., "posts", "profile"
  data JSONB,                 -- The actual data
  created_at TIMESTAMP
);
```

Save data:

```javascript
// Backend
async function saveWidgetData(widgetId, data, key = "default") {
  await pool.query(
    "INSERT INTO wigets_data (widget_id, key, data) VALUES ($1, $2, $3::jsonb)",
    [widgetId, key, JSON.stringify(data)]
  );
}
```

---

## Embed System

### Embed Code Generation

The embed code consists of:

1. **Account Script** - Loads once per site
2. **Widget Placeholder** - `<div class="poper-{widgetId}"></div>`

Generated in `components/EmbedCodeModal.js`:

```javascript
const accountScript = getPopupScript(userEmail);
const widgetTarget = `<div class="poper-${widget.id}"></div>`;
const embedCode = `${accountScript}\n${widgetTarget}`;
```

### Front Component Loading

When the embed script loads, it:

1. Fetches widget settings from backend
2. Renders the appropriate front component
3. Injects into the placeholder div

### Creating Front Component

Your preview component can often be reused as the front component. Ensure it:

- Works without React context dependencies
- Handles loading states
- Fetches data if needed
- Is responsive

---

## Styling Guide

### SCSS Structure

```scss
// preview/my-widget/index.scss

// Container styles
.widget-preview-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  background-color: #f9fafb;
}

.widget-preview-container {
  position: relative;
  background-color: white;
  overflow: hidden;
}

// View mode variants
.widget-preview-container-mobile {
  border-radius: 12px;
}

.widget-preview-container-desktop {
  border-radius: 0;
}

// Component-specific styles
.my-widget-header {
  padding: 1rem;
  background-color: var(--header-bg, #ffffff);
}
```

### Using Tailwind CSS

Preview components can use Tailwind classes:

```javascript
<div className="flex items-center justify-center p-4 bg-white rounded-lg">
  <h1 className="text-2xl font-bold text-gray-900">{title}</h1>
</div>
```

### CSS Variables

Use CSS variables for dynamic colors:

```scss
.my-widget {
  --primary-color: #2563eb;
  --text-color: #000000;
  
  color: var(--text-color);
  background-color: var(--primary-color);
}
```

Set in component:

```javascript
<div 
  className="my-widget"
  style={{
    "--primary-color": settings?.primaryColor || "#2563eb",
    "--text-color": settings?.textColor || "#000000",
  }}
>
```

---

## Best Practices

### 1. Settings Structure

- **Use flat objects** - No nesting by option ID
- **Use descriptive IDs** - `headerBackground` not `bg1`
- **Provide defaults** - Always set `defaultValue`
- **Validate types** - Ensure settings match expected types

### 2. Component Organization

- **Separate concerns** - Preview components, helper components, utilities
- **Reuse components** - Create shared components in `components/` folder
- **Keep previews simple** - Focus on rendering, not business logic

### 3. Performance

- **Memoize expensive renders** - Use `React.memo` for post/item components
- **Lazy load data** - Fetch data only when needed
- **Debounce auto-save** - Editor auto-saves with 100ms debounce

### 4. Accessibility

- **Use semantic HTML** - Proper heading hierarchy, landmarks
- **Add ARIA labels** - For interactive elements
- **Keyboard navigation** - Ensure all features are keyboard accessible

### 5. Error Handling

- **Handle missing data** - Always provide fallbacks
- **Validate settings** - Check for required settings
- **Show error states** - Display user-friendly error messages

---

## Case Study: Instagram Feed Profile Widget

### Template Definition

```javascript
{
  id: "profile-widget",
  name: "Profile Widget",
  description: "Display Instagram profile with feed",
  isRecommended: true,
  previewImage: "/widgets/profile_widgets.png",
  options: [
    {
      id: "source",
      name: "Source",
      menus: [
        {
          id: "sourceTypeInput",
          type: "source-type-input",
          defaultSourceType: "account",
          defaultUsername: "",
        },
      ],
    },
    {
      id: "layout",
      name: "Layout",
      menus: [
        {
          id: "layoutType",
          type: "layout-selector",
          defaultValue: "slider",
          options: [
            { value: "slider", label: "Slider", icon: "slider" },
            { value: "grid", label: "Grid", icon: "grid" },
          ],
        },
        // ... more menus
      ],
    },
  ],
}
```

### Preview Component

```javascript
// preview/instagram-feed/ProfileWidgetPreview.js
function ProfileWidgetPreview({ settings, viewMode, posts, profile }) {
  const layoutType = settings?.layoutType || "slider";
  const columns = getColumnsForViewMode(viewMode, settings, measuredWidth);
  const rows = getRows(settings, viewMode, measuredWidth);
  
  return (
    <div className="widget-preview-wrapper">
      <div className="widget-preview-container">
        <InstagramHeader settings={settings} />
        {showProfileHeader && (
          <InstagramProfile settings={settings} profile={profile} />
        )}
        {layoutType === "grid" ? (
          <InstagramGrid columns={columns} rows={rows} settings={settings} posts={posts} />
        ) : (
          <InstagramSlider columns={columns} rows={rows} settings={settings} posts={posts} />
        )}
      </div>
    </div>
  );
}
```

### Helper Functions

```javascript
// preview/instagram-feed/utils/helpers.js
export const getColumnsForViewMode = (viewMode, settings, containerWidth) => {
  const columnsMode = settings?.columnsMode;
  
  if (columnsMode === "manual") {
    return Number(settings?.columns) || 4;
  } else {
    // Auto mode: calculate based on container width
    return calculateAutoColumns(containerWidth);
  }
};
```

### Backend Integration

```javascript
// backend/api/routes/widgets/index.js
async function fetchInstagramWidgetData(widgetId) {
  // 1. Get widget config (access token)
  const config = await getWidgetConfigByWidgetId(widgetId, "instagram");
  
  // 2. Fetch Instagram data
  const profile = await fetchInstagramProfile(config.accessToken);
  const posts = await fetchInstagramPosts(config.accessToken);
  
  // 3. Save to wigets_data table
  await saveWidgetData(widgetId, { profile, posts }, "instagram");
  
  return { profile, posts };
}
```

---

## Summary Checklist

When creating a new widget:

- [ ] Define widget type in `widget-templates.js`
- [ ] Create template(s) with options and menus
- [ ] Create preview component(s)
- [ ] Register preview in `preview/index.js`
- [ ] Create front component router (`front.js`)
- [ ] Add styles (`index.scss`)
- [ ] Create helper functions if needed
- [ ] Add backend endpoints if needed
- [ ] Test in editor
- [ ] Test embed code
- [ ] Document any custom field types

---

**Last Updated**: January 2025  
**Maintainer**: Development Team


---

## 1. Directory Structure

Create a new directory for your widget in `app/src/pages/widgets/preview/[widget-name]`. A standard widget directory consists of:

- `settings.js`: Defines the widget's configuration options (inputs seen in the editor).
- `front.js`: The entry point for the widget, handling lazy loading specific to the preview environment.
- `[WidgetName]Preview.js`: The main React component containing the widget's logic and UI.
- `index.scss`: (Optional) SCSS file for styling.
- `components/`: (Optional) Directory for sub-components.

---

## 2. Running the Environment

Before development, ensure the environment is running. The project uses npm scripts to start the necessary services.

```bash
npm start
```

This will run the dashboard on http://localhost:3707.

---

## 3. Define Settings (`settings.js`)

This file exports a configuration object that determines how the widget appears in the dashboard and what options are available to the user.

**Example Structure:**

```js
const MyWidgetSettings = {
    id: "my-widget",
    name: "My Widget",
    description: "Description of my widget",
    icon: "/path/to/icon.png",
    templates: [
        {
            id: "default",
            name: "Default Template",
            description: "Default layout",
            options: [
                {
                    id: "content",
                    name: "Content",
                    menus: [
                        {
                            id: "title",
                            label: "Title",
                            type: "text", // Types: text, image, color, toggle, repeater, etc.
                            defaultValue: "Hello World"
                        }
                    ]
                }
            ]
        }
    ]
};
export default MyWidgetSettings;
```

---

## 4. Create Entry Point (`front.js`)

This file acts as the bridge between the lazy-loading mechanism and your actual component. It typically exports the main preview component.

**Example:**

```js
import React from "react";
import MyWidgetPreview from "./MyWidgetPreview";
const Front = (props) => {
  return <MyWidgetPreview {...props} />;
};
export default Front;
```

---

## 5. Implement Component Logic (`[WidgetName]Preview.js`)

This is where the actual code for your widget lives. The settings defined in step 2 are passed down as props.

**Example:**

```js
import React from "react";
import "./index.scss"; // Import styles
const MyWidgetPreview = ({ settings }) => {
  // Access values defined in settings.js
  const title = settings.title || "Default Title";
  return (
    <div className="my-widget-container">
      <h1>{title}</h1>
    </div>
  );
};
export default MyWidgetPreview;
```

---

## 6. Register the Widget

To make the widget available in the application, you need to register it in two places.

### A. Register Loader in `front-index.js`

Add the lazy load definition to `app/src/pages/widgets/preview/front-index.js`.

```js
const MyWidgetLoadable = React.lazy(() =>
  import(
    /* webpackChunkName: "widget-my-widget" */
    /* webpackMode: "lazy" */
    "./my-widget/front"
  ),
);
// Add to WIDGETS_LOADABLES object
const WIDGETS_LOADABLES = {
  // ... other widgets
  "my-widget": {
    widget: MyWidgetLoadable,
  },
};
```

### B. Register Settings in `widget-templates.js`

Add the settings import to `app/src/pages/widgets/data/widget-templates.js`.

```js
import MyWidgetSettings from "../preview/my-widget/settings";
export const WIDGET_TEMPLATES = {
  // ... other widgets
  "my-widget": MyWidgetSettings,
};
```

---

## 7. Development Tips

- **Unique IDs:** Ensure your widget ID and template IDs are unique.
- **Lazy Loading:** The `front.js` file is crucial for code splitting; do not import the heavy component directly in `front-index.js`.
- **CSS Scoping:** Use specific class names or CSS modules to avoid styles leaking into other parts of the application.

---

## 8. Version Control (Git)

Follow these steps to manage your widget development using Git:

### A. Initialize/Manage Branch

- **Check Status:** See any changes you've made.

```bash
git status
```

- **Create New Branch:** Always create a fresh branch for a new widget or feature.

```bash
git checkout -b feature/new-widget-name
```

### B. Stage and Commit

- **Stage Changes:** Add your new and modified files.

```bash
git add .
```

- **Commit Changes:** Save your snapshot with a descriptive message.

```bash
git commit -m "feat: implement new widget structure"
```

### C. Push and Share

- **Push to Remote:** Upload your branch to the repository.

```bash
git push origin feature/new-widget-name
```
