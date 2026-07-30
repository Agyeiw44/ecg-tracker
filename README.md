# ECG Outage Tracker

A product development case study, built in response to something I was already living through.

> **Note:** This tool reads ECG's publicly published load management timetable. It is not affiliated with the Electricity Company of Ghana, does not have inside information, and cannot influence when your power comes back on.

**Role:** Sole owner: design, data architecture, and build.
**Status:** Built and live at [ecg-tracker.vercel.app](https://ecg-tracker.vercel.app/).

---

## Overview

Ghana's load shedding — "dumsor" — isn't new. What's new is that ECG now publishes structured timetables: areas assigned to load groups A, B, or C, each rotating through six-hour on/off slots across the week. The information exists. The problem is that it exists as a PDF, or a social media screenshot, or a printed grid that assumes you already know which group your neighbourhood belongs to.

The ECG Outage Tracker cuts that gap. It's a single-page web tool: type your area, and you get your current power status — on or off, and how long until it changes — plus your full day's schedule, pulled from a live Supabase backend with a hardcoded fallback if connectivity drops.

The product has a secondary design goal beyond utility: it should feel appropriate to the context. "Dumsor Mode" — an OLED-black theme — isn't a novelty feature. It's what you'd actually want when the power's out and you're checking your phone's remaining battery to see when the lights come back.

---

## Tools & Key Decisions (at a glance)

**Built with:**
- **HTML/CSS/JS** — single file, no framework, no build step
- **Supabase** — live schedule and neighbourhood data
- **Google Fonts** — Space Mono (monospace) + Syne (display)

**Architecture and design decisions:**

| # | Decision | Why (the guardrail) |
|---|----------|----------------------|
| 1 | **Single-file, no-framework architecture:** the entire app ships as one `index.html` with no build pipeline, no dependencies to install, and nothing to update. | Removes the failure modes that come with complex toolchains. The tool needs to work in a low-bandwidth, low-trust environment. A file that opens is better than one that might not build. |
| 2 | **Tiered data architecture with hardcoded fallback:** Supabase is the primary source, but a hardcoded timetable snapshot serves if the fetch fails. | The time when someone is most likely to check this tool is also the time connectivity is most likely to be spotty. A fallback that degrades gracefully is not optional here — it's core to the brief. |
| 3 | **Load group logic, not raw addresses:** areas are mapped to ECG load groups (A, B, C). The schedule is computed from the group and the timetable — not stored per area. | ECG's own structure is group-based. Mirroring that means a schedule update requires changing one timetable table, not thousands of individual records. It also models the problem correctly: the same outage applies to everyone in a group. |
| 4 | **Dumsor Mode as a functional feature, not just a theme toggle:** the dark theme uses OLED true black (`#000000` background) to minimise pixel draw on the most common screen type on low-to-mid-range phones in Ghana. | Battery conservation matters here. A theme built for aesthetics would use dark grey. This one uses true black because the primary use case is someone checking their schedule on a draining phone at night. |
| 5 | **Visible ambiguity over silent guessing:** the search layer surfaces cases where an area name maps to multiple load groups rather than picking one automatically. | Silently picking the wrong group produces an incorrect schedule. A user who sees "we found two matches" can resolve it themselves; a user who gets the wrong schedule can't know to question it. |

---

## Case Study

### Challenge

ECG's 800MW load management timetable is publicly available. In practice, it's not particularly usable: the official format assumes you already know your area's load group, assumes you can read a grid-format PDF in the dark, and doesn't account for the fact that area names on the official list often differ from what residents actually call their neighbourhoods.

Three constraints pulled against each other:

- Cover Ghana's major regions and ECG districts with enough neighbourhood-level detail to be genuinely useful — without hardcoding thousands of individual outage windows that would expire the moment ECG updated the schedule.
- Build something that works in degraded conditions: low battery, low connectivity, possibly no connectivity.
- Be honest about what the tool can and cannot do. It reads ECG's published schedules. It doesn't have inside information, it isn't affiliated with ECG, and if the lights are on when they shouldn't be — or off when they should be on — that's an ECG matter, not an app bug.

### Process

The data architecture was the first problem to solve, because it determined everything else. ECG structures load shedding by groups, not individual addresses — so modelling the problem at group level was the correct decision architecturally, not just a simplification. That produced a two-tier structure: static district lists reflecting ECG's published geography, and a dynamic scheduled outages table where Supabase supplies live data and a hardcoded snapshot handles the fallback.

The neighbourhood search layer required the most manual work: consolidating thousands of area names from ECG timetable PDFs, normalising inconsistent spellings, and mapping each to the right region and load group. The data quality problem wasn't incidental — it was the product problem.

Design followed the context. The app is built for a phone, likely used at night with the power out. That meant mobile-first layout with bottom navigation, an OLED-black Dumsor Mode, and a share layer via WhatsApp and X — the channels Ghanaians already use to circulate this kind of information.

### Solution

The shipped product is a four-tab, single-session app: Home (search and current status), Favourites (starred areas for quick return), Map (coming), Feed (coming). The Home tab shows current on/off status, time until the next change, and a full day's schedule in six-hour slots. Favourites persist in `localStorage` so returning users can get to their area without re-searching.

The schedule logic takes the user's area → load group → today's timetable entry → current time and computes status dynamically. No schedule data is stored per user. The Supabase tables hold outage schedules and neighbourhood mappings; everything else is client-side.

The disclaimer in the footer says what it needs to: *"Not an ECG product. Created by someone who is also currently in the dark and just wants to know when they can use their fan."*

### Impact

The tool is built and live at [ecg-tracker.vercel.app](https://ecg-tracker.vercel.app/). It hasn't been widely distributed, so there's no usage data to report. I'd rather be clear about that than manufacture a metric.

The strongest design signal came from the brief itself: the tool works because the core premise is true. The data existed. The need was real. The usability gap between the two was large. That gap is now closed.

### Reflection

The data normalisation work was the unglamorous core of this project. The temptation with tools like this is to focus on the interface and assume the data layer will sort itself out. It won't. ECG's timetables have real inconsistencies — areas listed under multiple groups, neighbourhood names that don't match common usage, regions appearing in some timetables but not others. Getting that layer right was a longer job than building the UI.

The other useful tension was between completeness and honesty. An earlier version attempted to show "likely on / likely off" for hours where the schedule was ambiguous. I cut it. The tool should say what it knows and not guess what it doesn't. If an area search returns ambiguous results, showing that ambiguity is the correct call — even if it's a less polished experience than resolving it silently and wrongly.

Next steps: the Map tab (load group distribution across Ghana visualised on a map) and Feed tab (ECG announcement aggregation). Both are in the navigation but not yet built. I'd rather ship a working two-tab tool than a four-tab one where two tabs show placeholder text indefinitely.

---

## Let's Talk

Open to discussing the project, the data architecture, or collaboration. Reach me on [LinkedIn](https://www.linkedin.com/in/jay-pepera/).
