# JOIL user manual

## What is JOIL

JOIL (JetOctopus Internal Linker) is an AI-powered internal linking tool that helps you pass link equity from your strong pages to pages that need more visibility.

It works by analyzing two types of pages:
- **Donors**: Powerful pages with traffic, authority, and crawl budget that give links
- **Acceptors**: Pages that need a boost and receive links

Using AI embeddings, JOIL understands the semantic meaning of your content and matches donors to acceptors based on topical relevance. This ensures every recommended link makes contextual sense and helps your target pages rank better.

JOIL integrates seamlessly with Screaming Frog, taking your CSV exports and generating a ready-to-implement list of link recommendations with anchor text.

### What JOIL does

- Analyzes your website pages to find linking opportunities
- Uses AI to match pages based on content similarity
- Generates anchor text recommendations
- Respects your linking rules and limits
- Outputs a CSV file ready for implementation

### When to use JOIL

- Building internal link structure for new websites
- Improving internal linking on existing sites
- Scaling internal linking efforts across large websites
- Finding contextually relevant linking opportunities
- Automating link recommendations based on content similarity

## Core concepts

### Acceptors and donors

JOIL works with two types of pages:

**Donors** are pages that give links. These are typically "powerful pages" with:
- Crawl budget
- Organic traffic
- Search impressions
- Existing authority

Common donor page types:
- Blog posts with traffic
- Popular resource pages
- High-ranking content pages
- Category pages with visibility
- Long-form guides with authority

**Acceptors** are pages that receive links. These are typically "weak pages" that:
- May lack crawl budget
- Have low or no organic traffic
- Need a boost in internal linking value
- You want to help rank better

Common acceptor page types:
- New product pages
- Underperforming service pages
- Deep-level category pages
- Pages you want to promote
- Content that needs more visibility

**The opposite approach also works:**
Acceptors can be high-traffic pages while donors are a vast number of low-performance pages. This helps distribute link equity across many pages that need attention.

You can use the same CSV file for both acceptors and donors if you want any page to potentially link to any other page.

### How AI is used for linking

JOIL uses AI embeddings to understand the semantic meaning of your pages. Here's how it works:

**AI embeddings** are mathematical representations of your page content. 
Each page is converted into a list of numbers (typically 1,536 dimensions for OpenAI embeddings) that capture its meaning. 
Pages about similar topics have similar embeddings.

**AI distance** measures how similar two pages are:
- Lower distance = more similar content
- Higher distance = less related content

JOIL uses AI distance in two ways:

1. **Filtering**: Only creates links between pages that are sufficiently similar (above the minimum AI distance threshold)
2. **Sorting**: Ranks potential links by similarity, giving you the best matches first

The default minimum AI distance is 0.6, which creates links between moderately to highly related pages.

### What are matching fields

Matching fields are data columns that JOIL uses to determine which pages should link together. They act as filters before AI similarity is calculated.

**Examples of matching fields:**
- Category
- Topic
- Location (city, state, region)
- Product type
- Content type

**How matching fields work:**

If you set matching fields to "category,location":
- A donor about "Real Estate in Miami" can link to an acceptor about "Real Estate in Miami"
- But not to an acceptor about "Real Estate in Tampa" (location doesn't match)
- And not to an acceptor about "Restaurants in Miami" (category doesn't match)

Matching fields help ensure your links are contextually appropriate beyond just semantic similarity.

### JSON-LD breadcrumbs requirement

For JOIL to work effectively with Screaming Frog, your pages need to include JSON-LD breadcrumb schema markup.
This structured data helps JOIL understand your site hierarchy and page relationships.

If your site doesn't have JSON-LD breadcrumbs, you'll need to set up custom extraction in Screaming Frog to capture
the matching fields from pages, such as categories, brand names, types, and so on.


### How the algorithm works

JOIL offers two algorithms for creating link recommendations:

**Greedy algorithm** (default):
1. For each acceptor page, finds all matching donors
2. Calculates AI distance to each potential donor
3. Filters donors by minimum AI distance
4. Sorts donors by AI similarity (best matches first)
5. Creates links up to your acceptor limit (default: 5 links)
6. Moves to the next acceptor

This algorithm prioritizes quality matches. Popular acceptors may receive their full allocation quickly, while less common topics might get fewer links.

**Equal algorithm**:
1. Finds all potential matches for all acceptors
2. Sorts matches by AI similarity
3. Distributes links evenly across all acceptors in rounds
4. Each round gives one link to each acceptor
5. Continues until acceptors reach their limits or donors are exhausted

This algorithm ensures balanced distribution. All acceptors get similar numbers of links, preventing concentration on popular pages.

## Screaming Frog workflow

This is the recommended workflow for using JOIL with Screaming Frog.

### Prerequisites

Before starting, you need:
- Screaming Frog SEO Spider (desktop application)
- OpenAI API key
- Website to analyze
- JOIL binary on your computer

### Crawl settings

You need to configure Screaming Frog to collect the right data for JOIL.

#### Text embeddings setup

1. Open Screaming Frog
2. Go to **Configuration** > **API Access** > **OpenAI**
3. Enter your OpenAI API key
4. Click **OK** to save

Next, configure the embeddings:

1. Go to **Configuration** > **Custom** > **Extraction**
2. Click **Add** to create a new extraction
3. In the **Prompt Configuration** tab:
   - Name: "Embeddings for Page Text"
   - Select the embedding model (e.g., text-embedding-3-small)
4. Click **Test** to verify it works
5. Click **OK** to save

#### Structured data extraction

1. Go to **Configuration** > **Extraction** > **Structured Data** > **JSON-LD**
2. Check the checkbox to enable JSON-LD extraction
3. Click **OK**

#### Custom extraction setup

Extraction of the structured data:

1. Go to **Configuration** > **Custom** > **Extraction**
2. Click **Add** to create a new extraction
3. Configure:
   - Name: "JSON-LD"
   - Extractor: "CSSPath"
   - Expression: `script[type="application/ld+json"]`
   - Extract: "Inner HTML"
4. Click **OK** to save

#### Run the crawl

1. Enter your website URL in Screaming Frog
2. Click **Start** to begin crawling
3. Wait for the crawl to complete
4. The embeddings will be generated automatically for each page

### Required exports

You need to export three CSV files from Screaming Frog:

1. **Acceptors URLs** - Pages that will receive links
2. **Donors URLs** - Pages that will give links
3. **Internal links** - Existing links on your site (optional but recommended)

#### Export acceptors and donors

These exports follow the same process:

1. In the main Screaming Frog window, go to the **Internal** tab
2. In the dropdown menu, select **HTML**
3. Use the filter to select the pages you want:
   - For acceptors: Filter to important pages (products, services, categories)
   - For donors: Filter to content pages (blog posts, guides, resources)
   - For both: You can use the same export if all pages can be both acceptors and donors
4. Click the **Export** button
5. Select **Export As** > **CSV**
6. Save with a descriptive filename:
   - `acceptors.csv` for acceptor pages
   - `donors.csv` for donor pages
   - `all_pages.csv` if using the same file for both

#### Export internal links

Exporting existing links helps JOIL avoid recommending duplicate links:

1. In the Screaming Frog menu, go to **Bulk Export** > **Links** > **All Inlinks**
2. Save as CSV with filename like `existing_links.csv`

This export is optional but recommended to prevent duplicate link suggestions.

### Verify your exports

Before running JOIL, verify your CSV files have the required columns:

**Acceptors/donors CSV must include:**
- `Address` or `url` - The page URL
- `Extract embeddings from page content` - AI embeddings (if using OpenAI)
- Any custom fields you want to use for matching (category, location, etc.)

**Internal links CSV should include:**
- `url` - The source page URL
- `target_url` - The destination page URL

## Running JOIL

### Basic usage

The simplest way to run JOIL with Screaming Frog exports:

```bash
./joil --acceptors-file acceptors.csv --donors-file donors.csv
```

JOIL automatically detects the Screaming Frog format and creates an output file called `interlinking.csv` in the same directory as your acceptors file.

### Using the same file for acceptors and donors

If you want any page to potentially link to any other page:

```bash
./joil --acceptors-file all_pages.csv --donors-file all_pages.csv
```

JOIL automatically prevents pages from linking to themselves.

### Specifying output location

To save the results to a specific location:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --output-file /path/to/recommendations.csv
```

### Adding matching fields

To use matching fields for more targeted linking:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --match-fields "category,location"
```

This ensures links are only created between pages with matching category AND location values.

### Adjusting link limits

To control how many links each page gives or receives:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --max-acceptor-links 10 \
       --max-donor-links 5
```

- `--max-acceptor-links` - Maximum links each acceptor receives (default: 5)
- `--max-donor-links` - Maximum links each donor gives (default: 5)

### Changing AI distance threshold

To make linking more or less strict:

```bash
# Stricter - only very similar pages link
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --min-ai-distance 0.4

# Looser - more loosely related pages can link
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --min-ai-distance 0.7
```

Lower values = stricter matching, higher quality links
Higher values = more permissive matching, more link opportunities

### Choosing an algorithm

To use the equal distribution algorithm:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --algorithm equal
```

Use `greedy` (default) for quality-focused linking or `equal` for balanced distribution.

### Complete example

A comprehensive command with multiple options:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --output-file link_recommendations.csv \
       --match-fields "category,city" \
       --max-acceptor-links 8 \
       --max-donor-links 5 \
       --min-ai-distance 0.6 \
       --algorithm greedy \
       --anchor-field "title"
```

## Understanding the output

### Output file format

JOIL creates a CSV file with three columns:

```csv
donor_url,anchor_text,acceptor_url
```

**Example output:**

```csv
donor_url,anchor_text,acceptor_url
https://example.com/blog/miami-guide,Miami Real Estate,https://example.com/miami-homes
https://example.com/blog/tampa-tips,Tampa Properties,https://example.com/tampa-homes
https://example.com/resources/buying-guide,First Time Home Buyer,https://example.com/first-time-buyers
```

### Understanding each column

**donor_url** - Where to place the link
- This is the page that should contain the new link
- Typically a content page (blog post, article, guide)

**anchor_text** - The link text
- The clickable text for the link
- Generated from the acceptor page's title or custom anchors
- Should be descriptive and relevant

**acceptor_url** - Link destination
- This is where the link points to
- Typically an important page (product, service, category)

### How to implement the recommendations

1. **Review the output**:
   - Open the CSV in Excel or Google Sheets
   - Check a sample of recommendations for quality
   - Verify the anchor text makes sense

2. **Filter if needed**:
   - Remove any links that don't fit editorially
   - Adjust anchor text if necessary
   - Sort by donor URL to see all links for each page

3. **Import to your CMS**:
   - Most CMS platforms can import CSV files
   - Map the columns to your internal link fields
   - Process in batches if you have many recommendations

4. **Manual implementation**:
   - Give the CSV to your content team
   - Add links to the specified donor pages
   - Use the exact anchor text provided

5. **Automated implementation**:
   - Use the JOIL API if available
   - Build custom scripts to process the CSV
   - Integrate with your deployment pipeline

## Advanced features

### Custom anchor text

You can provide your own anchor text instead of using page titles.

#### Create a custom anchors file

Create a CSV with these columns:

```csv
url,query,importance
https://example.com/product-page,buy product name,10000
https://example.com/product-page,product name reviews,5000
https://example.com/product-page,best product name,3000
```

**Columns explained:**
- `url` - The acceptor page URL
- `query` - The anchor text to use
- `importance` - Weight for this anchor (higher = used more often)

#### How importance weighting works

If a page receives 10 links and has these anchors:
- "buy product" with importance 10,000 (62.5%)
- "product reviews" with importance 5,000 (31.25%)
- "best product" with importance 1,000 (6.25%)

JOIL will distribute:
- ~6 links with "buy product"
- ~3 links with "product reviews"
- ~1 link with "best product"

This helps create natural anchor text variation.

#### Using custom anchors

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --custom-anchors-file custom_anchors.csv
```

#### Mixing custom and standard anchors

To use both custom anchors and page titles:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --custom-anchors-file custom_anchors.csv \
       --standard-anchor-percent 0.3
```

This creates:
- 70% of links using custom anchors (weighted by importance)
- 30% of links using standard anchors (from page title field)

### Anchor text cleaning

Remove unwanted text from anchor text using regular expressions:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --anchor-clean-regexp '\[.*?\]' \
       --anchor-clean-regexp '®|™'
```

This removes:
- Text in square brackets: `[Updated 2024]`
- Trademark symbols: `®`, `™`

You can add multiple `--anchor-clean-regexp` flags to apply several cleaning rules.

### Avoiding duplicate links

To prevent JOIL from recommending links that already exist:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --existing-links-file existing_links.csv
```

The existing links CSV should have `url` and `target_url` columns (exported from Screaming Frog).

### Minimum field matching

Control how many matching fields are required:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --match-fields "category,city,state" \
       --min-fields-required 2
```

This allows links when at least 2 out of 3 fields match, creating more linking opportunities while maintaining relevance.

### AI-only linking

To skip field matching and rely entirely on AI semantic similarity:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --match-fields "" \
       --enable-ai-sorting
```

This is useful when:
- You don't have good matching fields
- Your content is highly varied
- You want maximum flexibility in link recommendations

### Disabling AI sorting

If you want to use only field matching without AI sorting:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --match-fields "category" \
       --enable-ai-sorting=false
```

Links will be created based on field matches only, without considering semantic similarity.

### Debug mode

To see detailed information about what JOIL is doing:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --debug
```

This shows:
- Which pages are being processed
- Matching results
- AI distance calculations
- Link creation decisions

## Command-line reference

### Required flags

| Flag | Description | Example |
|------|-------------|---------|
| `--acceptors-file` | CSV file with pages that receive links | `--acceptors-file acceptors.csv` |
| `--donors-file` | CSV file with pages that give links | `--donors-file donors.csv` |

### Output options

| Flag | Description | Default | Example |
|------|-------------|---------|---------|
| `--output-file` | Where to save recommendations | `interlinking.csv` in acceptors directory | `--output-file results.csv` |

### Matching options

| Flag | Description | Default | Example |
|------|-------------|---------|---------|
| `--match-fields` | Comma-separated field names for matching | None | `--match-fields "category,city"` |
| `--min-fields-required` | Minimum matching fields needed | All fields | `--min-fields-required 2` |
| `--fields-separator` | Character separating field values | `,` | `--fields-separator ";"` |

### Link limit options

| Flag | Description | Default | Example |
|------|-------------|---------|---------|
| `--max-acceptor-links` | Maximum links each acceptor receives | `5` | `--max-acceptor-links 10` |
| `--max-donor-links` | Maximum links each donor gives | `5` | `--max-donor-links 3` |

### Algorithm options

| Flag | Description | Default | Example |
|------|-------------|---------|---------|
| `--algorithm` | Linking algorithm (`greedy` or `equal`) | `greedy` | `--algorithm equal` |

### AI options

| Flag | Description | Default | Example |
|------|-------------|---------|---------|
| `--enable-ai-sorting` | Use AI to sort and filter links | `true` | `--enable-ai-sorting=false` |
| `--min-ai-distance` | Minimum AI similarity (0.0-1.0) | `0.6` | `--min-ai-distance 0.4` |
| `--ai-field-name` | Column name with AI embeddings | `Extract embeddings from page content` | `--ai-field-name "embeddings"` |

### Anchor text options

| Flag | Description | Default | Example |
|------|-------------|---------|---------|
| `--anchor-field` | Field to use for anchor text | `title` | `--anchor-field "h1"` |
| `--attributes` | Comma-separated fallback fields | None | `--attributes "title,h1"` |
| `--anchor-clean-regexp` | Regex to remove from anchors | None | `--anchor-clean-regexp '\[.*?\]'` |
| `--custom-anchors-file` | CSV with custom anchor text | None | `--custom-anchors-file anchors.csv` |
| `--standard-anchor-percent` | Percent using standard anchors (0.0-1.0) | `0` | `--standard-anchor-percent 0.3` |

### CSV format options

| Flag | Description | Default | Example |
|------|-------------|---------|---------|
| `--csv-delimiter` | CSV field delimiter | `,` | `--csv-delimiter ";"` |
| `--url-field-name` | Column name with URLs | `url` or `Address` | `--url-field-name "page_url"` |

### Utility options

| Flag | Description | Example |
|------|-------------|---------|
| `--help` | Show help message | `--help` |
| `--version` | Show JOIL version | `--version` |
| `--debug` | Enable debug logging | `--debug` |

## Troubleshooting

### No output file generated

**Problem**: JOIL runs but doesn't create an output file.

**Solutions**:
1. Check that your CSV files have the required columns
2. Verify the URL column is named `url`, `URL`, or `Address`
3. Make sure you have write permissions in the output directory
4. Check for error messages in the console
5. Try running with `--debug` flag to see detailed logs

### No links recommended

**Problem**: Output file is empty or has very few links.

**Possible causes and solutions**:

**AI distance too strict**:
- Try decreasing `--min-ai-distance` to 0.5
- Check if your pages have AI embeddings in the CSV

**Matching fields too restrictive**:
- Reduce the number of matching fields
- Lower `--min-fields-required`
- Try `--match-fields ""` to disable field matching

**Link limits too low**:
- Increase `--max-acceptor-links` and `--max-donor-links`

**Pages already linked**:
- Check if you provided an existing links file
- Remove the `--existing-links-file` flag to see all recommendations

### AI embeddings missing

**Problem**: Error about missing AI embeddings.

**Solutions**:
1. Verify you configured OpenAI API in Screaming Frog
2. Check the embeddings extraction ran successfully during crawl
3. Look for the column `Extract embeddings from page content` in your CSV
4. If using a different column name, specify it with `--ai-field-name`
5. Disable AI sorting with `--enable-ai-sorting=false` if you don't have embeddings

### Wrong CSV format detected

**Problem**: JOIL doesn't recognize your CSV format.

**Solutions**:
1. Ensure CSV is UTF-8 encoded
2. Check that the first row contains column headers
3. Verify URL column is named `url`, `URL`, or `Address`
4. If using a custom delimiter, specify with `--csv-delimiter`
5. Remove any BOM (Byte Order Mark) from the file

### Anchor text looks wrong

**Problem**: Generated anchor text is not what you expected.

**Solutions**:
1. Check which field is being used with `--anchor-field`
2. Try a different field: `--anchor-field "h1"` or `--anchor-field "meta_description"`
3. Use custom anchors file for complete control
4. Clean unwanted text with `--anchor-clean-regexp`
5. Verify the acceptor CSV has the field you're referencing

### Too many similar links

**Problem**: All recommended links use the same anchor text.

**Solutions**:
1. Use custom anchors with importance weighting for variation
2. Mix custom and standard anchors with `--standard-anchor-percent`
3. Check if your acceptor pages have diverse titles/headings
4. Consider manually editing the output CSV for more variety

### Performance issues

**Problem**: JOIL is running slowly on large datasets.

**Tips**:
1. JOIL is fast, but very large datasets (100,000+ pages) take time
2. AI embedding calculations are the slowest part
3. Consider splitting into smaller batches
4. Disable AI sorting if speed is critical: `--enable-ai-sorting=false`
5. Use matching fields to reduce the number of comparisons

### Permission denied errors

**Problem**: Can't read input files or write output file.

**Solutions**:
1. Check file permissions: `ls -la *.csv`
2. Ensure you have read access to input files
3. Ensure you have write access to output directory
4. Try saving output to a different location with `--output-file`
5. On Mac/Linux, you may need to make JOIL executable: `chmod +x joil`

## FAQ

### Can I use JOIL without Screaming Frog?

Yes. JOIL accepts any CSV file with the required columns:
- `url` - Page URL
- `Extract embeddings from page content` - AI embeddings (if using AI sorting)
- Any custom fields you want to match on

You can generate these CSVs from:
- Custom scripts
- Database exports
- Other SEO tools
- Manual creation

### Do I need AI embeddings to use JOIL?

No. You can disable AI sorting with `--enable-ai-sorting=false` and use only field matching. However, AI sorting provides better link quality by understanding semantic similarity.

### What's the difference between greedy and equal algorithms?

**Greedy** (default):
- Prioritizes best matches first
- Some acceptors may get all their links quickly
- Others might get fewer links
- Best for quality-focused linking

**Equal**:
- Distributes links evenly across all acceptors
- All acceptors get similar number of links
- Best for balanced distribution
- Good for ensuring all pages receive attention

### Can acceptors and donors be the same pages?

Yes. Use the same CSV file for both:

```bash
./joil --acceptors-file all_pages.csv --donors-file all_pages.csv
```

JOIL automatically prevents pages from linking to themselves.

### How do I choose the right min-ai-distance value?

**Recommended values:**
- `0.3-0.4` - Very strict, only highly similar pages (product variants, closely related topics)
- `0.5-0.6` - Moderate, reasonably related pages (default: 0.6)
- `0.7-0.8` - Loose, broadly related pages (more link opportunities)
- `0.9+` - Very loose, minimal relevance requirement

Start with the default (0.6) and adjust based on results. Review a sample of recommendations to determine if they're too strict or too loose.

### What if I don't have matching fields?

You have two options:

1. **Use AI-only matching**:
   ```bash
   --match-fields "" --enable-ai-sorting
   ```

2. **Add matching fields to your CSV**:
   - Export additional data from your CMS
   - Add category/topic columns manually
   - Use URL patterns as matching fields

### How many links should each page give and receive?

**General recommendations:**
- **Acceptor links**: 3-10 per page (default: 5)
- **Donor links**: 3-8 per page (default: 5)

Consider:
- Content length (longer pages can give more links)
- Page importance (important acceptors can receive more)
- User experience (too many links can be overwhelming)
- Editorial style (some sites prefer fewer, more strategic links)

### Can I run JOIL multiple times?

Yes. You can run JOIL multiple times with different settings:

1. **Test different parameters**:
   - Try different AI distance thresholds
   - Compare greedy vs equal algorithms
   - Test various matching field combinations

2. **Generate multiple batches**:
   - Create separate recommendations for different page types
   - Use different donor sets for different acceptors
   - Implement gradually over time

3. **Avoid duplicates**:
   - Use `--existing-links-file` with previous recommendations
   - Combine previous output with existing links export

### How accurate is the AI matching?

AI embeddings from OpenAI are highly accurate at understanding semantic similarity. However:

- **Not perfect**: Review a sample before implementing at scale
- **Context matters**: Some nuances may be missed
- **Combine approaches**: Use both AI and field matching for best results
- **Test and iterate**: Adjust parameters based on your specific content

### Can JOIL handle non-English content?

Yes. OpenAI embeddings support multiple languages. JOIL works with any language that OpenAI's embedding models support, which includes most major languages.

### What's the maximum file size JOIL can handle?

JOIL streams CSV files, so it can handle very large datasets:
- **Tested**: Up to 100,000+ pages
- **Practical limit**: Depends on your computer's memory
- **Performance**: Larger datasets take longer to process

If you have performance issues, consider splitting into smaller batches.

### How do I weight certain pages more heavily?

**For acceptors** (pages receiving links):
- Use custom anchors with higher importance values
- Run JOIL multiple times focusing on priority pages
- Manually adjust the output CSV to prioritize certain pages

**For donors** (pages giving links):
- Increase `--max-donor-links` for important donor pages
- Create separate donor CSVs for different priority levels
- Use matching fields to focus on specific donor types

### Can I exclude certain pages from linking?

Yes:

1. **Filter your CSVs**: Remove pages before running JOIL
2. **Use matching fields**: Create a field that excludes unwanted pages
3. **Post-process output**: Remove unwanted recommendations from the output CSV
4. **Use existing links**: Add unwanted page pairs to the existing links file

### What happens if a page is both an acceptor and a donor?

This is fine and common. A page can both give and receive links. JOIL will:
- Include it in matching for both roles
- Prevent it from linking to itself
- Apply both donor and acceptor link limits

### How often should I regenerate link recommendations?

**When to regenerate:**
- After publishing new content (weekly or monthly)
- When site structure changes significantly
- After updating page content substantially
- When adding new product/service pages
- As part of regular SEO maintenance

**Best practice**: Set a regular schedule (monthly or quarterly) to update internal link recommendations.

## Examples

### Example 1: Basic Screaming Frog workflow

Export all HTML pages from Screaming Frog and run JOIL:

```bash
./joil --acceptors-file all_pages.csv --donors-file all_pages.csv
```

Result: `interlinking.csv` with balanced link recommendations

### Example 2: Product pages linking

Link category pages to product pages with strict matching:

```bash
./joil --acceptors-file products.csv \
       --donors-file categories.csv \
       --match-fields "category,brand" \
       --min-ai-distance 0.5 \
       --max-acceptor-links 10
```

Result: Product pages receive up to 10 links from category pages with matching category and brand

### Example 3: Blog to service pages

Link blog posts to service pages using AI similarity only:

```bash
./joil --acceptors-file services.csv \
       --donors-file blog_posts.csv \
       --match-fields "" \
       --enable-ai-sorting \
       --min-ai-distance 0.6 \
       --max-donor-links 3
```

Result: Each blog post gives up to 3 links to semantically related service pages

### Example 4: Equal distribution for new site

Ensure all pages get equal link attention on a new website:

```bash
./joil --acceptors-file pages.csv \
       --donors-file pages.csv \
       --algorithm equal \
       --max-acceptor-links 5 \
       --max-donor-links 5
```

Result: Balanced link distribution across all pages

### Example 5: Custom anchor text with variation

Use custom anchors with natural variation:

```bash
./joil --acceptors-file products.csv \
       --donors-file blog.csv \
       --custom-anchors-file product_anchors.csv \
       --standard-anchor-percent 0.2 \
       --anchor-clean-regexp '®|™'
```

Result: 80% custom anchors, 20% standard anchors, trademark symbols removed

### Example 6: Location-based linking

Link local content using city and state matching:

```bash
./joil --acceptors-file locations.csv \
       --donors-file local_guides.csv \
       --match-fields "city,state" \
       --min-fields-required 1 \
       --anchor-field "h1" \
       --max-acceptor-links 8
```

Result: Location pages receive up to 8 links from guides in matching cities or states

### Example 7: Avoiding duplicate recommendations

Generate new recommendations while respecting existing links:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --existing-links-file current_links.csv \
       --output-file new_recommendations.csv
```

Result: Only new link recommendations that don't already exist

### Example 8: High-volume linking for large sites

Aggressive linking strategy for content-rich sites:

```bash
./joil --acceptors-file all_pages.csv \
       --donors-file all_pages.csv \
       --match-fields "topic,category" \
       --min-fields-required 1 \
       --min-ai-distance 0.7 \
       --max-acceptor-links 15 \
       --max-donor-links 10 \
       --algorithm greedy
```

Result: High-volume linking with quality controls via field matching and AI

## Getting help

### Command-line help

View all available options:

```bash
./joil --help
```

### Check version

Verify your JOIL version:

```bash
./joil --version
```

### Debug mode

Run with detailed logging to troubleshoot issues:

```bash
./joil --acceptors-file acceptors.csv \
       --donors-file donors.csv \
       --debug
```

### Support resources

For additional help:
- Review this manual thoroughly
- Check the examples section for similar use cases
- Verify your CSV files have the correct format
- Test with a small dataset first
- Use debug mode to understand what JOIL is doing
