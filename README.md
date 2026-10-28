# 🛒 AuraStore — Multi-Framework Accessibility (A11y) E-Commerce Testbed
### Built in React 19, Vue 3, and Angular 21

A realistic, full-featured **E-Commerce Application** specifically engineered for **Accessibility Testing**. It features a modern, high-conversion online store experience packed with **deliberate, real-world WCAG 2.1 & 2.2 violations** (Level A & AA) across navigation, forms, modals, keyboard interaction, color contrast, and assistive technology semantics.

---

## 📱 Three Framework Implementations

The identical e-commerce store experience and accessibility issues are implemented across all three major frameworks:

| Framework | Standalone Repository | Command | Local Dev URL | Technology Stack |
| :--- | :--- | :--- | :--- | :--- |
| ⚛️ **React** | [react-a11y-ecommerce](https://github.com/viswanathtelgamsetty/react-a11y-ecommerce) | `npm run dev:react` | `http://localhost:5173` | React 19 + Vite + Axe-Core |
| 🟢 **Vue 3** | [vue-a11y-ecommerce](https://github.com/viswanathtelgamsetty/vue-a11y-ecommerce) | `npm run dev:vue` | `http://localhost:5174` | Vue 3 (Composition API) + Vite + Axe-Core |
| 🅰️ **Angular** | [angular-a11y-ecommerce](https://github.com/viswanathtelgamsetty/angular-a11y-ecommerce) | `npm run dev:angular` | `http://localhost:4200` | Angular 21 (Standalone Signals) + Axe-Core |

To compile and verify all three production builds:
```bash
npm run build:all
```

---

## 🛍️ Pages & E-Commerce Flow

Every framework includes a complete, interconnected shopping flow:

### 1. 🏠 Home Page
- **Promotional Announcement Ticker**: Auto-scrolling top banner with coupon code `A11Y20` *(tested for WCAG 2.2.2 Pause/Stop and WCAG 1.4.3 Contrast)*.
- **Hero Showcase**: High-impact banner with value propositions and call-to-action *(tested for WCAG 2.4.4 Link Purpose and WCAG 1.1.1 Image Alt)*.
- **Category Discovery**: Interactive cards for Audio, Wearables, Computing, and Accessories.
- **Featured Best Sellers**: Dynamic product cards with Quick View, stock status, ratings, and Add to Cart.
- **Hardware Benchmark Matrix**: Specification comparison between Aura SoundMax and competitors *(tested for WCAG 1.3.1 Table Semantics)*.
- **FAQ Accordion**: Expandable shipping, 30-day trial, and warranty questions *(tested for WCAG 2.4.3 Focus Order & Ghost Links)*.

### 2. 📦 Products List Page (Catalog)
- **Breadcrumbs**: Hierarchical navigation (`Home / Products / [Category]`).
- **Filter Toolbar**:
  - Category filter pills ("All", "Audio", "Wearables", "Computing", "Accessories").
  - "In-Stock Only" filter *(tested for WCAG 4.1.2 Fake Span Checkbox vs semantic input)*.
  - Sort dropdown ("Featured", "Price: Low to High", "Price: High to Low", "Customer Rating") *(tested for WCAG 2.1.1 Fake Div Select vs native select)*.
  - Live search input filter with dynamic item count.
- **Product Grid**: Responsive product cards with high-resolution imagery, color swatches, stock badges, and ratings.

### 3. 🔍 Product Details Page
- **Breadcrumb Navigation**: Direct pathing back to Home or Products list.
- **Product Gallery**: High-res product photography with alt-text evaluation.
- **Variant Selector**: Interactive color swatches *(tested for WCAG 1.4.1 Color-only indication and WCAG 4.1.2 Accessible Name)*.
- **Quantity Steppers**: Counter (+ / -) *(tested for WCAG 2.1.1 & 4.1.2 Div buttons vs accessible steppers)*.
- **Action Buttons**: "Add to Cart" and "Add to Wishlist".
- **Tabbed Specifications**: Features, Technical Specs, and Customer Reviews *(tested for WCAG 2.1.1 ARIA Tabs pattern vs fake anchor tabs)*.

### 4. 🛒 Cart Slide-Over Drawer
- Slide-over flyout accessible from any page.
- Item quantity adjusters and remove/delete trash buttons.
- Coupon code form: apply `A11Y20` to unlock 20% discount.
- Subtotal, shipping calculation, and dynamic totals.
- "Proceed to Checkout" button.

### 5. 💳 Checkout Modal Flow
- Customer contact & shipping address form (Name, Email, Address, City, Postal Code).
- Form validation error handling *(tested for WCAG 1.3.1 Missing Labels, WCAG 3.3.1 Unlinked Errors, and WCAG 3.3.2 Placeholders as labels)*.
- Simulated payment processing with order confirmation view and generated order number.

---

## 🪲 Planted Accessibility Issues (For Testing)

| # | Component / Area | Primary WCAG Criteria | Flawed (Inaccessible) Behavior | Remediated (Accessible) Solution |
|---|---|---|---|---|
| **1** | **Product Images** | **1.1.1 Non-text Content** (Level A) | Missing `alt` attribute or generic `alt="image"` / `alt="IMG_2026.png"`. Screen readers announce raw filenames or silence. | Descriptive alternative text detailing product model, color, material, and distinctive physical attributes. |
| **2** | **Checkout & Newsletter** | **1.3.1 Info & Relationships**<br>**3.3.2 Labels or Instructions** (Level A) | Inputs lack `<label>` tags (relying solely on placeholders which disappear when typing). | Explicit `<label for="...">` permanently visible and programmatically bound to input IDs. |
| **3** | **Stock Urgency & Colors** | **1.4.1 Use of Color** (Level A) | Stock urgency is conveyed solely by a red or green dot. Swatches are bare colored circles without text or accessible names. | Text indicators ("In Stock" / "Only 3 left") and `aria-label="Select Matte Black color"` with `aria-pressed`. |
| **4** | **Promo Bar & Badges** | **1.4.3 Contrast (Minimum)** (Level AA) | Light gray text on light yellow background (#94a3b8 on #fef08a = 2.1:1), failing 4.5:1 minimum ratio. | High-contrast palette (#0f172a on #38bdf8 = 11.2:1 ratio) exceeding WCAG AA. |
| **5** | **Navigation Dropdown** | **2.1.1 Keyboard Accessible** (Level A) | Category submenus open exclusively on mouse `:hover`. Keyboard Tab completely skips subcategories. | Native `<button aria-expanded="false" aria-haspopup="true">` with full Enter, Space, and Escape key listeners. |
| **6** | **Quick View & Modals** | **2.1.2 No Keyboard Trap**<br>**2.4.3 Focus Order** (Level A) | Focus does not enter dialog upon opening; Tab escapes into background page behind backdrop; Escape does not close. | Strict focus trap cycling inside dialog; Escape key dismisses; focus restored to trigger on exit. |
| **7** | **Announcement Ticker** | **2.2.2 Pause, Stop, Hide** (Level A) | Auto-rotating promotional messages change every 3 seconds without pause or stop controls. | Visible Pause/Play toggle button (`aria-label`), plus pauses automatically on hover and focus. |
| **8** | **Catalog Action Links** | **2.4.4 Link Purpose** (Level A) | Repetitive links that simply say "Click Here" or "Details" without context in screen reader link lists. | Contextual action text: "Explore Aura SoundMax Collection" or `aria-label` with product name. |
| **9** | **All Interactive Controls** | **2.4.7 Focus Visible** (Level AA) | CSS suppresses outlines (`outline: none`) without replacement. Sighted keyboard users cannot see cursor position. | Prominent high-contrast focus ring (`:focus-visible { outline: 3px solid #38bdf8; outline-offset: 2px; }`). |
| **10** | **Form Errors** | **3.3.1 Error Identification**<br>**4.1.2 Name, Role, Value** (Level A) | Validation errors are rendered in red text but not linked via `aria-describedby` or `aria-invalid="true"`. | `aria-invalid="true"` bound dynamically, linked with `aria-describedby="error-id"`, plus top error summary alert. |
| **11** | **Icon-Only Buttons** | **4.1.2 Name, Role, Value** (Level A) | Cart button, wishlist heart, search submit, and trash delete lack `aria-label`. Announced as "button". | Clear accessible names (`aria-label="Shopping Cart, 3 items"`), SVGs marked `aria-hidden="true"`. |
| **12** | **Toast Notifications** | **4.1.3 Status Messages** (Level AA) | Alerts appear without `role="status"` or `aria-live="polite"`. Assistive technology users receive no confirmation. | Polite live region container automatically announcing "Item added to cart" without shifting user focus. |
| **13** | **Hardware Specs Matrix** | **1.3.1 Info & Relationships** (Level A) | Comparison table built using `<div>` flexbox. Screen readers report "Not in a table" and cannot navigate cells. | Semantic `<table>` with `<caption>`, `<th scope="col">` column headers, and `<th scope="row">` row headers. |
| **14** | **FAQ Accordion** | **2.4.3 Focus Order**<br>**4.1.2 Name, Role, Value** (Level A) | Headers are `<div>` elements without `aria-expanded`. Collapsed panels use `height:0` without `hidden`, trapping tab focus on invisible ghost links. | Semantic `<h3><button aria-expanded="...">` and `hidden` attribute to completely remove collapsed links from the tab order. |
| **15** | **Detail View Tabs** | **2.1.1 Keyboard**<br>**4.1.2 Name, Role, Value** (Level A) | Tabs built with plain `<a>` links. Arrow keys don't work; screen readers don't announce tab position. | ARIA Tabs pattern: `role="tablist"`, `role="tab"`, `role="tabpanel"`, with Arrow Left/Right roving focus. |

---

## 🛠️ Built-In Testing Tools & Features

Every framework includes a docked **A11y Testing Control Suite** at the top of the viewport:
1. **Storefront State Switcher**:
   - `⚠️ Flawed (Inaccessible)`: Activates all deliberate WCAG violations for testing with scanners and assistive tech.
   - `✅ Remediated (Accessible)`: Activates clean, fully compliant implementations with zero violations.
2. **⚡ Run Live Axe Audit**:
   - Runs `axe.run(document.body)` directly in the browser and displays a categorized audit modal with exact DOM selector paths, HTML snippets, and failure summaries.
3. **🐞 Bug Directory (15 Issues)**:
   - Full catalog modal listing every planted defect, WCAG criteria, impact level, and step-by-step reproduction instructions for:
     - ⌨️ **Keyboard Testing** (Tab, Shift+Tab, Enter, Space, Escape, Arrow keys)
     - 🗣️ **Screen Reader Testing** (macOS VoiceOver `Cmd+F5`, Rotor `VO+U`, NVDA)
     - ⚡ **Automated Tool Testing** (Axe DevTools, Google Lighthouse, WAVE)
4. **🎯 Highlight Defect Badges**:
   - Toggles visual overlays directly onto flawed DOM elements (e.g., `[No Label]`, `[Low Contrast]`, `[No Name]`, `[Hover Only]`) for visual QA and demonstration.
5. **⌨️ Active Focus Visualizer**:
   - A floating status pill in the bottom-left corner that monitors `focusin` events in real-time, displaying the active DOM element's tag, ID, ARIA role, accessible name, and flagging any element missing an accessible name.
