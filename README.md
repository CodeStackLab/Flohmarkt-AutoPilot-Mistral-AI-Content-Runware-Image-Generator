# Flohmarkt AutoPilot — Mistral AI Content + Runware Image Generator

An automated **n8n workflow** that publishes daily SEO-optimized German blog articles to WordPress using:
- 🤖 **Mistral AI** for content generation
- 🖼️ **Runware AI** for photorealistic image generation

## How the Workflow Works

### Node 1 — Schedule Trigger (Daily 9AM Germany)
Triggers automatically every day at 9:00 AM Berlin time using a cron schedule (`0 9 * * *`).

### Node 2 — Init Run & Pick Topic
- Loads 80 unique Flohmarkt/Trödelmarkt topic slugs
- Checks a permanent exclusion list (stored in n8n static data) to avoid repeating topics
- Shuffles available topics and picks one randomly

### Node 3 — Build Mistral Request
- Constructs a detailed system prompt for Mistral AI
- Enforces: German language only, SEO structure, min. 900 words, HTML format
- Includes recent published titles to avoid duplicate content

### Node 4 — Mistral AI — Generate Content
- Calls Mistral API (`mistral-small-latest`) with the prompt
- Returns: title, slug, meta description, keywords, article HTML, image prompt
- Retries up to 3 times on failure

### Node 5 — Parse Mistral Response
- Validates the returned JSON
- Checks if content is on-topic (rejects off-topic articles)
- Extracts image prompt for Runware

### Node 6 — Content OK? (IF node)
- If content is valid → proceed to image generation
- If content failed → retry with a new topic

### Node 7 — Build Runware Image Request
- Extracts H2 headings from the article
- Builds 3 unique image prompts:
  - **Featured image**: based on full article title (wide shot, 1216×640)
  - **Inline image 1**: based on 2nd H2 heading (medium shot)
  - **Inline image 2**: based on 5th H2 heading (close-up detail shot)

### Node 8 — Runware — Generate Image
- Calls Runware AI API to generate 3 photorealistic JPG images
- Model: `civitai:25694@143906` (realistic photography model)
- Settings: 25 steps, CFG 7, DPMSolverMultistep scheduler

### Node 9 — Parse Runware Response
- Extracts 3 image URLs from the API response
- Ensures uniqueness of all 3 images

### Node 10 — Download Generated Image
- Downloads the featured image as binary data

### Node 11 — Upload Image to WP Media
- Uploads the featured image to WordPress Media Library via REST API
- Returns the WordPress media ID for use as featured image

### Node 12 — Build WP Post
- Checks for duplicate titles using keyword overlap algorithm
- Smartly inserts inline images after relevant H2 sections
- Builds the full WordPress post body with:
  - SEO title, slug, meta description
  - Yoast SEO / Rank Math meta fields
  - Featured image ID
  - Article HTML with 2 inline images

### Node 13 — Publish to WordPress
- POSTs the article to WordPress via REST API (`/wp-json/wp/v2/posts`)
- Status: `publish` (immediately live)
- Retries 3 times on failure

### Node 14 — Save Topic to Exclusion List
- Saves the published topic slug to the permanent exclusion list
- Prevents this topic from being selected again
- Logs: Post ID, Post URL, Topic slug

### Node 15 — Retry — Pick New Topic
- If content validation fails, loops back to Init Run node
- Increments retry counter

## Configuration Required
| Setting | Location | Value |
|---|---|---|
| Mistral API Key | Node: Mistral AI — Generate Content | Bearer token in Authorization header |
| Runware API Key | Node: Runware — Generate Image | Bearer token in Authorization header |
| WordPress URL | Nodes: Upload + Publish | https://your-site.com/wp-json/wp/v2/ |
| WordPress Auth | Nodes: Upload + Publish | Basic Auth (Base64: user:app-password) |

## Troubleshooting
- **Workflow stops at Mistral node**: API key expired or rate limit hit
- **No images generated**: Runware credits exhausted or API key invalid
- **WordPress publish fails**: Check Application Password; regenerate if needed
- **Topics repeat**: Clear `EX_topics` from n8n static data
- **Static data not working**: Requires n8n self-hosted (not n8n.cloud free tier)



---

## Support & Hiring

**Hire our team for setup, please message me:**
* [Fiverr](https://www.fiverr.com/s/EgGm8pq)
* [Upwork](https://www.upwork.com/freelancers/~01b7bb1733953e942f)

**Support & Donations:**
* **PayPal:** [https://paypal.me/khan1899?locale.x=en_GB&country.x=IN](https://paypal.me/khan1899?locale.x=en_GB&country.x=IN)
* **Binance ID:** `538454480`
* **Litecoin (LTC) Address:** `LaJGvzQJGmqfCFkP9cY1kjLp6hphECxWS2` (Network: LTC / Litecoin)
* **Name for Verification:** Mohd Akeel
* **Username:** Mohdakeel1899
