# 🛒 AuraStore — Multi-Framework Accessibility (A11y) E-Commerce Testbed
### Built in React 19, Vue 3, Angular 21, and Svelte 5

A realistic, full-featured **E-Commerce Application** engineered specifically for **Accessibility (a11y) Testing**. Each application is streamlined into **3 core components** (Header, ProductDetail, Footer) packed with **deliberate, real-world WCAG 2.1 & 2.2 violations** (Level A & AA) across navigation, forms, modals, keyboard interaction, color contrast, and screen reader semantics.

---

## 📱 Four Framework Repositories

Each framework is maintained as an independent, standalone repository under [`viswanathtelgamsetty`](https://github.com/viswanathtelgamsetty):

| Framework | Standalone Repository | Local Dev Command | Local Dev URL | Technology Stack |
| :--- | :--- | :--- | :--- | :--- |
| ⚛️ **React** | [react-a11y-ecommerce](https://github.com/viswanathtelgamsetty/react-a11y-ecommerce) | `npm run dev:react` | `http://localhost:5173` | React 19 + Vite |
| 🟢 **Vue 3** | [vue-a11y-ecommerce](https://github.com/viswanathtelgamsetty/vue-a11y-ecommerce) | `npm run dev:vue` | `http://localhost:5174` | Vue 3 (Composition API) + Vite |
| 🅰️ **Angular** | [angular-a11y-ecommerce](https://github.com/viswanathtelgamsetty/angular-a11y-ecommerce) | `npm run dev:angular` | `http://localhost:4200` | Angular 21 (Standalone Signals) |
| 🧡 **Svelte** | [svelte-a11y-ecommerce](https://github.com/viswanathtelgamsetty/svelte-a11y-ecommerce) | `npm run dev:svelte` | `http://localhost:5175` | Svelte 5 (Runes) + Vite |

To compile and verify all 4 production bundles at once:
```bash
npm run build:all
```

---

## 🧱 The 3 Core Components

Every framework implements the identical 3-component architecture:

### 1. `Header`
- **Auto-Scrolling Ticker**: Continuous marquee for promotional discount code `A11Y20` *(tested for WCAG 2.2.2 Pause/Stop and WCAG 1.4.3 Contrast)*.
- **Bypass Block**: Broken skip link target *(tested for WCAG 2.4.1)*.
- **Brand Logo**: Logo image using generic `alt="image"` *(tested for WCAG 1.1.1)*.
- **Search Bar**: Input lacking `<label>` and icon button missing accessible name *(tested for WCAG 1.3.1, 3.3.2, 4.1.2)*.
- **Navigation Menu**: Dropdown categories menu that opens strictly on mouse `:hover` and is skipped by keyboard Tab *(tested for WCAG 2.1.1)*.
- **Action Buttons**: Wishlist & Cart buttons with raw icon SVGs and numeric badges lacking `aria-label` *(tested for WCAG 1.1.1 & 4.1.2)*.
- **User Avatar**: `<div>` with click listener lacking `role="button"` and `tabindex="0"` *(tested for WCAG 4.1.2 & 2.1.1)*.

### 2. `ProductDetail`
- **Breadcrumbs**: Hierarchical path constructed using unsemantic `<div>` elements without `<nav aria-label="Breadcrumb">` *(tested for WCAG 1.3.1)*.
- **Product Gallery**: High-res product photography using `alt="image"` and thumbnails with missing `alt` attributes *(tested for WCAG 1.1.1)*.
- **Heading Hierarchy**: `<h1>` Product Title jumps straight to `<h4>` Tagline *(tested for WCAG 1.3.1)*.
- **Star Ratings**: Visual unicode characters `★★★★☆` without an accessible name *(tested for WCAG 1.1.1)*.
- **Stock Status**: Availability communicated solely by colored dots (`dot-green` / `dot-red`) with no text indicator *(tested for WCAG 1.4.1)*.
- **Color Swatches**: Custom circular `<div>` swatches with no text labels, no `role="radio"`, and no keyboard support *(tested for WCAG 1.4.1, 2.1.1, 4.1.2)*.
- **Quantity Stepper**: Custom `<div>` counter buttons (+ / -) not reachable or operable by keyboard *(tested for WCAG 2.1.1 & 4.1.2)*.
- **Add to Cart Notification**: Floating toast alert lacking `role="status"` or `aria-live="polite"` *(tested for WCAG 4.1.3)*.
- **Tabs Interface**: Overview, Hardware Specifications, and Customer Reviews constructed with plain `<a>` tags lacking `role="tablist"`, `role="tab"`, and arrow-key navigation *(tested for WCAG 4.1.2 & 2.1.1)*.
- **Ghost Tab Links**: Inactive tab panels hidden with opacity and absolute positioning, allowing keyboard Tab to enter hidden links *(tested for WCAG 2.4.3)*.
- **Specifications Table**: Benchmark comparison matrix built with unsemantic nested `<div>`s instead of semantic `<table>`, `<th>`, and `<td>` *(tested for WCAG 1.3.1)*.
- **Review Modal Dialog**:
  - Does not trap keyboard focus (Tab key escapes to the background page) *(tested for WCAG 2.1.2)*.
  - Cannot be closed with the `Escape` key *(tested for WCAG 2.1.1)*.
  - Lacks `role="dialog"`, `aria-modal="true"`, and `aria-labelledby` *(tested for WCAG 4.1.2)*.
  - Form fields lack `<label>` tags *(tested for WCAG 1.3.1 & 3.3.2)*.
  - Validation error indicated exclusively by a red border with no text message or `aria-invalid="true"` *(tested for WCAG 3.3.1 & 1.4.1)*.

### 3. `Footer`
- **Newsletter Subscription**: Email input lacking `<label>` and submit button with no accessible name *(tested for WCAG 1.3.1, 3.3.2, 4.1.2)*.
- **Low Contrast Copy**: Newsletter description, column subtext, and copyright notice fail WCAG AA 4.5:1 ratio *(tested for WCAG 1.4.3)*.
- **Social Media Links**: Empty anchor tags containing only SVG icons with no text or `aria-label` *(tested for WCAG 1.1.1 & 4.1.2)*.
- **Region/Currency Selector**: Custom dropdown opening only on hover *(tested for WCAG 2.1.1)*.
- **Ambiguous Links**: Repeated generic links ("Learn More", "Click Here", "Details", "Support", "Privacy", "Terms") *(tested for WCAG 2.4.4)*.

---

## 🧪 Accessibility Testing Checklist

Use these testbeds to evaluate:
- **Automated Scanners**:
  - [Axe DevTools Extension](https://www.deque.com/axe/devtools/)
  - Chrome Lighthouse Accessibility Audit
  - WAVE Web Accessibility Evaluation Tool
- **Screen Readers**:
  - Apple VoiceOver (macOS / iOS)
  - NVDA / JAWS (Windows)
  - Google TalkBack (Android)
- **Manual Keyboard Audit**:
  - Test keyboard `Tab`, `Shift+Tab`, `Enter`, `Space`, and `Esc` throughout all interactive flows.
