# Stellar Tails Universe Website

A responsive, single-page website showcasing the Stellar Tails furry-themed sci-fi universe with a modern, dark sci-fi aesthetic.

## Features

- **Responsive Design**: Works seamlessly on desktop, tablet, and mobile devices
- **Modern Sci-Fi Aesthetic**: Dark color palette with cyan, magenta, and yellow accents
- **Interactive Elements**: Smooth scrolling, parallax effects, and hover animations
- **Character Gallery**: Meet the diverse cast of the Stellar Tails universe
- **Image Hosting**: Dedicated gallery page for embedding images in external sites
- **Docker Support**: Containerized deployment with nginx
- **Performance Optimized**: Gzip compression, caching headers, and minified assets

## Quick Start

### Using Docker (Recommended)

1. **Build the Docker image:**
   ```bash
   docker build -t stellar-tails .
   ```

2. **Run the container:**
   ```bash
   docker run -d -p 8080:80 --name stellar-tails-website stellar-tails
   ```

3. **Access the website:**
   Open your browser and navigate to `http://localhost:8080`

### Manual Setup

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd stellar-tails
   ```

2. **Serve the files:**
   Using Python:
   ```bash
   python -m http.server 8080
   ```
   
   Using Node.js (if you have http-server installed):
   ```bash
   npx http-server -p 8080
   ```

3. **Access the website:**
   Open your browser and navigate to `http://localhost:8080`

## Project Structure

```
stellar-tails/
├── index.html              # Main landing page
├── gallery.html            # Image gallery for hosting
├── styles.css              # Main stylesheet with sci-fi theme
├── script.js               # Interactive JavaScript features
├── Dockerfile              # Docker configuration
├── nginx.conf              # Nginx server configuration
├── README.md               # This file
└── assets/
    └── images/             # Character and logo images
        ├── StellarTails.png
        ├── Captain Tiberius Tibbs Kane.png
        ├── Captain Isabelle Izzy Corvo.png
        └── ... (other character images)
```

## Website Sections

### Home Page (`index.html`)

- **Hero Section**: Animated landing area with typing effect and parallax stars
- **Universe Section**: Information about the Stellar Tails universe set in 2385
- **Characters Section**: Grid layout showcasing all characters with hover effects
- **Navigation**: Smooth scrolling between sections with mobile hamburger menu

### Gallery Page (`gallery.html`)

- **Image Grid**: Browse all character images in grid or list view
- **Search Functionality**: Filter images by name
- **Copy URLs**: One-click copying of image URLs for external embedding
- **Modal Preview**: Click images for full-size preview
- **Responsive Design**: Adapts to different screen sizes

## Docker Configuration

The website uses nginx for serving static files with the following optimizations:

- **Gzip Compression**: Reduces file sizes for faster loading
- **Caching Headers**: Proper cache control for different file types
- **Security Headers**: XSS protection, content type options, and frame options
- **Health Check**: `/health` endpoint for monitoring

## GitHub CI/CD Pipeline

### Setting up GitHub Actions

1. **Create the workflow directory:**
   ```bash
   mkdir -p .github/workflows
   ```

2. **Create the CI/CD workflow file:**
   ```yaml
   # .github/workflows/deploy.yml
   name: Build and Deploy Docker Image
   
   on:
     push:
       branches: [ main, develop ]
     pull_request:
       branches: [ main ]
   
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
       - uses: actions/checkout@v3
       
       - name: Validate HTML
         run: |
           # Install HTML validator
           sudo apt-get update
           sudo apt-get install -y html5validator
           # Validate HTML files
           html5validator --root . --match "*.html"
       
       - name: Check CSS
         run: |
           # Install CSS validator
           npm install -g css-validator
           # Validate CSS
           css-validator styles.css
       
       - name: Lint JavaScript
         run: |
           # Install ESLint
           npm install -g eslint
           # Create .eslintrc.json if it doesn't exist
           echo '{"env":{"browser":true,"es2021":true},"extends":"eslint:recommended","parserOptions":{"ecmaVersion":12,"sourceType":"module"},"rules":{}}' > .eslintrc.json
           # Lint JavaScript
           eslint script.js
   
     build:
       needs: test
       runs-on: ubuntu-latest
       if: github.ref == 'refs/heads/main'
       
       steps:
       - uses: actions/checkout@v3
       
       - name: Set up Docker Buildx
         uses: docker/setup-buildx-action@v2
       
       - name: Login to Docker Hub
         uses: docker/login-action@v2
         with:
           username: ${{ secrets.DOCKER_USERNAME }}
           password: ${{ secrets.DOCKER_PASSWORD }}
       
       - name: Build and push Docker image
         uses: docker/build-push-action@v4
         with:
           context: .
           push: true
           tags: |
             ${{ secrets.DOCKER_USERNAME }}/stellar-tails:latest
             ${{ secrets.DOCKER_USERNAME }}/stellar-tails:${{ github.sha }}
           cache-from: type=gha
           cache-to: type=gha,mode=max
   
     deploy:
       needs: build
       runs-on: ubuntu-latest
       if: github.ref == 'refs/heads/main'
       
       steps:
       - uses: actions/checkout@v3
       
       - name: Deploy to production
         uses: appleboy/ssh-action@v0.1.5
         with:
           host: ${{ secrets.PRODUCTION_HOST }}
           username: ${{ secrets.PRODUCTION_USER }}
           key: ${{ secrets.PRODUCTION_SSH_KEY }}
           script: |
             cd /opt/stellar-tails
             docker-compose pull
             docker-compose up -d
             docker system prune -f
   ```

3. **Create Docker Compose file for production:**
   ```yaml
   # docker-compose.yml
   version: '3.8'
   
   services:
     stellar-tails:
       image: your-docker-username/stellar-tails:latest
       ports:
         - "80:80"
         - "443:443"
       restart: unless-stopped
       volumes:
         - ./nginx-logs:/var/log/nginx
       environment:
         - NODE_ENV=production
   ```

### Required GitHub Secrets

Set up these secrets in your GitHub repository settings:

- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password or access token
- `PRODUCTION_HOST`: Your production server IP/hostname
- `PRODUCTION_USER`: SSH username for production server
- `PRODUCTION_SSH_KEY`: Private SSH key for production server

### Alternative CI/CD with GitHub Pages

If you prefer using GitHub Pages instead of Docker:

1. **Create a GitHub Pages workflow:**
   ```yaml
   # .github/workflows/pages.yml
   name: Deploy to GitHub Pages
   
   on:
     push:
       branches: [ main ]
   
   jobs:
     deploy:
       runs-on: ubuntu-latest
       permissions:
         pages: write
         id-token: write
       
       steps:
       - uses: actions/checkout@v3
       
       - name: Setup Pages
         uses: actions/configure-pages@v3
       
       - name: Upload artifact
         uses: actions/upload-pages-artifact@v2
         with:
           path: '.'
       
       - name: Deploy to GitHub Pages
         id: deployment
         uses: actions/deploy-pages@v2
   ```

2. **Configure GitHub Pages:**
   - Go to repository Settings → Pages
   - Source: GitHub Actions
   - The workflow will automatically deploy your site

## Environment Variables

The website supports the following environment variables:

- `NODE_ENV`: Set to `production` for production optimizations
- `PORT`: Port number for the server (default: 80)

## Browser Support

- Chrome/Chromium 60+
- Firefox 55+
- Safari 12+
- Edge 79+

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature-name`
3. Make your changes and test thoroughly
4. Commit your changes: `git commit -m 'Add some feature'`
5. Push to the branch: `git push origin feature-name`
6. Open a pull request

## Performance Optimization

The website includes several performance optimizations:

- **Lazy Loading**: Images load as needed
- **Gzip Compression**: Reduces file transfer sizes
- **Minified Assets**: CSS and JS are optimized
- **CDN Ready**: Can be easily deployed to CDN services
- **Progressive Enhancement**: Works without JavaScript

## Security Features

- **Content Security Policy**: Prevents XSS attacks
- **HTTPS Ready**: SSL/TLS configuration included
- **Secure Headers**: Additional security headers configured
- **Input Validation**: Form inputs are validated and sanitized

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues and questions:
- Create an issue in the GitHub repository
- Check the troubleshooting section below
- Review the documentation for common solutions

## Troubleshooting

### Docker Issues

**Problem:** Container fails to start
**Solution:** Check if port 80 is already in use:
```bash
sudo lsof -i :80
```

**Problem:** Images not loading
**Solution:** Verify the assets/images directory structure and file permissions

### Build Issues

**Problem:** CSS not applying correctly
**Solution:** Check for syntax errors in styles.css and verify file paths

**Problem:** JavaScript errors in console
**Solution:** Use browser developer tools to debug script.js issues

### Deployment Issues

**Problem:** GitHub Actions failing
**Solution:** Check secrets configuration and workflow syntax

**Problem:** Site not accessible after deployment
**Solution:** Verify DNS settings and server configuration

---

Built with ❤️ for the Stellar Tails universe