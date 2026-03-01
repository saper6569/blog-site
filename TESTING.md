# Testing Your Jekyll Blog Locally

## Prerequisites

You need Ruby installed on your Windows machine to test Jekyll locally.

## Step 1: Install Ruby

### Option A: Using RubyInstaller (Recommended for Windows)

1. Download RubyInstaller from: https://rubyinstaller.org/downloads/
2. Download the **Ruby+Devkit** version (recommended: Ruby 3.2.x or 3.3.x)
3. Run the installer and follow the installation wizard
4. **Important**: At the end of installation, check the box to "Run 'ridk install'" to install MSYS2 development toolchain
5. Restart your terminal/PowerShell after installation

### Option B: Using Chocolatey (if you have it)

```powershell
choco install ruby
```

### Verify Installation

Open a new PowerShell window and run:
```powershell
ruby --version
```

You should see something like: `ruby 3.2.x` or `ruby 3.3.x`

## Step 2: Install Bundler

Bundler manages Ruby gem dependencies:

```powershell
gem install bundler
```

## Step 3: Install Jekyll and Dependencies

Navigate to your blog directory and install dependencies:

```powershell
cd "C:\Users\saper\Downloads\blog site"
bundle install
```

This will install Jekyll and all required plugins listed in your `Gemfile`.

**Note**: The first time you run this, it may take a few minutes to download and compile gems.

## Step 4: Run the Development Server

Start the Jekyll development server:

```powershell
bundle exec jekyll serve
```

You should see output like:
```
Configuration file: C:/Users/saper/Downloads/blog site/_config.yml
            Source: C:/Users/saper/Downloads/blog site
       Destination: C:/Users/saper/Downloads/blog site/_site
...
Server address: http://127.0.0.1:4000/
Server running... press ctrl-c to stop.
```

## Step 5: View Your Blog

1. Open your web browser
2. Navigate to: `http://localhost:4000` or `http://127.0.0.1:4000`
3. You should see your blog homepage!

## Testing Tips

- **Auto-reload**: Jekyll automatically rebuilds when you save changes to files
- **Stop the server**: Press `Ctrl+C` in the terminal
- **Check for errors**: If the site doesn't load, check the terminal output for error messages
- **Test all pages**: Navigate through Home, Projects, Blog, and About pages
- **Test blog posts**: Click on the welcome post to see the post layout

## Common Issues

### Issue: "Could not find gem 'jekyll'"
**Solution**: Run `bundle install` again

### Issue: "Port 4000 already in use"
**Solution**: Use a different port:
```powershell
bundle exec jekyll serve --port 4001
```

### Issue: "Liquid syntax error"
**Solution**: Check your Markdown files for syntax errors in the front matter (the YAML section at the top)

### Issue: "Theme not found"
**Solution**: The theme line is commented out in `_config.yml` since we're using custom layouts. This is correct.

## Alternative: Test on GitHub Pages (No Local Setup Required)

If you don't want to install Ruby locally, you can:

1. Push your code to a GitHub repository
2. Enable GitHub Pages in repository settings
3. GitHub will automatically build and host your site
4. Access it at `https://yourusername.github.io/repository-name`

However, local testing is faster for development and debugging.

## Next Steps

Once your blog is working locally:
1. Customize content in `index.md`, `about.md`, and `projects.md`
2. Add more blog posts to `_posts/` directory
3. Add images to `assets/images/`
4. When ready, push to GitHub and enable GitHub Pages
