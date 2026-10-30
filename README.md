# Pawan Suthar — Portfolio

A single-page developer portfolio with no navbar. Navigation lives in a floating **Menu** pill, and the signature feature is an **interactive canvas**: a full-screen workspace where certificates, achievements and project snapshots float as draggable objects.

Built with React + Vite, Tailwind CSS v4, Framer Motion and Lucide icons.

---

## Getting started

Requires Node.js 18 or newer.

```bash
npm install
npm run dev        # http://localhost:5173
npm run build      # production build in /dist
npm run preview    # serve the production build locally
```

---

## Project structure

```text
public/
  assets/
    certificates/     ← your certificate images (PNG/JPG/SVG/PDF)
    achievements/     ← photos for achievements / hackathons
    projects/         ← project screenshots
  favicon.svg
src/
  data/
    profile.js        ← name, intro, links (email / GitHub / LinkedIn)
    skills.js         ← skill groups
    projects.js       ← featured projects
    experience.js     ← roles, education, hackathons, internships
  canvas/
    canvasData.js     ← every object on the canvas  ★
    CanvasView.jsx    ← workspace: pan, zoom, pinch, toolbar, keyboard
    CanvasObject.jsx  ← one draggable card (5 visual types)
    CertificateModal.jsx ← large preview / lightbox
  hooks/
    useCanvasPositions.js ← positions, stacking order, localStorage
    usePanZoom.js         ← camera maths (zoom to cursor, fit, recenter)
    useActiveSection.js   ← highlights the current section in the menu
    useFocusTrap.js       ← keeps keyboard focus inside overlays
  components/         ← Button, FloatingMenu, SectionHeading, SocialLinks, ProjectCard…
  sections/           ← Hero, About, Skills, Projects, Experience, Testimonials, Contact
  styles/global.css   ← design tokens (colors, fonts) + base styles
  App.jsx
  main.jsx
```

Images live in `public/assets` rather than `src/assets`, so adding a certificate is "drop a file + add one object". No imports are needed. Paths are written without a leading slash, for example `'assets/certificates/aws.png'`, and are resolved for you. This also works if you deploy under a sub-path.

---

## Editing your content

All content lives in `src/data/` and `src/canvas/canvasData.js`. You never need to touch component code.

> **Tip:** while `npm run dev` is running, every placeholder shows a small yellow **TODO** or **sample** flag on the page. Production builds never show these flags.

### Profile and links — `src/data/profile.js`

Replace `email`, `github` and `linkedin`. You can also edit `tagline`, `intro`, `location` and `status`.

### Projects — `src/data/projects.js`

Each project is one object. Add, remove or reorder freely.

```js
{
  id: 'my-app',
  title: 'My App',
  category: 'Productivity',
  description: 'One or two sentences.',
  stack: ['React', 'Node.js'],
  image: 'assets/projects/my-app.png', // put the file in public/assets/projects/
  accent: '#36c5f0',                    // card tint
  github: 'https://github.com/…',       // '' hides the button
  live: 'https://…',                    // '' hides the button
  canvasId: 'snap-my-app',              // optional: adds a "See on canvas" button
}
```

### Experience — `src/data/experience.js`

Entries marked `sample: true` are layout templates. They show in development and are **hidden in production** until you fill them in and delete the `sample` line. Empty fields (`period`, `org`, `location`…) are not displayed.

Optional fields: `achievementsTitle` renames the bullet list (default "Key Achievements"), and `cover` puts an image behind the card's header, for example `'assets/experience/artificial-arcade.jpg'`.

**How the row scrolls:** on desktop (at least 1024 px wide and 720 px tall) the section pins in place and scrolling down slides the cards sideways, smoothed with a spring. On phones, tablets, short screens, or with reduced motion turned on, it's a normal swipeable row with arrow buttons instead.

---

## Adding a certificate to the canvas

1. Save the image as `public/assets/certificates/my-cert.png`. A landscape image works best, around 1200×850.
2. Open `src/canvas/canvasData.js` and add an object to `canvasItems`:

```js
{
  id: 'cert-aws',                       // unique
  category: 'certificate',
  title: 'AWS Cloud Practitioner',
  issuer: 'Amazon Web Services',
  date: 'Mar 2026',
  description: 'What it covered, in one line.',
  image: 'assets/certificates/my-cert.png',
  file: 'assets/certificates/my-cert.pdf', // optional, for the "Open original" button
  initialX: 200,
  initialY: 900,
  width: 340,
  height: 300,
  rotation: -3,                         // optional tilt, keep it between -6 and 6
},
```

3. Save. The certificate appears on the canvas, opens in the preview, and is draggable.

To **replace the samples**, swap the files in `public/assets/certificates/` and edit `cert-1` to `cert-4`. Delete `sample: true` from each one you make real. To **remove** an item, delete its object.

### Canvas item fields

| Field | Required | Notes |
| --- | --- | --- |
| `id` | yes | Unique. Also the key for saved positions. |
| `category` | yes | `certificate`, `achievement`, `project`, `note` or `links` |
| `title` | yes | |
| `image` | for certificates and projects | Path in `/public` or a full `https://` URL |
| `file` | no | Original PDF or link for the **Open original** button |
| `description`, `issuer`, `date` | no | Shown on the card and in the preview |
| `initialX`, `initialY` | yes | Top-left corner in canvas pixels at 100% zoom |
| `width`, `height` | yes | Card size in canvas pixels |
| `rotation` | no | Degrees |
| `color` | no | Background for achievements and notes |
| `link` | no | Projects only. Shown as **Visit project** in the preview. |
| `sample` | no | Marks placeholder content |

Category looks:

- **Certificate:** white card with the image, title and issuer.
- **Achievement:** colored sticker card with an optional photo.
- **Project:** browser-window frame around a screenshot.
- **Note:** handwritten sticky note.
- **Links:** dark card with your GitHub, LinkedIn and email.

Certificates, achievements with an image, and projects open the large preview when clicked.

---

## Canvas behavior

| Action | Mouse / trackpad | Touch | Keyboard |
| --- | --- | --- | --- |
| Move an object | drag it | drag it | Tab to it, then arrow keys (Shift for bigger steps) |
| Preview | click | tap | Enter |
| Pan | drag empty space, or scroll | drag empty space | |
| Zoom | Ctrl/⌘ + scroll, or pinch | pinch | `+` / `-` |
| Fit everything | **Fit all** button | | `0` |
| Close | **Close** button | | `Esc` (closes preview, then deselects, then closes canvas) |

- **Recenter** returns to 100% zoom centered on your items. **Reset** restores every object to its `initialX` and `initialY` and clears the saved layout.
- **Persistence:** positions and stacking order are saved to `localStorage`, so they survive a refresh. If you change an item's `initialX` or `initialY` in the config, that item's old saved position is ignored automatically. To turn saving off, set `PERSIST_CANVAS_LAYOUT = false` in `src/App.jsx`.
- On phones, the canvas opens zoomed in on the first note instead of fitting everything, so text stays readable. **Fit all** shows the whole space.
- The page cannot scroll while the canvas is open, and every gesture uses `touch-action: none`, so dragging never scrolls the page.
- The canvas has its own URL, `/#canvas`. The browser back button closes it.

---

## Contact form

There is no backend. After validating the form, it opens the visitor's email app with a pre-filled message to `profile.email`. To use a form service such as Formspree or Web3Forms instead, replace `handleSubmit` in `src/sections/Contact.jsx`.

---

## Design notes

The visual language follows the reference site:

- White canvas and near-black ink (`#111212`).
- Big, tight Inter headings.
- *Titan One* only for the hero name and the giant footer wordmark (whose letters float and zoom on hover).
- DM Mono labels.
- Handwritten accents in *Just Me Again Down Here*.
- Rotated sticker shapes in a small palette of blue, yellow, mint, lilac and coral.

Colors and fonts are tokens in `src/styles/global.css` under `@theme`. Animations respect the visitor's reduced-motion setting.

---

## Details still to fill in

- [ ] Your photo for the About section: put it in `public/assets/profile/` and set `about.photo` in `profile.js`
- [ ] Hero learning-streak heatmap, in `src/data/activity.js`:
  - A higher-resolution hero photo. The current one is cropped from a screenshot.
  - Your real day-by-day activity. Until then, the squares are a sample matching 104 active days and a 42-day longest streak; "current streak" and "contributions" appear once real data is in.
- [ ] Email address, in `profile.js`
- [ ] GitHub profile URL, in `profile.js`
- [ ] LinkedIn profile URL, in `profile.js`
- [ ] Location, if you want more than "India", in `profile.js`
- [ ] Status line such as "Open to internships…", in `profile.js`
- [ ] Each project's real tech stack, GitHub link, live link and screenshot, in `projects.js`
- [ ] CSI ACE Treasurer dates (`period`), in `experience.js`
- [ ] Internships, replacing the `sample-internship` entry in `experience.js`
- [ ] Real testimonials, replacing the `sample` entries in `src/data/testimonials.js`
- [ ] Real certificate images and details, replacing `cert-1` to `cert-4` in `canvasData.js`
- [ ] Real achievements, replacing `ach-hackathon` and `ach-award` in `canvasData.js`
