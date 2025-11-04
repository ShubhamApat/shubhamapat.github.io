# AGENTS.md - Hugo Portfolio Site Guidelines

## Build Commands
- **Build site**: `hugo` - Generate static site in `public/` directory
- **Serve locally**: `hugo server -D` - Start development server with drafts
- **Clean build**: `hugo --gc --minify` - Build with garbage collection and minification

## Code Style Guidelines

### Content (Markdown)
- Use YAML frontmatter with `title`, `date`, `tags`, `description`
- Follow Hugo's content organization: `content/post/`, `content/projects/`
- Use descriptive, SEO-friendly titles and tags
- Include cover images in `/images/` directory

### HTML Templates (Go Templates)
- Use Hugo's template syntax: `{{ .Title }}`, `{{- range .Params.tags }}`
- Follow PaperMod theme conventions for partials and layouts
- Maintain semantic HTML structure

### CSS/SCSS
- Extend PaperMod theme in `assets/css/extended/custom.css`
- Use CSS custom properties for theming (`var(--entry)`, `var(--radius)`)
- Implement responsive design with mobile-first approach
- Use Flexbox for layout components

### Naming Conventions
- Files: kebab-case for content (`my-first-project.md`)
- Directories: lowercase (`content/post/`, `static/images/`)
- CSS classes: kebab-case with BEM-like structure (`home-info-left`)

### Error Handling
- Validate Hugo builds with `hugo --minify` before deployment
- Check for broken links and missing images
- Ensure responsive design works across devices