# Plex Watchlist Viewer

A lightweight, client-side web application for viewing Plex watchlists via RSS feeds. Features dual-view modes (minimal & fancy) with IMDb/TVDB integration and clipboard functionality.

![Plex Watchlist Viewer](https://img.shields.io/badge/plex-watchlist-00ff41?style=flat-square)
![No Dependencies](https://img.shields.io/badge/dependencies-none-blue?style=flat-square)
![Pure HTML/CSS/JS](https://img.shields.io/badge/tech-vanilla-green?style=flat-square)

## Features

- 📺 **Dual View Modes**: Switch between minimal list and detailed card views
- 🔄 **Auto-Refresh**: Updates every 5 minutes automatically
- 📋 **Clipboard Integration**: Copy individual titles or entire lists
- 🎬 **IMDb/TVDB Links**: Direct links to movie/TV show databases
- 🎨 **Dark Theme**: Easy on the eyes with green accent colors
- ⚡ **Zero Dependencies**: Pure HTML, CSS, and JavaScript

## Demo

Visit the live deployment at: https://watchlist.800801.online (OAuth2 protected)

## Quick Start

### 1. Get Your Plex RSS Feed URLs

To use this viewer, you need your Plex watchlist RSS feed URLs:

1. Log into [Plex.tv](https://www.plex.tv/)
2. Go to your Watchlist
3. Look for the RSS feed option (usually in settings or export options)
4. Copy the RSS feed URL(s)

### 2. Configure the HTML

Open `plex-watchlist-viewer.html` and update the RSS feed URLs on **lines 320-321**:

```javascript
const PERSONAL_RSS = 'https://rss.plex.tv/YOUR-PERSONAL-RSS-ID';
const FRIENDS_RSS = 'https://rss.plex.tv/YOUR-FRIENDS-RSS-ID';
```

### 3. Deploy

#### Option A: Simple (Any Web Server)
```bash
# Just serve the HTML file
python3 -m http.server 8080
# Visit http://localhost:8080
```

#### Option B: nginx (Production)
```bash
# Copy to nginx web root
sudo mkdir -p /var/www/plex-watchlist
sudo cp plex-watchlist-viewer.html /var/www/plex-watchlist/index.html

# Create nginx config
sudo nano /etc/nginx/sites-available/plex-watchlist-subdomain
```

Example nginx configuration with OAuth2 support:
```nginx
server {
    listen 80;
    server_name watchlist.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name watchlist.yourdomain.com;

    # SSL configuration
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # OAuth2 proxy authentication (optional)
    location = /oauth2/auth {
        proxy_pass http://127.0.0.1:4180/oauth2/auth;
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";
    }

    location @oauth2_signin {
        return 302 https://auth.yourdomain.com/oauth2/start?rd=$scheme://$host$request_uri;
    }

    location / {
        # Optional: Enable OAuth2 authentication
        # auth_request /oauth2/auth;
        # error_page 401 = @oauth2_signin;

        root /var/www/plex-watchlist;
        index index.html;
        try_files $uri $uri/ =404;

        # Cache control for static content
        add_header Cache-Control "public, max-age=300";
    }
}
```

## How It Works

1. **Client-Side Only**: Fetches Plex RSS feeds directly from the browser using `fetch()` API
2. **No Backend Needed**: Pure client-side JavaScript parsing of RSS XML
3. **CORS-Friendly**: Plex RSS feeds support cross-origin requests
4. **Responsive Design**: Works on desktop and mobile

## Views

### Minimal View (Default)
- Clean numbered list with type badges `[MOVIE]` / `[TV]`
- Individual copy buttons for each title
- "Copy All Titles" button for bulk operations

### Fancy View
- Detailed cards with metadata
- IMDb/TVDB integration
- Year, category badges, and publication dates
- Hover effects and visual polish

## Technical Details

- **Language**: Vanilla JavaScript (ES6+)
- **Size**: ~17KB (single HTML file)
- **Browser Support**: Modern browsers (Chrome, Firefox, Safari, Edge)
- **Dependencies**: None
- **Framework**: None - pure HTML/CSS/JS

## Configuration

The app displays the **last 25 items** from each feed by default. To change this, modify line 330:

```javascript
const items = Array.from(xml.querySelectorAll('item')).slice(0, 25); // Change 25 to desired count
```

## Privacy & Security

- **Client-Side Processing**: All RSS parsing happens in your browser
- **No Analytics**: No tracking or external services
- **HTTPS Recommended**: Use HTTPS in production to protect RSS feed URLs
- **OAuth2 Ready**: Compatible with OAuth2 proxy protection (see nginx config above)

## Production Deployment

Current production deployment at https://watchlist.800801.online:
- nginx reverse proxy with Google OAuth2 authentication
- SSL/TLS encryption via Let's Encrypt wildcard certificate
- Source file: `/home/tyler/Desktop/mediaserver/plex-watchlist/plex-watchlist-viewer.html`
- Deployed to: `/var/www/plex-watchlist/index.html`
- Part of the [800801.online media server stack](https://github.com/tylerbcrawford/media-server-config)

## License

MIT License - feel free to use and modify!

## Credits

Created by Tyler Crawford ([@tylerbcrawford](https://github.com/tylerbcrawford))

Part of the 800801.online media server infrastructure.
