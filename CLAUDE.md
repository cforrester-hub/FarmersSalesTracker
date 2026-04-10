# Farmers Sales Tracker

## Project Overview
A single-page web dashboard for tracking insurance policy sales progress toward time-bound goals. Used by a farmers insurance sales team to monitor performance, visualize daily trends, and filter by carrier, producer, lead source, and policy type.

## Architecture Rules
- **Single-file app** — all HTML, CSS, and JavaScript lives in `index.html`. Do not split into separate files.
- **No frameworks, no bundlers** — vanilla HTML/JS/CSS. Small utility libraries loaded via CDN `<script>` tags are allowed when they make a feature meaningfully better (e.g., `html2canvas`, `gif.js`). No React, no jQuery.
- **No build tools** — no npm, webpack, or compilation. The HTML file is served directly.
- **No backend** — everything is client-side. No server, no API calls, no database.
- **No backend persistence** — CSV data is loaded fresh each session. Configuration (goal, deadline, title) is persisted via sessionStorage.

## Code Conventions
- CSS uses custom properties (variables) for theming, defined in `:root`
- Dark theme with gold accents
- Google Fonts: Oswald (headings), Source Sans 3 (body)
- Responsive design with media queries for mobile

## Key Defaults
- Default carriers: Farmers and Foremost (pre-selected on data load)
- Goal, deadline, and title are user-configurable via the settings panel

## Data
- Input format: CSV with columns for Policy #, Carrier, Customer, Sold Date, Effective Date, Policy Type, Premium, Producer, Lead Source, Zip Code
- Sample data files are in `SampleFiles/`
