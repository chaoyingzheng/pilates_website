# Pilates Studio Website

A studio website built around a **group class schedule with booking**: members book, join waitlists and cancel; admins manage the schedule, rosters, members and booking rules.

- **Design & build plan:** [docs/DESIGN.md](docs/DESIGN.md)
- **Clickable prototype:** [prototype/index.html](prototype/index.html)

## Try the prototype

It's a single HTML file with no build step. Download or clone the repo and open `prototype/index.html` in a browser, or serve the folder:

```sh
python -m http.server 8137 --directory prototype
# then open http://localhost:8137
```

Inside the prototype:
- Use the **"View as"** menu (top right) to switch between Guest, Member (Mia Chen) and Admin.
- Demo data is saved in your browser only. Use **Reset demo data** in the top bar to start over.
- The **◐ Theme** button shows the theme presets that were considered. **Sand & Clay** was chosen.
- Emails aren't sent; they show up in the Admin dashboard's "Email outbox".

The prototype is for reviewing the flows and design only. It has no real login or server; the production app will be built with Next.js (see the plan).
