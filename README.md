# Psyduck's Lair - Personal Portfolio & Blog

A professional DevOps engineer's portfolio and blog built with Hugo static site generator and the PaperMod theme.

## 🚀 Features

- **Professional Resume Page** with collapsible sections
- **Certificate Badges** with official PNG images
- **Copy-to-Clipboard** functionality for contact information
- **Technical Blog Posts** covering DevOps topics
- **Responsive Design** optimized for all devices
- **Dark/Light Mode** support via PaperMod theme
- **SEO Optimized** with proper meta tags and structure

## 📋 Prerequisites

Before running this project locally, ensure you have the following installed:

### Required Software

1. **Hugo Extended** (v0.148.2 or later)
   - Download from: https://github.com/gohugoio/hugo/releases
   - Make sure to get the **extended** version for SCSS support

2. **Git** (for cloning and theme management)
   - Download from: https://git-scm.com/downloads

3. **Go** (optional, for Hugo modules)
   - Download from: https://golang.org/dl/

### Installation Verification

Verify your installations:

```bash
# Check Hugo version (should show extended)
hugo version

# Check Git
git --version

# Check Go (optional)
go version
```

## 🛠️ Local Setup Instructions

### Step 1: Clone the Repository

```bash
# Clone the repository
git clone https://github.com/mahesh1b/mahesh1b.github.io.git

# Navigate to project directory
cd mahesh1b.github.io
```

### Step 2: Initialize Git Submodules

The PaperMod theme is included as a Git submodule:

```bash
# Initialize and update submodules
git submodule init
git submodule update

# Alternative: Clone with submodules in one command
# git clone --recurse-submodules https://github.com/mahesh1b/mahesh1b.github.io.git
```

### Step 3: Install Dependencies

No additional dependencies are required as Hugo handles everything internally.

### Step 4: Run the Development Server

```bash
# Start the Hugo development server
hugo server

# Alternative with additional options
hugo server --buildDrafts --buildFuture --bind 0.0.0.0 --port 1313
```

### Step 5: Access the Site

Open your browser and navigate to:
- **Local URL**: http://localhost:1313
- **Network URL**: http://[your-ip]:1313 (if using --bind 0.0.0.0)

## 📁 Project Structure

```
mahesh1b.github.io/
├── archetypes/          # Content templates
├── content/             # Site content
│   ├── blogs/          # Blog posts
│   ├── tldr/           # Quick reference guides
│   ├── about-me.md     # About page
│   ├── dotfiles.md     # Configuration files
│   └── resume.md       # Resume page
├── layouts/             # Custom layouts
│   ├── _default/       # Default layouts
│   │   └── resume.html # Custom resume layout
│   └── shortcodes/     # Custom shortcodes
│       └── certificates.html
├── static/              # Static assets
│   ├── images/         # Images and media
│   │   └── badges/     # Certificate badge PNGs
│   └── assets/         # Other static files
├── themes/              # Hugo themes
│   └── PaperMod/       # PaperMod theme (submodule)
├── hugo.yaml           # Hugo configuration
├── .gitignore          # Git ignore rules
└── README.md           # This file
```

## ⚙️ Configuration

### Hugo Configuration (`hugo.yaml`)

Key configuration options:

```yaml
baseURL: "https://mahesh1b.github.io/"
languageCode: "en-us"
title: "Psyduck's Lair"
theme: "PaperMod"

params:
  env: production
  title: "Psyduck's Lair"
  description: "DevOps Engineer's Portfolio & Blog"
  keywords: [Blog, Portfolio, PaperMod]
  author: "Mahesh Bhosle"
  
  # Theme settings
  ShowReadingTime: true
  ShowShareButtons: true
  ShowPostNavLinks: true
  ShowBreadCrumbs: true
  ShowCodeCopyButtons: true
  ShowWordCount: true
  ShowRssButtonInSectionTermList: true
  UseHugoToc: true
  disableSpecial1stPost: false
  disableScrollToTop: false
  comments: false
  hidemeta: false
  hideSummary: false
  showtoc: false
  tocopen: false
```

### Adding New Content

#### Create a New Blog Post

```bash
# Create a new blog post
hugo new blogs/my-new-post.md

# Edit the created file
# content/blogs/my-new-post.md
```

#### Create a New TLDR Guide

```bash
# Create a new TLDR guide
hugo new tldr/my-guide.md

# Edit the created file
# content/tldr/my-guide.md
```

## 🎨 Customization

### Resume Page

The resume page uses a custom layout with:
- **Collapsible sections** for better organization
- **Certificate badges** from PNG files
- **Copy-to-clipboard** functionality for contact info
- **Responsive design** for all devices

### Certificate Badges

To add new certificate badges:

1. Add PNG files to `static/images/badges/`
2. Update the `certificates` array in `content/resume.md` front matter:

```yaml
certificates:
  - image: "/images/badges/your-cert.png"
    alt: "Your Certification Name"
    title: "Your Certification Title"
```

### Theme Customization

The site uses the PaperMod theme with minimal customizations:
- Custom resume layout in `layouts/_default/resume.html`
- Certificate shortcode in `layouts/shortcodes/certificates.html`
- All styling uses PaperMod's CSS variables for consistency

## 🚀 Deployment

### GitHub Pages (Recommended)

This site is configured for GitHub Pages deployment:

1. **Push to GitHub**: All changes pushed to the main branch
2. **Automatic Build**: GitHub Actions builds and deploys the site
3. **Live Site**: Available at https://mahesh1b.github.io

### Manual Build

To build the site manually:

```bash
# Build the site
hugo

# Output will be in the 'public' directory
# Deploy the contents of 'public' to your web server
```

### Build with Different Environment

```bash
# Build for production
hugo --environment production

# Build with drafts and future posts
hugo --buildDrafts --buildFuture

# Build and minify
hugo --minify
```

## 🔧 Troubleshooting

### Common Issues

#### Hugo Version Issues
```bash
# Check if you have Hugo Extended
hugo version

# Should show something like:
# hugo v0.148.2+extended+withdeploy darwin/arm64
```

#### Submodule Issues
```bash
# If PaperMod theme is missing
git submodule update --init --recursive

# Force update submodules
git submodule update --remote --force
```

#### Port Already in Use
```bash
# Use a different port
hugo server --port 1314

# Or kill the process using port 1313
lsof -ti:1313 | xargs kill -9
```

#### Certificate Badges Not Showing
1. Ensure PNG files are in `static/images/badges/`
2. Check file names match the `certificates` array in `resume.md`
3. Verify the shortcode is properly called: `{{< certificates >}}`

### Performance Tips

```bash
# Fast render mode (for development)
hugo server --disableFastRender

# Enable live reload
hugo server --liveReloadPort 1313

# Verbose output for debugging
hugo server --verbose
```

## 📝 Content Guidelines

### Blog Posts

- Use descriptive titles and tags
- Include a summary in the front matter
- Add cover images when relevant
- Use proper markdown formatting

### TLDR Guides

- Keep content concise and actionable
- Use code blocks for commands
- Include practical examples
- Add relevant tags for discoverability

## 🤝 Contributing

This is a personal portfolio, but suggestions and improvements are welcome:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

## 📞 Contact

- **Email**: mahesh22071999@gmail.com
- **LinkedIn**: [Mahesh Bhosle](https://linkedin.com/in/mahesh-bhosle-301a3619b)
- **GitHub**: [mahesh1b](https://github.com/mahesh1b/)

---

**Happy Coding! 🚀**

*Built with ❤️ using Hugo and PaperMod theme*