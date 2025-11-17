# Vercel Deployment Guide for Oh My Git!

This guide explains how to deploy the Oh My Git! game to Vercel.

## Prerequisites

1. **Godot 3.x** installed on your local machine (download from [godotengine.org](https://godotengine.org/download/3.x))
2. **Vercel CLI** (optional, for command-line deployment)
3. A **Vercel account** (free tier works fine)

## Deployment Steps

### 1. Build the HTML5 Version

First, you need to export the game to HTML5 using Godot:

#### Option A: Using the Makefile (Recommended)

```bash
make html5
```

This will create the HTML5 build in `build/html5/` directory.

#### Option B: Using Godot Editor

1. Open the project in Godot 3
2. Go to **Project > Export**
3. Select the **HTML5** preset
4. Set export path to `build/html5/index.html`
5. Click **Export Project**

#### Option C: Using Godot CLI

```bash
godot --export "HTML5" "build/html5/index.html"
```

### 2. Verify the Build

After building, your `build/html5/` directory should contain:
- `index.html` - Main HTML file
- `*.wasm` - WebAssembly binary
- `*.pck` - Godot packed resources
- Other supporting files

### 3. Deploy to Vercel

#### Option A: Using Vercel CLI

1. Install Vercel CLI (if not already installed):
   ```bash
   npm install -g vercel
   ```

2. Deploy from the project root:
   ```bash
   vercel --prod
   ```

3. Follow the prompts to link your project and deploy

#### Option B: Using Vercel Dashboard

1. Go to [vercel.com](https://vercel.com) and log in
2. Click **"Add New Project"**
3. Import your Git repository
4. Vercel will automatically detect the `vercel.json` configuration
5. Click **"Deploy"**

**Important**: Make sure to build the HTML5 version and commit the `build/html5/` directory before deploying, as Vercel doesn't have Godot installed to build it automatically.

#### Option C: GitHub Actions (Automated)

For automatic builds on every push, you can set up GitHub Actions. Create `.github/workflows/deploy.yml`:

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Install Godot
        run: |
          wget https://downloads.tuxfamily.org/godotengine/3.5.3/Godot_v3.5.3-stable_linux_headless.64.zip
          unzip Godot_v3.5.3-stable_linux_headless.64.zip
          sudo mv Godot_v3.5.3-stable_linux_headless.64 /usr/local/bin/godot
          sudo chmod +x /usr/local/bin/godot

      - name: Export HTML5
        run: make html5

      - name: Commit build
        run: |
          git config user.name github-actions
          git config user.email github-actions@github.com
          git add build/html5/
          git commit -m "Build HTML5 version" || echo "No changes"
          git push

      - name: Deploy to Vercel
        env:
          VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
        run: |
          npm install -g vercel
          vercel --prod --token=$VERCEL_TOKEN
```

### 4. Configuration Files

The following files have been configured for Vercel deployment:

- **`vercel.json`**: Vercel configuration with routing and headers
- **`package.json`**: Project metadata
- **`.vercelignore`**: Files to exclude from deployment
- **`export_presets.cfg`**: Updated with HTML5 export path
- **`Makefile`**: Added `html5` target for building

### 5. Important Notes

#### Cross-Origin Headers

The `vercel.json` includes necessary headers for WebAssembly:
- `Cross-Origin-Embedder-Policy: require-corp`
- `Cross-Origin-Opener-Policy: same-origin`

These are required for SharedArrayBuffer support in modern browsers.

#### Content Types

Proper MIME types are configured for:
- `.wasm` files: `application/wasm`
- `.pck` files: `application/octet-stream`

#### Build Directory

The HTML5 build must be in the `build/html5/` directory. Don't forget to:
1. Build the HTML5 version locally
2. Commit the `build/html5/` directory to your repository
3. Push to trigger deployment (if using Git integration)

### 6. Updating the Game

When you make changes to the game:

1. Make your changes to the source files
2. Rebuild the HTML5 version: `make html5`
3. Commit both source changes and the new build
4. Push to your repository
5. Vercel will automatically redeploy (if using Git integration)

### 7. Custom Domain (Optional)

To use a custom domain:

1. Go to your project in Vercel dashboard
2. Navigate to **Settings > Domains**
3. Add your custom domain
4. Update your DNS records as instructed

## Troubleshooting

### Game doesn't load
- Check browser console for errors
- Verify all files are in `build/html5/`
- Ensure CORS headers are properly set

### Build fails
- Make sure Godot 3.x is installed (not Godot 4)
- Verify the export templates are installed
- Check that the HTML5 preset exists in `export_presets.cfg`

### Files not found
- Ensure `build/html5/` directory is committed to git
- Check `.vercelignore` isn't excluding necessary files
- Verify the routing in `vercel.json`

## Support

For issues specific to:
- **Oh My Git! game**: [GitHub Issues](https://github.com/git-learning-game/oh-my-git/issues)
- **Vercel deployment**: [Vercel Documentation](https://vercel.com/docs)
- **Godot export**: [Godot Documentation](https://docs.godotengine.org/en/3.5/tutorials/export/exporting_for_web.html)
