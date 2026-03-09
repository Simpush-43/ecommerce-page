# ECommerce-Page

A fast, fully configurable storefront built with **Next.js** and **Tailwind CSS**. Ships with static inventory out of the box — swap in any backend (Shopify, Contentful, custom API) by changing a single provider file.

**[Live Preview →](https://www.jamstackecommerce.dev/)**

---

## Screenshots

| Home | Category | Item | Cart |
|------|----------|------|------|
| ![Home](example-images/1.png) | ![Category](example-images/2.png) | ![Item](example-images/3.png) | ![Cart](example-images/4.png) |

---

## Getting Started

```sh
# 1. Clone
git clone https://github.com/Simpush-43/ecommerce-page.git
cd ecommerce-page

# 2. Install dependencies
npm install

# 3. Start dev server
npm run dev
```

Visit `http://localhost:3000` to see the site.

---

## Project Structure

| Path | Purpose |
|------|---------|
| `public/logo.png` | Site logo |
| `components/` | Shared UI components (Button, ListItem, etc.) |
| `components/formComponents/` | Form inputs and checkout fields |
| `context/mainContext.js` | Global state (cart, totals) |
| `pages/` | App pages — `index`, `cart`, `checkout`, `admin` |
| `templates/` | Category view, single item view, inventory layouts |
| `utils/inventoryProvider.js` | **Swap this** to connect a real data source |

---

## Connecting a Real Inventory Source

By default, inventory is read from a local hardcoded array. To pull from a live API, CMS, or Shopify, edit `utils/inventoryProvider.js` and update the `getInventory` function with your own fetch logic.

### Optional: Download Images at Build Time

If your inventory source returns remote image URLs, you can cache them locally at build time for better performance:

```js
import fs from 'fs'
import axios from 'axios'
import path from 'path'

async function downloadImage(url) {
  const key = url.split('/').pop().split('?')[0].replace(/%/g, '')
  const destPath = path.join(__dirname, '..', 'public', 'downloads', key)

  return new Promise(async (resolve, reject) => {
    const writer = fs.createWriteStream(destPath)
    const response = await axios({ url, method: 'GET', responseType: 'stream' })
    response.data.pipe(writer)
    writer.on('finish', resolve)
    writer.on('error', reject)
  })
}
```

Then map over your inventory after fetching:

```js
await Promise.all(
  inventory.map(async (item, index) => {
    try {
      if (!fs.existsSync(`${__dirname}/public/downloads/${item.image}`)) {
        await downloadImage(item.image)
      }
      inventory[index].image = `../downloads/${item.image}`
    } catch (err) {
      console.error('Image download failed:', err)
    }
  })
)
```

---

## Admin Panel

The admin panel (`pages/admin.js`) is included as a starting point. To activate it:

1. **`pages/admin.js`** — add sign up, sign in, sign out, and confirm sign-in methods
2. **`components/ViewInventory.js`** — connect to your inventory API for reading items
3. **`components/formComponents/AddInventory.js`** — connect to your inventory API for adding items

---

## Payments (Stripe)

The checkout page uses Stripe Elements for card collection. For **server-side payment processing**, see the [example Lambda function](https://github.com/jamstack-cms/jamstack-ecommerce/blob/next/snippets/lambda.js) in the snippets folder.

> **Tip:** For extra security, pass an array of item IDs to your server function, recalculate the total server-side, and compare it against the client total before charging.

---

## Deployment

**Vercel** (recommended):
```sh
vercel
```

**AWS Lambda (Serverless Framework):**
```sh
npx serverless
```

---

## Roadmap

- Full product and category search
- Auto dropdown navigation for large category lists
- Configurable item metadata
- Theming and dark mode
- Optional user accounts and profiles
- Responsive admin panel

Have an idea? [Open an issue](https://github.com/jamstack-cms/jamstack-ecommerce/issues) or [submit a PR](https://github.com/jamstack-cms/jamstack-ecommerce/pulls).
