# Liebermann Pottery - Headless WordPress Site Project Plan

## Overview

Building a stable, long-term static website for Liebermann Pottery using headless WordPress with a custom TypeScript frontend.

**Primary Goal:** Create a site that will remain stable for 10+ years with minimal maintenance.

---

## Tech Stack

- **Backend CMS:** WordPress (content management)
- **Frontend:** TypeScript + Vite + Vanilla JS
- **Styling:** Tailwind CSS
- **Deployment:** GitHub Pages with GitHub Actions
- **WordPress Hosting:** Cheap shared hosting (~$5-10/month)

**Philosophy:** Maximum boring tech for maximum longevity.

---

## Architecture

```
WordPress (Backend CMS)
  ├─ Client edits pottery items
  ├─ Stores content in MySQL
  ├─ Exposes via REST API
  └─ Sends webhook on publish
       ↓
GitHub Actions (Build Pipeline)
  ├─ Triggered by WordPress webhook
  ├─ Fetches content from WP API
  ├─ Builds static site
  └─ Deploys to GitHub Pages
       ↓
GitHub Pages (Static Hosting)
  ├─ Serves static HTML/CSS/JS
  ├─ Custom domain: liebermannpottery.com
  └─ Free HTTPS via Let's Encrypt
```

---

## Phase 1: WordPress Backend Setup

**Time Estimate:** ~2-3 hours

### 1.1 Local Development Environment

- [ ] Install [Local by Flywheel](https://localwp.com/) for local WordPress development
- [ ] Create new site: "liebermann-pottery"
- [ ] Configure basic WordPress settings:
  - Site title: "Liebermann Pottery"
  - Permalinks: Post name (SEO-friendly URLs)
  - Timezone: Set correctly

### 1.2 Install Essential Plugins

**Keep minimal for longevity:**

- [ ] **Advanced Custom Fields (ACF)** - For structured pottery data
- [ ] **Custom Post Type UI** - For creating pottery post type
- [ ] **WP Webhooks** - For triggering GitHub Actions on publish
- [ ] **Disable Comments** - Probably unnecessary for pottery site
- [ ] **UpdraftPlus** - Automated backups (production only)

### 1.3 Create Custom Post Type

**"Pottery" Post Type:**
- Slug: `pottery`
- Public: Yes
- Show in REST API: ✅ **Yes** (critical!)
- Supports: Title, editor, featured image
- Archive: Yes (for gallery page)

**Optional: "Gallery" Post Type** (if separate from pottery items)
- For portfolio/exhibition images
- Show in REST API: Yes

### 1.4 Set Up Custom Fields (ACF)

**For "Pottery" post type:**

| Field Name | Field Type | Required | Notes |
|------------|-----------|----------|-------|
| Price | Number | No | Can be left blank if sold/NFS |
| Dimensions | Text | Yes | e.g., "12" x 8" x 6"" |
| Materials | Text/Select | Yes | e.g., "Stoneware, glaze" |
| Primary Image | Image | Yes | Main product photo |
| Gallery Images | Gallery | No | Additional angles |
| Available | True/False | Yes | In stock vs. sold |
| Featured | True/False | No | Show on homepage |
| Weight | Text | No | For shipping info |
| Year Created | Number | No | For archive/history |

**Important:** Ensure all ACF fields are set to appear in REST API.

### 1.5 Add Test Content

- [ ] Create 8-10 sample pottery items with images
- [ ] Use realistic dimensions, prices, descriptions
- [ ] Test REST API endpoint: `http://liebermann-pottery.local/wp-json/wp/v2/pottery`
- [ ] Verify custom fields appear in `acf` object in API response
- [ ] Test with `?_embed` parameter to get featured images

### 1.6 Content Guidelines Document

Create simple guide for client:

- **Image specifications:**
  - Format: JPG or PNG
  - Minimum size: 1200x1200px
  - Maximum file size: 2MB (compress if larger)
  - Square or landscape orientation works best

- **Description writing:**
  - Be descriptive but concise
  - Include inspiration, technique, unique features
  - Keep under 200 words for each item

---

## Phase 2: Frontend Project Setup

**Time Estimate:** ~1-2 hours

### 2.1 Initialize Vite Project

```bash
npm create vite@latest liebermann-pottery-frontend -- --template vanilla-ts
cd liebermann-pottery-frontend
npm install
```

### 2.2 Install Dependencies

```bash
# Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# TypeScript helpers
npm install -D @types/node

# Deployment
npm install -D gh-pages
```

### 2.3 Project Structure

```
liebermann-pottery-frontend/
├── .github/
│   └── workflows/
│       └── deploy.yml           # GitHub Actions workflow
├── src/
│   ├── main.ts                  # Entry point
│   ├── styles/
│   │   └── main.css             # Tailwind imports + custom styles
│   ├── api/
│   │   └── wordpress.ts         # WP REST API client
│   ├── components/
│   │   ├── PotteryCard.ts       # Pottery item card
│   │   ├── Gallery.ts           # Gallery grid
│   │   ├── Header.ts            # Site header/nav
│   │   ├── Footer.ts            # Site footer
│   │   └── Modal.ts             # Image lightbox
│   ├── utils/
│   │   ├── helpers.ts           # Utility functions
│   │   └── imageLoader.ts       # Lazy loading, optimization
│   ├── types/
│   │   └── pottery.ts           # TypeScript interfaces
│   └── pages/
│       ├── home.ts              # Homepage logic
│       ├── gallery.ts           # Gallery page logic
│       └── about.ts             # About page logic
├── public/
│   └── images/                  # Static images (logo, etc.)
├── index.html                   # Homepage
├── gallery.html                 # Gallery page
├── about.html                   # About page
├── contact.html                 # Contact page (optional)
├── vite.config.ts               # Vite configuration
├── tailwind.config.js           # Tailwind configuration
├── tsconfig.json                # TypeScript configuration
├── .env                         # Local environment variables
├── .env.production              # Production environment variables
├── .gitignore
├── package.json
└── README.md
```

### 2.4 Configure Vite

**vite.config.ts:**
- Set up multi-page app (multiple HTML entry points)
- Configure path aliases for cleaner imports
- Set base URL for GitHub Pages (if using repo subdirectory)
- Configure build output directory as `dist/`

### 2.5 Configure Tailwind CSS

**tailwind.config.js:**
- Add content paths: `./index.html`, `./src/**/*.{js,ts,jsx,tsx}`
- Define custom color palette (pottery/earthy tones):
  - Primary: Warm terracotta/clay colors
  - Secondary: Natural earth tones
  - Accent: Glaze colors
- Add custom fonts if needed (Google Fonts)

**src/styles/main.css:**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom styles here */
```

### 2.6 Environment Variables

**.env (local development):**
```
VITE_WP_API_URL=http://liebermann-pottery.local
```

**.env.production:**
```
VITE_WP_API_URL=https://cms.liebermannpottery.com
```

**Add to .gitignore:**
```
.env
.env.production
```

**Add to GitHub Secrets:**
- `WP_API_URL` - Production WordPress URL

---

## Phase 3: TypeScript Types & API Integration

**Time Estimate:** ~2-3 hours

### 3.1 Define TypeScript Interfaces

**src/types/pottery.ts:**

Define interfaces for:
- `PotteryItem` - Main pottery object from API
- `WPImage` - WordPress media object
- `ACFFields` - Custom field structure
- `WPResponse` - API response wrapper
- `PotteryFilters` - For gallery filtering

Include proper typing for:
- WordPress `_embedded` objects
- Rendered HTML content
- Image size variations
- ACF field groups

### 3.2 WordPress API Client

**src/api/wordpress.ts:**

Functions to implement:
- `fetchAllPottery()` - Get all pottery items with embeds
- `fetchFeaturedPottery()` - Get items where `featured = true`
- `fetchPotteryById(id)` - Get single item
- `fetchAvailablePottery()` - Filter by `available = true`
- Error handling and retry logic
- Response caching (optional, for performance)

**API Parameters to use:**
- `?_embed` - Include featured images
- `?per_page=100` - Get all items (adjust if more)
- `?acf_format=standard` - Get ACF fields in standard format

### 3.3 Error Handling

Implement robust error handling:
- Network errors (API down)
- Parsing errors (malformed JSON)
- Missing data (expected fields don't exist)
- Loading states
- User-friendly error messages

---

## Phase 4: Build Frontend Components

**Time Estimate:** ~4-6 hours

### 4.1 Shared Components

**Header Component:**
- Logo (Liebermann Pottery)
- Navigation: Home, Gallery, About, Contact
- Mobile-responsive hamburger menu
- Active page indication

**Footer Component:**
- Copyright notice
- Social media links (if applicable)
- Contact information
- Credit/attribution

**Pottery Card Component:**
- Pottery image (lazy loaded)
- Title
- Price (if available) or "Sold" badge
- Dimensions
- "View Details" button
- Hover effects

**Modal/Lightbox Component:**
- Full-size image viewer
- Image gallery navigation
- Pottery details overlay
- Close button
- Keyboard navigation (arrow keys, ESC)

### 4.2 Homepage

**Sections:**
1. **Hero Section:**
   - Large feature image or carousel
   - Tagline/mission statement
   - CTA button to gallery

2. **About Snippet:**
   - Brief intro to artist/pottery
   - Photo of artist
   - Link to full About page

3. **Featured Pottery:**
   - Grid of 6-8 featured items
   - Clean, image-focused layout
   - "View All" CTA to gallery

4. **Call-to-Action:**
   - Contact info or newsletter signup
   - Workshop/commission information

### 4.3 Gallery Page

**Features:**
- Grid layout (responsive: 1-2-3-4 columns)
- Filter options:
  - All / Available / Sold
  - By material type (if applicable)
  - By price range
  - By year created
- Sort options:
  - Newest first
  - Price: Low to High
  - Price: High to Low
- Pottery card grid
- Click card to open modal with full details
- Image gallery in modal
- Loading states
- Empty states (no results)

### 4.4 About Page

**Content sections:**
- Artist bio (from WordPress page or hardcoded)
- Studio/process photos
- Artist statement
- Contact information
- Workshop/commission info
- Press/exhibitions (optional)

Can be:
- Fetched from WordPress as a Page
- Or hardcoded (rarely changes)

### 4.5 Accessibility & Performance

**Accessibility:**
- Semantic HTML
- ARIA labels where needed
- Keyboard navigation
- Alt text for all images
- Sufficient color contrast
- Focus states on interactive elements

**Performance:**
- Lazy loading images
- Image optimization (serve appropriate sizes)
- Minimize bundle size
- Fast initial load (<3 seconds)

---

## Phase 5: Production WordPress Setup

**Time Estimate:** ~1-2 hours

### 5.1 Choose WordPress Hosting

**Recommended hosts (in order of boringness):**

1. **SiteGround** (~$15/month)
   - Good performance
   - Easy WordPress installation
   - Good support

2. **Bluehost** (~$10/month)
   - Very common
   - WordPress recommended
   - Decent performance

3. **DreamHost** (~$10/month)
   - Good uptime
   - Generous resources
   - WordPress optimized

**Avoid:**
- GoDaddy (upsells, bloat)
- EIG-owned brands (low quality)

**Requirements:**
- PHP 7.4+
- MySQL 5.7+
- HTTPS (Let's Encrypt)
- SSH access (helpful)
- Automated backups

### 5.2 Deploy WordPress

- [ ] Install WordPress on production host (usually one-click install)
- [ ] Set strong admin password (use password manager)
- [ ] Install same plugins as local
- [ ] Export local WordPress database
- [ ] Import to production (via phpMyAdmin or plugin)
- [ ] Search/Replace URLs: `http://liebermann-pottery.local` → `https://cms.liebermannpottery.com`
- [ ] Upload media files from local
- [ ] Verify REST API works: `https://cms.liebermannpottery.com/wp-json/wp/v2/pottery`

### 5.3 Security Hardening

**Essential security measures:**

- [ ] Change default "admin" username to something unique
- [ ] Install security plugin: **Wordfence** (free) or **Sucuri**
- [ ] Enable two-factor authentication (2FA)
- [ ] Limit login attempts
- [ ] Disable XML-RPC (if not needed)
- [ ] Hide WordPress version
- [ ] Use strong passwords for all users
- [ ] Keep plugins to minimum (fewer attack vectors)
- [ ] Set proper file permissions (644 for files, 755 for directories)

### 5.4 Configure Automated Backups

**Using UpdraftPlus (recommended):**

- [ ] Install UpdraftPlus plugin
- [ ] Configure backup schedule:
  - Database: Weekly
  - Files: Monthly
- [ ] Choose remote storage:
  - Google Drive (free, easy)
  - Dropbox
  - AWS S3 (cheap)
- [ ] Test restore process once
- [ ] Set up email notifications for backup status

### 5.5 WordPress Configuration

**Settings to configure:**

- [ ] Site title and tagline
- [ ] Permalinks: Post name structure
- [ ] Timezone
- [ ] Date/time format
- [ ] Discussion settings (disable comments if not needed)
- [ ] Media settings (image sizes)
- [ ] Reading settings (posts per page)

---

## Phase 6: GitHub & Deployment Setup

**Time Estimate:** ~1-2 hours

### 6.1 Create GitHub Repository

- [ ] Create new public repository: `liebermann-pottery-frontend`
- [ ] Add README with project overview
- [ ] Push local code to GitHub
- [ ] Add `.gitignore` for node_modules, .env, dist/

### 6.2 Configure GitHub Pages

**In GitHub repo settings:**

- [ ] Go to Settings → Pages
- [ ] Source: GitHub Actions (not branch)
- [ ] Custom domain: `liebermannpottery.com`
- [ ] Enforce HTTPS: ✅ (wait for SSL provisioning)

### 6.3 GitHub Actions Workflow

**Create: `.github/workflows/deploy.yml`**

**Workflow should:**
1. Trigger on:
   - Push to main branch (for code changes)
   - Webhook from WordPress (for content changes)
   - Manual trigger option
2. Set up Node.js environment
3. Install dependencies (`npm ci`)
4. Build site (`npm run build`) with production env vars
5. Deploy `dist/` folder to GitHub Pages branch
6. Handle errors gracefully

### 6.4 Add GitHub Secrets

**In repo Settings → Secrets and variables → Actions:**

- [ ] Add `WP_API_URL` - Production WordPress REST API URL
  - Value: `https://cms.liebermannpottery.com`

### 6.5 WordPress Webhook Configuration

**Using WP Webhooks plugin:**

- [ ] Install and activate WP Webhooks
- [ ] Create new webhook:
  - Name: "Trigger GitHub Pages Deploy"
  - Trigger: Post published/updated (pottery post type)
  - URL: GitHub repository dispatch endpoint
  - Method: POST
  - Headers:
    - `Authorization: Bearer YOUR_GITHUB_TOKEN`
    - `Accept: application/vnd.github.v3+json`
  - Body: `{"event_type": "wordpress-update"}`

**Create GitHub Personal Access Token:**
1. GitHub → Settings → Developer settings → Personal access tokens
2. Generate new token (classic)
3. Permissions: `repo` (full control)
4. Copy token and add to WordPress webhook config
5. Store token securely (won't be shown again)

### 6.6 DNS Configuration

**At domain registrar (Namecheap, Google Domains, etc.):**

**For apex domain (liebermannpottery.com):**

Add A records pointing to GitHub Pages:
```
A    @    185.199.108.153
A    @    185.199.109.153
A    @    185.199.110.153
A    @    185.199.111.153
```

**For www subdomain:**

Add CNAME record:
```
CNAME    www    yourusername.github.io
```

**Wait 10-60 minutes for DNS propagation.**

### 6.7 Test Deployment Flow

**Complete end-to-end test:**

1. [ ] Edit a pottery item in WordPress
2. [ ] Click "Update" or "Publish"
3. [ ] Verify webhook sent (check WP Webhooks log)
4. [ ] Check GitHub Actions tab (workflow should be running)
5. [ ] Wait 2-3 minutes for build to complete
6. [ ] Visit liebermannpottery.com
7. [ ] Verify changes are live
8. [ ] Check browser console for errors
9. [ ] Test on mobile device

---

## Phase 7: Client Handoff & Documentation

**Time Estimate:** ~1 hour

### 7.1 Create WordPress Training Materials

**Video tutorial (5-10 minutes):**
- How to log in
- How to add a new pottery item
- How to upload images (proper size/format)
- How to fill in custom fields
- How to publish (and what happens after)
- How to edit existing items
- How to mark items as sold

**Tool:** Loom, Vimeo, or YouTube (unlisted)

### 7.2 Written Documentation

**Create client-facing README:**

**Topics to cover:**
- WordPress login URL and credentials
- How to add/edit pottery items
- Image guidelines (size, format, compression)
- What each custom field means
- Publishing workflow (draft → review → publish)
- How long until changes appear (2-3 minutes)
- What to do if something goes wrong
- Emergency contact information
- How to request new features or changes

**Save as:** Google Doc or PDF, share with client

### 7.3 Technical Documentation

**For future developers (including yourself):**

**README.md in repo should include:**
- Project overview and architecture
- Tech stack with versions
- Local development setup
- Environment variables needed
- Build and deployment process
- Troubleshooting common issues
- Where WordPress is hosted
- Contact information

### 7.4 Password Management

**Secure storage of credentials:**

- [ ] WordPress admin username/password
- [ ] WordPress hosting (cPanel/FTP)
- [ ] Domain registrar
- [ ] GitHub Personal Access Token
- [ ] Backup storage credentials

**Tool:** 1Password, Bitwarden, LastPass

Share necessary credentials with client securely (not via email).

### 7.5 Monitoring Setup

**Free monitoring tools:**

- [ ] **Uptime Robot** - Monitor WordPress uptime
  - Check every 5 minutes
  - Email alert if down
  - URL: https://cms.liebermannpottery.com

- [ ] **GitHub Actions notifications** - Email on failed builds
  - Enabled in repo settings

- [ ] Give client your contact info for issues

---

## Phase 8: Maintenance Plan

### 8.1 Monthly Maintenance (~10 minutes)

**WordPress:**
- [ ] Log in to WordPress admin
- [ ] Check for WordPress core updates → click "Update Now"
- [ ] Check for plugin updates → update all
- [ ] Verify latest backup succeeded (UpdraftPlus dashboard)
- [ ] Quick visual check of site

**Frontend:**
- [ ] Check GitHub Actions for any failed builds
- [ ] Visit site, check for broken images or errors

### 8.2 Quarterly Maintenance (~30 minutes)

- [ ] Review site performance (Google PageSpeed Insights)
- [ ] Check for broken links (use online tool)
- [ ] Review security scan results (Wordfence)
- [ ] Verify SSL certificate is valid
- [ ] Check domain expiration date
- [ ] Review hosting bill/plan

### 8.3 Yearly Maintenance (~2-3 hours)

**Frontend dependencies:**
- [ ] Review npm dependencies for major updates
- [ ] Update dependencies: `npm update`
- [ ] Test site thoroughly after updates
- [ ] Update Node.js version in GitHub Actions if needed
- [ ] Review and update documentation if process changed

**WordPress:**
- [ ] Test backup restoration process
- [ ] Review plugin necessity (remove unused ones)
- [ ] Review user accounts (remove old/unused)
- [ ] Check hosting plan (upgrade if needed)

**Domain:**
- [ ] Renew domain (set to auto-renew)

### 8.4 Maintenance Log

**Keep a simple log:**

| Date | Task | Notes | Time Spent |
|------|------|-------|------------|
| 2025-01-15 | Initial setup | Deployed to production | 20 hours |
| 2025-02-01 | WP update | Updated to 6.x | 5 min |

**Use:** Google Sheets, Notion, or simple text file

---

## Budget Breakdown

### Development Time
- **Phase 1-4 (Development):** 12-15 hours
- **Phase 5-6 (Deployment):** 3-4 hours
- **Phase 7-8 (Documentation/Training):** 2-3 hours
- **Total Development:** ~17-22 hours

### Ongoing Costs (Annual)

| Item | Cost |
|------|------|
| WordPress hosting | $120-180 |
| Domain registration | $15 |
| GitHub Pages | FREE |
| SSL certificate | FREE (via GitHub/Let's Encrypt) |
| **Total Annual** | **$135-195** |

### One-Time Costs
- Domain purchase: ~$15
- Initial development: Your time

---

## Longevity Assessment

### 10-Year Stability Ratings

| Component | Rating | Notes |
|-----------|--------|-------|
| WordPress REST API | 10/10 | Stable since 2016, won't change |
| TypeScript/Vanilla JS | 9/10 | Core web technologies |
| Vite | 8/10 | May need occasional updates |
| Tailwind CSS | 8/10 | V3 is stable, V4+ may change |
| GitHub Pages | 9/10 | Microsoft-backed, very stable |
| WordPress Hosting | 8/10 | Can migrate if needed |

**Overall confidence: ~90% for 10-year stability**

### Risk Mitigation Strategies

**Code Portability:**
- Static files work on any host
- Can export and move instantly
- No vendor lock-in

**WordPress Portability:**
- Standard WordPress installation
- Can migrate to any host
- Database exports are standard

**Minimal Dependencies:**
- Vanilla JS over framework
- Fewer things to break
- Easier to maintain long-term

**Documentation:**
- Everything is documented
- Any developer can take over
- Client can manage content independently

---

## Success Metrics

### Technical Metrics
- [ ] Lighthouse score: 90+ (all categories)
- [ ] Page load time: < 2 seconds
- [ ] Mobile-friendly: Yes (Google test)
- [ ] Images optimized: < 200kb each
- [ ] Accessibility score: 90+
- [ ] Zero console errors

### User Experience Metrics
- [ ] Client can add pottery in < 5 minutes
- [ ] Changes go live in < 3 minutes
- [ ] Client understands the workflow
- [ ] Site works on all major browsers
- [ ] Site works on mobile devices
- [ ] Contact form submissions work

### Business Metrics
- [ ] Client is satisfied with interface
- [ ] Site accurately represents brand
- [ ] Pottery items display correctly
- [ ] Contact information is correct
- [ ] SEO basics implemented (meta tags, alt text)

---

## Troubleshooting Guide

### Common Issues

**Issue: Webhook not triggering GitHub Actions**
- Check WP Webhooks plugin log for errors
- Verify GitHub token has correct permissions
- Check repository dispatch URL is correct
- Test webhook manually in plugin settings

**Issue: Build failing in GitHub Actions**
- Check Actions tab for error logs
- Verify environment variables are set in GitHub Secrets
- Test build locally: `npm run build`
- Check if WordPress API is accessible

**Issue: Images not loading on site**
- Verify WordPress media files uploaded
- Check CORS settings on WordPress host
- Verify image URLs in API response
- Check browser console for errors

**Issue: Changes not appearing after publish**
- Check if webhook triggered (WP Webhooks log)
- Check GitHub Actions completed successfully
- Clear browser cache
- Check if DNS is correct

**Issue: Site down/unreachable**
- Check Uptime Robot alerts
- Verify DNS settings
- Check GitHub Pages status
- Verify WordPress hosting is up

---

## Next Steps

### Immediate Actions (This Week)

1. **Set up local WordPress:**
   - Install Local by Flywheel
   - Create site
   - Install plugins

2. **Create frontend project:**
   - Initialize Vite project
   - Set up Tailwind
   - Create basic structure

3. **Build one complete page:**
   - Start with homepage
   - Get feedback before proceeding

### Week 2-3

4. **Complete all pages:**
   - Gallery with full functionality
   - About page
   - Test end-to-end locally

5. **Production WordPress setup:**
   - Choose and purchase hosting
   - Deploy WordPress
   - Configure security

### Week 4

6. **Deploy to GitHub Pages:**
   - Set up repository
   - Configure GitHub Actions
   - Test webhook flow

7. **Client training:**
   - Record tutorial
   - Write documentation
   - Handoff credentials

---

## Resources & References

### Documentation
- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/)
- [Advanced Custom Fields Documentation](https://www.advancedcustomfields.com/resources/)
- [Vite Documentation](https://vitejs.dev/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

### Tools
- [Local by Flywheel](https://localwp.com/) - Local WordPress development
- [WP REST API Tester](https://wordpress.org/plugins/wp-rest-api-controller/) - Test API responses
- [Postman](https://www.postman.com/) - API testing
- [Uptime Robot](https://uptimerobot.com/) - Uptime monitoring
- [PageSpeed Insights](https://pagespeed.web.dev/) - Performance testing

### Inspiration
- Look at other pottery/ceramics websites for design ideas
- Keep it clean, image-focused
- Let the pottery be the star

---

## Contact & Support

**Developer:** [Your Name]
**Email:** [Your Email]
**Phone:** [Your Phone]

**For Client Issues:**
- WordPress login problems: [Password reset link]
- Site emergency: [Emergency contact]
- General questions: [Email]

**For Technical Issues:**
- GitHub repository: [Repo URL]
- WordPress admin: [WP URL/wp-admin]
- Hosting support: [Host support link]

---

## Project Sign-Off

**Client Approval:**
- [ ] Design approved
- [ ] Content structure approved
- [ ] Training completed
- [ ] Documentation received
- [ ] Credentials received
- [ ] Site launched

**Developer Handoff:**
- [ ] All phases completed
- [ ] Documentation complete
- [ ] Client trained
- [ ] Monitoring set up
- [ ] Maintenance plan explained

---

**Project Start Date:** _____________

**Expected Completion:** _____________

**Actual Completion:** _____________

---

*This plan prioritizes longevity and stability over cutting-edge features. The goal is a site that works reliably for 10+ years with minimal maintenance.*