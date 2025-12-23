# widget-preparation

## 🧩 Widget Development Process

This document outlines the **step-by-step process for developing a new widget** in the **Poper application**.

---

## 📁 Directory Structure

Create a new directory for your widget at:

```
app/src/pages/widgets/preview/[widget-name]/
```

**Standard Widget Directory:**

```
[widget-name]/
├── settings.js
├── front.js
├── [WidgetName]Preview.js
├── index.scss        # optional
└── components/       # optional
```

**File Descriptions:**

- `settings.js`: Defines the widget's configuration options (inputs shown in the editor).
- `front.js`: Entry point for the widget. Handles lazy loading in the preview environment.
- `[WidgetName]Preview.js`: Main React component containing widget logic and UI.
- `index.scss` (optional): SCSS file for widget-specific styling.
- `components/` (optional): Directory for reusable sub-components.

---

## ⚙️ Define Settings (`settings.js`)

This file exports a configuration object that determines:
- How the widget appears in the dashboard
- What options are available to the user in the editor

**Example:**

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
              type: "text", // text, image, color, toggle, repeater, etc.
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

## 🚪 Create Entry Point (`front.js`)

This file acts as the bridge between the lazy-loading mechanism and the preview component.

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

## 🧠 Implement Component Logic (`[WidgetName]Preview.js`)

This is where the actual widget logic and UI are implemented. Settings defined in `settings.js` are passed as props.

**Example:**

```js
import React from "react";
import "./index.scss";

const MyWidgetPreview = ({ settings }) => {
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

## 🧾 Register the Widget

To make the widget available in the application, it must be registered in two places:

### 1. Register Loader (`front-index.js`)

**Path:**

```
app/src/pages/widgets/preview/front-index.js
```

**Example:**

```js
const MyWidgetLoadable = React.lazy(() =>
  import(
    /* webpackChunkName: "widget-my-widget" */
    /* webpackMode: "lazy" */
    "./my-widget/front"
  )
);

const WIDGETS_LOADABLES = {
  "my-widget": {
    widget: MyWidgetLoadable,
  },
};
```

### 2. Register Settings (`widget-templates.js`)

**Path:**

```
app/src/pages/widgets/data/widget-templates.js
```

**Example:**

```js
import MyWidgetSettings from "../preview/my-widget/settings";

export const WIDGET_TEMPLATES = {
  "my-widget": MyWidgetSettings,
};
```

---

## 💡 Development Tips

- **Unique IDs:** Ensure widget IDs and template IDs are globally unique.
- **Lazy Loading:** The `front.js` file is crucial for code splitting. ❌ Do not import heavy components directly in `front-index.js`.
- **CSS Scoping:** Use specific class names or CSS modules to avoid style leakage.

---

## 🔁 Version Control (Git)

**Create a New Branch:**

```bash
git checkout -b feature/new-widget-name
```

**Stage and Commit Changes:**

```bash
git add .
git commit -m "feat: implement new widget structure"
```

**Push to Remote:**

```bash
git push origin feature/new-widget-name
```