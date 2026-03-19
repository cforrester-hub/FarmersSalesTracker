# Farmers Sales Tracker

## Project Overview
A single-page web dashboard for tracking insurance policy sales progress toward time-bound goals. Used by a farmers insurance sales team to monitor performance, visualize daily trends, and filter by carrier, producer, lead source, and policy type.

## Architecture Rules
- **Single-file app** — all HTML, CSS, and JavaScript lives in `index.html`. Do not split into separate files.
- **No frameworks or libraries** — vanilla HTML/JS/CSS only. No React, no jQuery, no bundlers.
- **No build tools** — no npm, webpack, or compilation. The HTML file is served directly.
- **No backend** — everything is client-side. No server, no API calls, no database.
- **No data persistence** — data is loaded fresh from CSV each session (no localStorage).

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
