# Widget Development Process

This document outlines the step-by-step process for developing a new widget in the Poper application.

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