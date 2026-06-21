# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-page React landing/marketing site template, bootstrapped with Create React App (`react-scripts`). All visible content (headlines, services, gallery, testimonials, team, contact info) is driven by one JSON file rather than hardcoded in components.

## Commands

This project uses npm/yarn interchangeably (both lockfiles are present).

- `npm start` / `yarn start` — run the CRA dev server (http://localhost:3000) with hot reload
- `npm run build` / `yarn build` — production build to `build/`
- `npm test` / `yarn test` — run CRA's Jest test runner in watch mode (no test files currently exist in the repo; `src/setupTests.js` wires up `@testing-library/jest-dom`)
- `npm run eject` — irreversible CRA eject; do not run unless explicitly requested

There is no separate lint script; ESLint runs via the `eslintConfig: { extends: "react-app" }` block in `package.json` as part of the CRA build/start pipeline.

## Architecture

**Data flow:** `src/App.jsx` loads `src/data/data.json` into state on mount (`useEffect` + `useState`, not props/context) and passes each top-level JSON key down as a `data` prop to one section component. Every section component follows the same defensive pattern: `props.data ? props.data.field : "Loading"` (or `"Loading..."`), since data arrives asynchronously even though it's a static import.

**Section components** (`src/components/*.jsx`) each own one full-width semantic page section: `<header id="header">`, `<nav id="menu">`, `<section id="...">` for `features`/`about`/`services`/`portfolio`/`testimonials`/`team`/`contact`, and `<footer id="footer">`. Section IDs are the anchors used by in-page nav links (`#features`, `#about`, etc. in `navigation.jsx`) and smooth-scrolled via the `smooth-scroll` library instantiated once in `App.jsx` (`export const scroll = new SmoothScroll(...)`). When adding a section, use `<section id="...">` rather than a bare `<div id="...">` to keep this convention.

**To add/change site content** (text, services, gallery images, team members, contact details): edit `src/data/data.json` only — do not hardcode copy into components. Image paths in the JSON are relative to `public/` (e.g. `img/portfolio/01-large.jpg` → `public/img/portfolio/01-large.jpg`).

**To add a new page section:** create a component in `src/components/`, import and render it in `src/App.jsx` with `data={landingPageData.YourKey}`, add the matching key to `data/data.json`, a matching `public/css/style/<section>.css` file linked from `index.html`, and a nav anchor in `navigation.jsx` if it should appear in the menu.

**Styling** is plain CSS/Bootstrap, not CSS Modules or styled-components. `public/css/style/` holds one file per concern, each linked individually (in cascade order) from `public/index.html`: `tokens.css` (the `:root` design-token color palette), `base.css` (typography/reset), `layout.css` (shared `.section-title` utility), `buttons.css`, `navigation.css`, then one file per page section (`header.css`, `features.css`, `about.css`, `services.css`, `portfolio.css`, `testimonials.css`, `team.css`, `contact.css`, `footer.css`) mirroring the `src/components/*.jsx` split 1:1. `public/css/bootstrap.css` is loaded separately, plus `src/App.css`/`src/index.css` for component-local tweaks. Bootstrap 3 grid classes (`col-md-*`, `row`, `container`) are used throughout JSX. All section colors reference the tokens in `tokens.css` — don't hardcode new hex colors.

**Contact form:** `src/components/contact.jsx` is presentation-only; form state and the EmailJS submit logic live in `src/hooks/useContactForm.js` (`emailjs-com`). The Service ID, Template ID, and Public Key are placeholder strings (`"YOUR_SERVICE_ID"` etc.) — this is intentional template behavior, not a bug to "fix" silently; only replace them if the user provides real EmailJS credentials. The site footer is its own component, `src/components/footer.jsx`, rendered directly in `App.jsx` (not nested inside `Contact`).

## Deployment

`.github/workflows/deploy.yml` deploys to GitHub Pages on every push/PR to `master` via `tanwanimohit/deploy-react-to-ghpages@v1.0.1`. `_config.yml` (`jekyll-theme-minimal`) is a leftover GitHub Pages config and not part of the React build.
