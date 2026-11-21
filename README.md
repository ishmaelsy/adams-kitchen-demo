# Adams Kitchen – ordering website (demo)

A single-page, mobile-first ordering site for **Adams Kitchen** (KNUST, Ayeduase Gate branch, plus Tanoso). Customers browse the menu, choose sizes and options, build a cart, and the full order is turned into a WhatsApp message.

Built by SyberCodes as a **sales demo**. Prices, the WhatsApp number and the phone numbers in the footer are **placeholders** until confirmed with the business (see "Placeholders to confirm").

---

## 1. Tech and constraints

- **Plain HTML + CSS + JavaScript in one file** (`index.html`). No framework, no build step, no npm, no backend, no database.
- Only external requests: Google Fonts (**Fredoka** for headings, **Nunito** for body text).
- Works by opening `index.html` in a browser or by uploading the folder to any static host.
- **No payments.** Orders are sent to WhatsApp and the business collects payment itself (mobile money, cash on delivery).
- The site is `noindex` (demo). Remove `<meta name="robots" content="noindex">` when it goes live.

## 2. Folder structure

```
adams-kitchen/
├── index.html          # the whole site: markup, styles, data and logic
├── README.md           # this file
└── img/
    ├── logo.png        # round Adams Kitchen logo (transparent corners), used in header, favicon source, and as photo placeholder
    ├── favicon.png     # 64px version of the logo
    ├── spread.jpg      # hero background (branded bags, pizza, peppered chicken)
    ├── tray.jpg        # "Adams Tray" item + "Ordering for an event?" section
    ├── jollof.jpg      # Jollof Rice
    ├── jollof2.jpg     # Fried Rice
    ├── pizza.jpg       # Adams Delicious Pizza
    ├── chicken.jpg     # Peppered Chicken
    ├── shawarma.jpg    # Shawarma (white background cut-out)
    ├── wrap.jpg        # Wrap (white background cut-out, checkerboard removed)
    ├── burger.jpg      # Burger
    └── drinks.jpg      # Soft Drink
```

`index.html` must stay at the **root** of whatever folder is deployed (Vercel/Netlify serve `index.html` from the root; nested folders give a 404).

## 3. Page anatomy (top to bottom)

| Section | Selector | Notes |
|---|---|---|
| Demo bar | `.demo` | White strip saying this is a demo. **Delete for the live site.** |
| Header | `header.top` | Sticky navy bar: logo, name, branch line, **Order (n)** button (`#openCart`, count in `#count`). |
| Promo strip | `.promo` | Red "Adams Bogo Feast Promo – Buy one, get one free". Text comes from their Instagram flyer. |
| Hero | `.hero` | Full-width photo (`img/spread.jpg` set in CSS), headline, "See the menu" button linking to `#menu`. |
| Category tabs | `nav.tabs` / `#tabs` | Sticky pills. Generated from `MENU`. Active tab follows scrolling. |
| Menu | `#menu` | One `<section id="cat-N">` per category, a card grid inside. Generated from `MENU`. |
| Event section | `.about` | Photo + "Ordering for an event?" copy. |
| Footer | `footer` | Branches and phone numbers (**hard-coded text**, edit directly). |
| Sticky cart bar | `#bar` | Mobile only. Appears when the cart has items ("View order (n) · GH₵x"). |
| Item sheet | `#sheet` | Bottom sheet on mobile, centred panel on desktop. Opened by tapping a card. |
| Cart drawer | `#drawer` | Slide-in from the right. Lines, branch, delivery/pickup, name, location, note, promo tick-box, total, send button. |
| Demo modal | `#modal` | Shows the WhatsApp message that *would* be sent. |
| Toast | `#toast` | Small "X added" confirmation. |

The menu, tabs and item sheet are **rendered by JavaScript** from the `MENU` array. Do not hand-edit generated markup; change the data instead.

## 4. Brand design tokens

Defined as CSS variables at the top of the `<style>` block:

```css
--navy:#16263d;   /* header, hero overlay, footer, text */
--navy2:#0f1c2e;  /* darker navy */
--orange:#f59a23; /* primary buttons, active accents (from the logo circle) */
--red:#d7191f;    /* promo strip, prices, promo tick-box (from their flyers) */
--cream:#fff8ee;  /* page background */
--paper:#fffdf9;  /* cards, sheet, drawer */
--line:#efe2cc;   /* borders */
--mut:#6a6a6a;    /* secondary text */
```

Fonts: `Fredoka` (class `.f`) for headings/prices, `Nunito` for everything else. Shape language is rounded (pill buttons, 20px card radius) to echo the chef mascot logo.

## 5. Data model (the part you will edit most)

All menu content lives in the `MENU` array in the `<script>` block.

```js
const WA='233536654839';          // WhatsApp number, international format, no + or spaces
const MEAT=[['Chicken',0],['Beef',5],['Fish',8]];   // reusable option group: [label, extra cost]

const MENU=[
  { cat:'Rice', items:[
    { id:'jollof',                       // unique, lowercase, no spaces
      name:'Jollof Rice',
      desc:'Smoky jollof with your choice of protein.',
      img:'jollof',                      // file img/jollof.jpg (omit for no photo: shows the logo)
      cut:1,                             // optional: white-background cut-out photo (white card, image "contain")
      sizes:[['Regular',40],['Large',60]],        // optional: [label, base price]
      groups:[{n:'Protein', c:MEAT}]               // optional: option groups, each c = [[label, extra cost], ...]
    }
  ]}
];
```

Rules the code relies on:

- An item must have **either** `sizes` (price per size) **or** `price` (single price). `groups` are optional add-ons on top.
- **Price = size price (or `price`) + the extra cost of the chosen option in every group.** The first size and first choice in each group are selected by default.
- Items with `sizes` and/or `groups` open the item sheet. Items with neither add straight to the cart when the card is tapped, unless they have an `img` (then the sheet opens so the photo is visible).
- Card price label: `from GH₵x` when there are several sizes, otherwise the plain price.
- Items without `img` show the round logo on a peach block.
- Cart lines are merged by `id + chosen options`, so the same pizza twice becomes `x2`.
- Categories appear as tabs in array order.

### Adding or changing things

- **New item:** add an object to a category's `items`. Put its photo in `img/` and set `img:'filename-without-extension'`.
- **New category:** add `{cat:'Name', items:[...]}`. The tab and section are created automatically.
- **Change a price:** edit the number in `sizes` / `price` / the group extra.
- **Remove an item:** delete its object.

## 6. JavaScript structure

Everything is in one `<script>` at the bottom of `index.html`.

| Name | What it does |
|---|---|
| `WA`, `P`, `MEAT`, `MENU` | Config and data (see above). |
| `$`, `cur` | `document.querySelector` shortcut; formats money as `GH₵40`. |
| `all`, `byId` | Flat list of items and a lookup by `id`. |
| `minP(item)`, `hasOpts(item)` | Lowest price for the "from" label; whether the item has options. |
| State: `cart`, `otype`, `branch`, `cur_id`, `sel`, `lock` | Cart lines (`{k,name,detail,price,qty}`), delivery/pickup, selected branch, item currently in the sheet, its chosen options (`sel.g[]`, `sel.s`), and a time lock used while a tab click scrolls. |
| `calc(item, sel)` | Price for an item with the given selection. |
| `detail(item, sel)` | Text such as `Large, Mixed` shown in the cart and message. |
| `openSheet(id)` | Builds and opens the item sheet. |
| `closeAll()` | Closes sheet and drawer and hides the scrim. |
| `addLine(item, sel)` | Adds or merges a cart line, re-renders, shows a toast. |
| `render()` | Updates the count, totals, mobile cart bar and the cart lines list. |
| `setActive(k)` | Highlights a category tab and scrolls the tab row. |
| Click handler on `document` | One delegated handler for cards, sheet chips, tabs, delivery/pickup chips, branch chips and quantity buttons. |
| `#send` click handler | Validates the form, builds the WhatsApp message text, opens the demo modal. |
| Scroll listener | Updates the active tab based on which category is near the top (threshold 250px, ignored while `lock` is active). |

### The order message

Built in the `#send` handler. Example output:

```
Hello Adams Kitchen, I would like to place an order:

1. Adams Delicious Pizza (Large, Mixed) x1 = GH₵150
2. Bottled Water x1 = GH₵5

Total: GH₵155 (delivery fee not included)
Branch: KNUST (Ayeduase Gate)
Order type: Delivery
Name: Kwame
Delivery location: Unity Hall
Promo: Adams Bogo Feast Promo (buy one, get one free)
Note: Less pepper
```

Validation: name is required; delivery location is required only for Delivery; the send button is disabled while the cart is empty. The promo line only appears when the tick-box (`#bogo`) is checked.

## 7. Demo mode vs live mode

**Right now the site is in demo mode.** `#send` fills `#mmsg` and opens `#modal`; nothing is sent. This is deliberate so the business owner can try it safely and the wrong number can never receive orders.

To go **live**, in the `#send` click handler replace

```js
$('#mmsg').textContent=msg;closeAll();$('#modal').classList.add('on')
```

with

```js
window.open('https://wa.me/'+WA+'?text='+encodeURIComponent(msg),'_blank','noopener');closeAll();
```

Then also: remove the `.demo` bar, remove the `noindex` meta tag, delete the `#modal` markup and its handlers, and change the footer "Demo prepared by…" line.

**Per-branch numbers:** `WA` is one number for both branches. If each branch has its own WhatsApp number, make `const WA={'KNUST (Ayeduase Gate)':'233…','Tanoso':'233…'}` and use `WA[branch]` in the live `window.open` call.

## 8. Placeholders to confirm with the business

1. **All prices** in `MENU` are sample figures (GH₵). Replace with the real menu.
2. **Items** other than rice, pizza, chicken and the tray are inferred from their Instagram and shop sign (burgers, grills, wraps, meals, beverages). Confirm what they actually sell and add anything missing (for example more pizza toppings or meals).
3. **`WA` number** `233536654839` is taken from their event bag and flyer. Confirm which number takes KNUST orders.
4. **Phone numbers in the footer** (053 665 4839, 025 657 0803) come from their flyers. Also seen on packaging: 025 657 0803 / 020 320 4416 / 020 807 5606. Confirm which to show.
5. **Opening hours** are not shown because they are unknown. Add a line in the footer.
6. **Promo terms.** The banner and the tick-box only say "Buy one, get one free". Confirm what qualifies and for how long, then update the wording (`.promo`, `.chk`, and the message line).
7. **Delivery:** the site says the delivery fee is confirmed on WhatsApp. Add delivery areas and fees if they have fixed ones.
8. **Branch names** (`KNUST (Ayeduase Gate)`, `Tanoso`) are in the HTML (`#branch` chips and footer) **and** in the JS default `branch`.

## 9. Images

- The logo is a circular crop of their Instagram profile picture (`logo.png`, transparent corners).
- Food photos were cropped from the owner's Instagram screenshots (Instagram UI icons and bars removed) and lightly denoised and sharpened. They cannot be sharper than the source. **Replace them with original photos from the business when available** (keep the same filenames, or update `img:` in `MENU`).
- `shawarma.jpg` and `wrap.jpg` are stock-style cut-outs on white. `wrap.jpg` originally had a fake checkerboard baked in, which was replaced with white. Items using them have `cut:1`.
- `drinks.jpg` is a generic stock photo showing real soft-drink brands. Swap for the business's own drinks photo when possible.
- Keep photos at roughly 900–1400px wide and under ~250KB each for fast loading on mobile data.
- Logo and photos belong to Adams Kitchen. Do not publish the demo without their permission.

## 10. Responsive behaviour

- Designed mobile-first and tested at 320, 390, 768 and 1280px wide, with no horizontal scrolling.
- Menu grid: 2 columns on phones, 3 columns from 760px.
- Item sheet: slides up from the bottom on phones; a centred two-column panel from 860px.
- The mobile sticky cart bar (`#bar`) is hidden from 760px up, where the header button is enough.
- The cart drawer is full width on small screens, 440px wide otherwise.
- Header branch line is hidden below 380px to avoid wrapping.

## 11. Accessibility notes

- Cards, chips and buttons are real `<button>` elements with visible focus outlines.
- Cart, sheet and modal use `role="dialog"` and `aria-modal`. Escape closes them.
- Form errors are in an `aria-live`-style `role="alert"` paragraph (`#err`).
- Images have empty `alt` when decorative.
- Known gap: focus is not trapped inside open dialogs.

## 12. Testing

Done manually and with Playwright (headless Chromium) at 320/390/1280px:

- No horizontal overflow and no JS errors on load.
- Open the pizza sheet, pick Large + Mixed, expect `GH₵150`.
- Add items, check the count and totals, change quantities.
- Tabs highlight the right category after a click and while scrolling.
- Send with no name → "Please enter your name." Fill name and location → modal shows the correct message, including the promo line when ticked.

If you change the menu structure, re-run these checks.

## 13. Deploying

- Any static host works: upload the folder with `index.html` at the root.
- **Check the host's terms for commercial use.** Vercel's free (Hobby) plan is for non-commercial use, so a client's live business site needs a paid plan or a different host (Netlify, Cloudflare Pages, GitHub Pages and similar have their own terms; verify before using).
- A custom domain (e.g. `adamskitchen.com`) is optional, bought separately and should be registered in the client's name.

## 14. Ideas for later

- Per-branch WhatsApp numbers and opening hours.
- Remember the customer's name and location in `localStorage` for repeat orders.
- Item-level notes ("no onions").
- Delivery fee by hostel/area.
- A simple admin page or a Google Sheet feeding `MENU`, so staff can change prices without editing code.
- Move CSS and JS to separate files if the site grows past one page.
