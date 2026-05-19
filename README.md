# shopify-product-import-skill
This skill was created to allow people to actually easily transform an existing csv into a format that would actually work for their specific shopify store by just providing their export file and the new file that contains new products

---

## What it does

- Maps columns from any source spreadsheet to your store's Shopify format
- Handles multiple images and multiple variants correctly (the multi-row/handle structure that breaks everything)
- Detects duplicates and conflicts before you import
- Generates a small test file first so you can verify it works
- Produces a clean, ready-to-import CSV

---

## What you need

- [Claude Code](https://claude.ai/code) or Claude with skills support
- A spreadsheet of new products (any format — supplier CSV, Excel, whatever)
- An export of your existing Shopify products (from Shopify Admin → Products → Export)

---

## Install

**Upload to claude.ai:**
1. Download ZIP from this repo (green Code button → Download ZIP)
2. Go to claude.ai → Settings → Customize → Skills → Upload
3. Upload the ZIP — done

**Via Claude Code:**
```bash
git clone https://github.com/mmeziad/shopify-product-import-skill ~/.claude/skills/shopify-product-import
```

---

## How to use it

Once installed, just tell Claude what you're trying to do:

> "I have a supplier CSV and a Shopify export, help me build an import file"

It'll walk you through the rest — column mapping, filters, duplicate handling, test file, full file.

---

## Why this exists

Shopify's CSV import is annoying. The docs don't make it clear which fields do what, image rows have to be in exactly the right order, and one wrong column name silently breaks everything. I built this because I kept running into the same problems and wanted a way to get it right the first time.

---

## Tests done

I tested this on a simple single variant dataset from one of my more basic stores, but haven't had the chance to test it on a more complex data set with multi-variant and multi-image cases and more complicated custom metaobjects, so if you do, please let me know how it goes!

---

## License

MIT — use it, fork it, improve it.
