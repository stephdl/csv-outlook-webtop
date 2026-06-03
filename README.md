# outlook-webtop-contacts

Convert Outlook contacts CSV exports to WebTop-ready CSV with automatic field mapping and automatic encoding detection.

## Problem

Migrating contacts from Outlook/Exchange to WebTop requires a CSV import. Outlook exports in **Windows-1252** (or UTF-8 BOM on recent Office 365), while WebTop expects **UTF-8**. On top of that, WebTop's import wizard requires manual field mapping unless the column names match its internal field names exactly.

This tool handles both problems: encoding conversion and column naming — so the import wizard maps all fields automatically at step 4, with zero manual work.

## Usage

Open `outlook_to_webtop.html` in any browser — no installation, no server, no dependencies.

1. Click **Choose file** and select the Outlook CSV
2. Click **Convert and download** — `yourfile_webtop.csv` is saved automatically

## Exporting from Outlook

**File → Open & Export → Import/Export → Export to a file → Comma Separated Values → Contacts**

## Importing into WebTop

1. Right-click your address book → **Import contacts**
2. Select the `_webtop.csv` file
3. **Step 2** — Delimiter: `Comma`, Text qualifier: `"`, Encoding: `UTF-8`
4. **Step 4** — all fields are mapped automatically → Next → Start

> `TaxCode`, `VATNumber` and `eInvoicingCode` will not be mapped — these are accounting fields absent from Outlook. This is expected and harmless.

## How it works

### Encoding detection

The file is read as raw bytes. The first 3 bytes are inspected:

- `EF BB BF` → UTF-8 BOM (recent Outlook 365) → decoded with `TextDecoder('utf-8')`
- No BOM → Windows-1252 (classic Outlook FR) → decoded byte-by-byte as latin-1, then `fixKey()` reinterprets the bytes as UTF-8

`fixKey()` replicates the Python idiom `k.encode("latin-1").decode("utf-8")` in JavaScript.

### Header normalisation

After decoding, all headers are passed through `norm()` which strips accents for comparison:

```
"Prénom"            → "prenom"
"Société"           → "societe"
"Dép/Région (bureau)" → "dep/region (bureau)"
```

Columns are matched by keyword, not by positional index — making the converter robust across Outlook versions and regional variants.

### Field mapping

The output CSV uses column names that match WebTop's internal field names exactly (`FirstName`, `LastName`, `WorkEmail`, `Company`, `Function`...), enabling automatic mapping at step 4 of the import wizard.

### Outlook's double "Titre" column

Outlook exports two columns named `Titre`: the first is the **salutation** (M., Mme, Dr...) mapped to `Title`, the second is the **job title** mapped to `Function`. Both are detected by finding all positions where the header normalises to `"titre"`.

## Supported encodings

| Encoding | Detected by | Typical source |
|----------|-------------|----------------|
| UTF-8 BOM | `EF BB BF` prefix | Outlook 365 (recent) |
| Windows-1252 | No BOM | Outlook on French Windows |

Other regional encodings (Windows-1250, Windows-1251, Shift-JIS) are not supported. For a Paris-based Office installation, the two above cover all cases.

## Field mapping reference

| Outlook (French) | WebTop field |
|------------------|-------------|
| Titre (1st) | Title |
| Prénom | FirstName |
| Nom | LastName |
| Titre (2nd) | Function |
| Société | Company |
| Service | Department |
| Adresse de courrier | WorkEmail |
| Téléphone (bureau) | WorkTelephone |
| Tél. mobile | WorkMobile |
| Télécopie (bureau) | WorkFax |
| Rue (bureau) | WorkAddress |
| Code postal (bureau) | WorkPostalCode |
| Ville (bureau) | WorkCity |
| Pays/Région (bureau) | WorkCountry |
| Téléphone (domicile) | HomeTelephone |
| Adresse de courrier 2 | HomeEmail |
| Adresse de courrier 3 | OtherEmail |
| Responsable | Manager |
| Nom de l'assistant(e) | Assistant |
| Page web | Url |
| Notes | Notes |

## Privacy

Runs entirely in the browser. No data is sent to any server.

## License

GPLV3
