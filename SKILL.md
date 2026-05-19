---
name: shopify-product-import
description: >
  Use this skill whenever the user wants to prepare, generate, or fix a Shopify product import CSV.
  Triggers include: importing products to Shopify, preparing a product CSV, mapping supplier data to Shopify format,
  handling product variants or images for import, merging new products with an existing Shopify export,
  fixing a Shopify CSV that isn't importing correctly, or generating a test import file.
  Also use when the user mentions "handle", "variants", "Shopify CSV", product images not linking correctly,
  duplicate rows, or wanting to update existing products. This skill is especially important when both a
  source spreadsheet AND a Shopify export are involved — always use it in that case.
---

# Shopify Product Import Skill

This skill guides Claude through transforming one or more source spreadsheets into a valid Shopify product import CSV, with a test file generated first. It accounts for Shopify's tricky multi-row/handle structure, variant logic, image ordering, and store-specific column setups.

---

## Overview of the Workflow

1. **Analyze both files** — understand the source data structure and the store's existing Shopify export format
2. **Column mapping** — interactively map source columns → Shopify columns, guided by the store's own export
3. **Filter setup** — let the user define what to include/exclude (new products only, specific vendors, etc.)
4. **Validation pass** — catch issues before generating (duplicates, missing required fields, image problems)
5. **Generate test file** — 3–5 representative products covering edge cases (multi-image, multi-variant, single)
6. **Review & confirm** — present the test file for user review, explain any decisions made
7. **Generate full import file** — produce the final CSV ready for Shopify admin import

---

## Step 1: Analyze Both Files

Read both files carefully. For each, extract:
- Column headers (exact names, casing, spacing)
- A sample of 3–5 rows to understand the data shape
- Presence of key Shopify fields: Handle, Title, Option columns, Image Src, Variant SKU, Variant Price
- Any custom metafield columns (format: `custom.field_name` or namespace format)
- Whether the export already uses multi-row handle grouping for images/variants

**From the Shopify export specifically**, note:
- Which optional columns the store actually uses (ignore columns that are entirely empty)
- The store's handle format (e.g., `product-title`, `vendor-product-title`)
- Metafield columns and their namespaces
- Whether products use Option1/2/3 and what those option names are (Color, Size, Material, etc.)

Read the reference file for full Shopify CSV column details:
→ `references/shopify-csv-reference.md`

---

## Step 2: Column Mapping

Present a clear mapping table to the user. For each Shopify column that has data in the source:
- Show the source column name → suggested Shopify column name
- Mark confidence: ✅ confident match, ⚠️ needs confirmation, ❓ no clear match

Format as a readable table the user can quickly scan and correct, like:

```
Source Column          → Shopify Column              Status
─────────────────────────────────────────────────────────
Product Name           → Title                       ✅
SKU                    → Variant SKU                 ✅
Retail Price           → Variant Price               ✅
MSRP                   → Variant Compare At Price    ⚠️ confirm
Color                  → Option1 Value               ⚠️ (Option1 Name = "Color"?)
Size                   → Option2 Value               ⚠️ (Option2 Name = "Size"?)
Image 1                → Image Src                   ✅
Image 2                → (additional image row)      ✅ will generate extra rows
Description            → Body (HTML)                 ✅
Brand                  → Vendor                      ✅
Category               → Type                        ⚠️ confirm
Barcode                → Variant Barcode             ✅
Weight (kg)            → Variant Grams               ⚠️ will convert to grams
─────────────────────────────────────────────────────────
NOT MAPPED (source):   wholesale_price, internal_id
NOT MAPPED (Shopify):  Tags, Published, SEO Title, SEO Description
```

Ask the user to confirm or correct before proceeding.

For unmapped Shopify columns, ask:
- Should they be left blank? (fine for most optional fields)
- Should a default value be used? (e.g., `Published = TRUE`, `Variant Inventory Policy = deny`)

---

## Step 3: Filter Setup

Ask the user how to filter the source data:

**Common filters to offer:**
- Include all rows, or only rows where a certain column matches a value (e.g., `Status = Active`)
- Exclude products already in the Shopify export (match by SKU, Title, or Handle)
- Only include rows where a key field (price, image, SKU) is not empty
- Vendor filter (only include/exclude certain brands)

For **duplicate detection**, always check:
- Are there rows in the source that would generate the same Handle? (see Handle Generation below)
- Are there SKUs that appear more than once in unexpected ways?
- Are there products in the source that already exist in the Shopify export?

Present findings clearly:
```
⚠️  Found 3 potential duplicates:
    - "Blue Widget 500ml" appears twice in source (rows 14 and 47)
    - "Red Cap" already exists in your Shopify export (handle: red-cap)

How should these be handled?
  [a] Skip duplicates (keep existing Shopify version)
  [b] Overwrite with source data
  [c] Review each one manually
```

---

## Step 4: Handle Generation

Shopify uses the Handle as the unique product identifier. It must be:
- Lowercase
- Hyphens instead of spaces
- No special characters
- Unique across all products

**Generate handles from Title** unless the source already has a handle column:
```
"Blue Widget 500ml" → "blue-widget-500ml"
"Red Cap (Large)" → "red-cap-large"
```

If the store's export shows a different pattern (e.g., `vendor-title`), match that convention.

**Multi-row products**: All variant rows and image rows for a product share the same Handle. This is how Shopify groups them.

---

## Step 5: Build the Multi-Row Structure

This is the most critical and error-prone part of Shopify CSV imports. Follow these rules precisely:

### Row Types

**Row 1 of a product (required):**
- ALL product-level fields: Handle, Title, Body (HTML), Vendor, Type, Tags, Published
- First variant data: Option1 Name + Value, Option2 Name + Value (if applicable), Variant SKU, Variant Price, etc.
- First image: Image Src, Image Position = 1, Image Alt Text

**Additional variant rows (same Handle):**
- Handle only (no Title, Body, Vendor, Type, Tags, Published — leave blank)
- New option values: just the Option Value columns, not Option Name
- Variant-specific fields: SKU, Price, Compare At Price, Barcode, Grams, etc.
- Can optionally include a Variant Image (URL linking this variant to its specific image)
- Image Src can be included if this variant has a new image to add

**Image-only rows (same Handle, when there are more images than variants):**
- Handle only
- Image Src, Image Position (incrementing: 2, 3, 4...), Image Alt Text
- Leave all variant fields blank

### Example: 1 product, 2 colors, 3 images

```csv
Handle,Title,Body (HTML),Vendor,Option1 Name,Option1 Value,Variant SKU,Variant Price,Image Src,Image Position,Variant Image
blue-cap,Blue Cap,<p>A great cap</p>,BrandCo,Color,Blue,CAP-BLU,29.99,https://img.com/cap-blue.jpg,1,https://img.com/cap-blue.jpg
blue-cap,,,,Color,Red,CAP-RED,29.99,https://img.com/cap-red.jpg,2,https://img.com/cap-red.jpg
blue-cap,,,,,,,,https://img.com/cap-lifestyle.jpg,3,
```

### Key Rules
- Image Position must be sequential integers starting at 1 per product
- Variant Image must exactly match one of the Image Src values for that product
- If a source row has multiple image columns (Image 1, Image 2, Image 3), explode into multiple rows
- Never repeat Option1 Name / Option2 Name on non-first rows — only the Value columns
- Variant Inventory Policy should default to `deny` unless told otherwise
- Variant Fulfillment Service should default to `manual`
- Variant Requires Shipping defaults to `TRUE`
- Variant Taxable defaults to `TRUE`

---

## Step 6: Validation Before Output

Run these checks before generating either file:

| Check | What to look for |
|-------|-----------------|
| Required fields | Every product has Handle, Title, Variant Price |
| Handle uniqueness | No two different products share a handle |
| Image URLs | URLs are not empty strings, don't contain spaces (encode as %20) |
| Image Position | Sequential per handle, starting at 1, no gaps |
| Variant Image | Each Variant Image URL exactly matches an Image Src for that handle |
| Option consistency | All rows for a handle use the same Option1 Name (can't mix "Color" and "Colour") |
| Price format | Numbers only, no currency symbols, max 2 decimal places |
| Grams | Integer, no decimals |
| Published | TRUE or FALSE only |
| Variant Inventory Qty | Integer only |
| Duplicate SKUs | Warn if same SKU appears on multiple products (ok within one product's variants) |

Report all issues found. For warnings (non-blocking), note them but proceed. For errors (would cause import failure), fix them or ask the user.

---

## Step 7: Generate Test File

Before generating the full file, produce a test CSV with 3–5 products that together cover:
- ✅ A simple product (1 variant, 1 image)
- ✅ A product with multiple variants (at least 2 option values)
- ✅ A product with multiple images
- ✅ A product with both multiple variants AND multiple images (if any exist)
- ✅ Any edge case specific to this data (e.g., missing images, very long descriptions)

Label the file clearly: `test_import_YYYY-MM-DD.csv`

After presenting it, explain:
- Which products were chosen and why
- Any decisions made (defaulted values, generated handles, etc.)
- Any warnings found
- How to test it: "Import this in Shopify Admin → Products → Import, check 3 things: product titles look right, images attached correctly, variants showing as expected"

---

## Step 8: Generate Full Import File

Once the user confirms the test file looks good, generate the full file.

Name it: `shopify_import_YYYY-MM-DD.csv`

Include a brief summary after:
```
✅ Import file ready: shopify_import_2024-11-15.csv

📊 Summary:
   • 47 products
   • 134 total rows (products + variant rows + image rows)
   • 3 products had duplicate detection applied (skipped)
   • 2 products had missing images (Image Src left blank — check these)
   • Default values applied: Published=TRUE, Inventory Policy=deny, Fulfillment=manual

⚠️  Before importing:
   • Review the 2 products with missing images (rows 23, 67)
   • Shopify will match existing products by Handle — 5 products will be UPDATED, not created
   • Always back up your current products with an export before importing
```

---

## Common Gotchas to Watch For

**"My images aren't showing up"**
→ Image Position must start at 1 and be sequential. A gap or restart causes images to drop.

**"My variants look wrong / merged into one"**
→ Option1 Name must be identical across all rows for a handle. Check for typos or case differences.

**"I'm getting duplicate products"**
→ Two source rows generated the same handle. Usually happens with products that have similar names. Deduplicate before import or rename.

**"Some products updated unexpectedly"**
→ Shopify matches by Handle on import. If a handle in your import file matches an existing product, it will update it.

**"Variant images are blank"**
→ Variant Image URL must exactly match Image Src (same URL, not a different crop or format).

**"Tags are getting wiped"**
→ If Tags column is present but empty, Shopify will clear existing tags. Only include Tags column if you're intentionally setting them.

---

## Reference Files

→ `references/shopify-csv-reference.md` — Full column reference, types, defaults, and Shopify import rules
