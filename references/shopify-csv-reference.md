# Shopify CSV Column Reference

Full reference for every column in a Shopify product import CSV. Use this to validate mappings and set correct defaults.

---

## Required Columns

| Column | Type | Notes |
|--------|------|-------|
| Handle | string | URL-safe slug. Lowercase, hyphens. Unique per product. Shared across all rows for the same product. |
| Title | string | Only on first row per handle. Leave blank on subsequent rows. |
| Variant Price | decimal | Required on every variant row. No currency symbols. E.g., `29.99` |

---

## Product-Level Columns (first row of each handle only)

| Column | Type | Default | Notes |
|--------|------|---------|-------|
| Title | string | — | Product title |
| Body (HTML) | string | blank | Product description. Can contain HTML. |
| Vendor | string | blank | Brand/manufacturer name |
| Product Category | string | blank | Shopify's standardized taxonomy (e.g., `Apparel & Accessories > Clothing`) |
| Type | string | blank | Custom product type (freeform, used for filtering) |
| Tags | string | blank | Comma-separated. **Caution**: if column present but empty, wipes existing tags. |
| Published | boolean | TRUE | `TRUE` = visible in storefront. `FALSE` = hidden/draft. |
| Status | string | active | `active`, `draft`, or `archived`. Overrides Published in newer Shopify. |
| Gift Card | boolean | FALSE | `TRUE` only for gift card products |
| SEO Title | string | blank | Overrides title in search engines |
| SEO Description | string | blank | Meta description for search engines |

---

## Option Columns

Shopify supports up to 3 options per product (e.g., Color, Size, Material).

| Column | Type | Notes |
|--------|------|-------|
| Option1 Name | string | E.g., `Color`. Only on first row per handle. |
| Option1 Value | string | E.g., `Blue`. On every variant row. |
| Option2 Name | string | E.g., `Size`. Only on first row per handle. |
| Option2 Value | string | E.g., `Large`. On every variant row. |
| Option3 Name | string | E.g., `Material`. Only on first row per handle. |
| Option3 Value | string | On every variant row. |

**Rules:**
- If a product has no variants/options, use `Option1 Name = Title` and `Option1 Value = Default Title`
- Option Names must be consistent across all rows for the same handle
- If you only use Option1, leave Option2 and Option3 columns blank (don't include them or leave empty)

---

## Variant Columns (every variant row)

| Column | Type | Default | Notes |
|--------|------|---------|-------|
| Variant SKU | string | blank | Stock keeping unit |
| Variant Grams | integer | 0 | Weight in grams. No decimals. |
| Variant Inventory Tracker | string | blank | `shopify` to track inventory, blank to not track |
| Variant Inventory Qty | integer | 0 | Starting inventory. Only applied on NEW products; ignored on updates unless "Overwrite" selected. |
| Variant Inventory Policy | string | `deny` | `deny` = stop selling when out of stock. `continue` = allow overselling. |
| Variant Fulfillment Service | string | `manual` | `manual` for self-fulfilled. Or third-party fulfillment service handle. |
| Variant Price | decimal | — | **Required**. E.g., `19.99` |
| Variant Compare At Price | decimal | blank | Original/crossed-out price. Must be higher than Variant Price or leave blank. |
| Variant Requires Shipping | boolean | `TRUE` | `FALSE` for digital products |
| Variant Taxable | boolean | `TRUE` | Whether to charge tax |
| Variant Barcode | string | blank | ISBN, UPC, GTIN, etc. |
| Variant Weight Unit | string | `kg` | `kg`, `g`, `lb`, or `oz`. Shopify stores internally as grams. |
| Variant Tax Code | string | blank | For Avalara/tax providers |
| Cost per item | decimal | blank | Your cost (not shown to customers) |

---

## Image Columns

| Column | Type | Notes |
|--------|------|-------|
| Image Src | string | Full URL to image. Must be publicly accessible. |
| Image Position | integer | Sequential, starting at 1 per product. Image 1 is the featured image. |
| Image Alt Text | string | Alt text for accessibility and SEO. |
| Variant Image | string | URL of the image to associate with this variant. Must exactly match one of the Image Src values for this handle. |

**Image row rules:**
- Each unique `Image Src` value for a handle gets its own row or is on a variant row
- `Image Position` must be 1, 2, 3, 4... with no gaps and no repeats per handle
- If a variant has no specific image, leave `Variant Image` blank (it will use the product's first image)
- Image URLs should not contain spaces (use `%20` if needed, though best to fix at source)
- Shopify accepts: jpg, jpeg, png, gif, webp
- Max image size: 20MB, recommended max resolution: 5000x5000px

---

## Metafield Columns

Metafields allow storing extra data on products. They appear as additional columns in exports.

| Format | Example | Notes |
|--------|---------|-------|
| `namespace.key` (legacy) | `custom.care_instructions` | Older format |
| `namespace.key [type]` | `custom.wash_temp [integer]` | Newer format with type hint |

**Common metafield types:** `single_line_text_field`, `multi_line_text_field`, `integer`, `number_decimal`, `url`, `boolean`, `date`, `json`

When preserving metafields from an existing Shopify export, copy the column headers exactly as they appear.

---

## Google Shopping Columns (optional)

These appear in exports but rarely need to be set manually:

- `Google Shopping / Google Product Category`
- `Google Shopping / Gender`
- `Google Shopping / Age Group`
- `Google Shopping / MPN`
- `Google Shopping / AdWords Grouping`
- `Google Shopping / AdWords Labels`
- `Google Shopping / Condition`
- `Google Shopping / Custom Product`
- `Google Shopping / Custom Label 0` through `Custom Label 4`

---

## Import Behavior Reference

### How Shopify Matches on Import

| Scenario | Behavior |
|----------|----------|
| Handle exists in store | **Updates** the existing product |
| Handle does not exist | **Creates** a new product |
| Handle exists, new variant option combo | **Adds** the variant to the product |
| Handle exists, variant already exists | **Updates** the variant |

### What Gets Overwritten vs. Preserved

| Field | On Update |
|-------|-----------|
| Title, Body, Vendor, Type | Overwritten if column present |
| Tags | Overwritten if column present (even if blank!) |
| Images | Merged/added — existing images not removed unless you remove Image Src entirely |
| Metafields | Overwritten if column present |
| Variant Inventory Qty | **NOT updated by default** — need "Overwrite existing products" option |

### The "Overwrite existing products" checkbox in Shopify Admin

- **Unchecked (default)**: New products created, existing products updated but inventory NOT changed
- **Checked**: Everything overwritten including inventory quantities

---

## Minimal Valid CSV Example

Single product, no variants, one image:

```csv
Handle,Title,Body (HTML),Vendor,Type,Tags,Published,Option1 Name,Option1 Value,Variant SKU,Variant Grams,Variant Inventory Tracker,Variant Inventory Policy,Variant Fulfillment Service,Variant Price,Variant Requires Shipping,Variant Taxable,Image Src,Image Position
blue-widget,Blue Widget,"<p>A great widget</p>",WidgetCo,Widgets,"widget,blue",TRUE,Title,Default Title,WDG-BLU-001,200,shopify,deny,manual,24.99,TRUE,TRUE,https://example.com/blue-widget.jpg,1
```

Multi-variant product with multiple images:

```csv
Handle,Title,Body (HTML),Vendor,Option1 Name,Option1 Value,Option2 Name,Option2 Value,Variant SKU,Variant Price,Image Src,Image Position,Variant Image
cool-hat,Cool Hat,"<p>A cool hat</p>",HatCo,Color,Black,Size,S,HAT-BLK-S,34.99,https://example.com/hat-black.jpg,1,https://example.com/hat-black.jpg
cool-hat,,,,Color,Black,Size,M,HAT-BLK-M,34.99,,,https://example.com/hat-black.jpg
cool-hat,,,,Color,Black,Size,L,HAT-BLK-L,34.99,,,https://example.com/hat-black.jpg
cool-hat,,,,Color,White,Size,S,HAT-WHT-S,34.99,https://example.com/hat-white.jpg,2,https://example.com/hat-white.jpg
cool-hat,,,,Color,White,Size,M,HAT-WHT-M,34.99,,,https://example.com/hat-white.jpg
cool-hat,,,,Color,White,Size,L,HAT-WHT-L,34.99,,,https://example.com/hat-white.jpg
cool-hat,,,,,,,,,,https://example.com/hat-lifestyle.jpg,3,
```

---

## Common Import Error Messages

| Shopify Error | Likely Cause |
|---------------|-------------|
| "Variant price can't be blank" | Variant Price missing on a row |
| "Handle has already been taken" | Duplicate handle within same import file |
| "Option value can't be blank" | Option1 Value missing on a variant row |
| "Images could not be downloaded" | Image URL is broken, private, or contains spaces |
| "Variant SKU has already been taken" | Same SKU used on a different product (SKUs must be unique store-wide) |
