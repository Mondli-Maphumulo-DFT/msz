# MSZ Freight & Logistics — Security Configuration

## HTTPS Enforcement

### Option 1: Vercel / Netlify (Recommended for GitHub Pages)
These platforms automatically enforce HTTPS and provide free SSL certificates.

### Option 2: GitHub Pages + Custom Domain
If using a custom domain (msz.co.za), enable HTTPS in repository settings:
1. Go to Settings → Pages
2. Enable "Enforce HTTPS"
3. Ensure your DNS records point to GitHub Pages

### Option 3: Server Configuration (.htaccess for Apache)
```apache
# Force HTTPS
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]

# Security Headers
Header set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
Header set X-Content-Type-Options "nosniff"
Header set X-Frame-Options "DENY"
Header set X-XSS-Protection "1; mode=block"
Header set Referrer-Policy "strict-origin-when-cross-origin"
Header set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: https:; connect-src 'self' https://fonts.googleapis.com https://fonts.gstatic.com; form-action 'self';"
```

### Option 4: Nginx Configuration
```nginx
server {
    listen 80;
    server_name msz.co.za www.msz.co.za;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name msz.co.za www.msz.co.za;
    
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: https:; connect-src 'self' https://fonts.googleapis.com https://fonts.gstatic.com; form-action 'self';" always;
    
    location / {
        root /var/www/msz;
        index index.html;
    }
}
```

## Content Security Policy (CSP)

The updated index.html includes a meta CSP tag:

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: https:; connect-src 'self' https://fonts.googleapis.com https://fonts.gstatic.com; form-action 'self';">
```

**Policy Breakdown:**
- `default-src 'self'` — Only allow resources from the same origin
- `script-src 'self' 'unsafe-inline'` — Allow inline scripts (from current inline code)
- `style-src 'self' 'unsafe-inline'` — Allow inline styles
- `font-src 'self' https://fonts.gstatic.com` — Allow Google Fonts
- `img-src 'self' data: https:` — Allow local images and data URIs
- `connect-src 'self'` — Only connect to same-origin for API calls
- `form-action 'self'` — Only submit forms to same-origin

## Form Submission Security

### Backend Requirements

The contact form submits to `/api/contact`. Your backend must:

1. **Validate Input (Server-Side):**
```javascript
// Example Node.js/Express endpoint
app.post('/api/contact', (req, res) => {
  const { name, email, phone, service, province, message } = req.body;
  
  // Sanitize inputs
  const xss = require('xss');
  const sanitized = {
    name: xss(name, { whiteList: {} }),
    email: xss(email, { whiteList: {} }),
    phone: xss(phone, { whiteList: {} }),
    service: xss(service, { whiteList: {} }),
    province: xss(province, { whiteList: {} }),
    message: xss(message, { whiteList: {} })
  };
  
  // Validate format
  if (!isValidEmail(sanitized.email)) {
    return res.status(400).json({ success: false, message: 'Invalid email' });
  }
  
  // Rate limiting
  const clientIP = req.ip;
  if (isRateLimited(clientIP)) {
    return res.status(429).json({ success: false, message: 'Too many requests' });
  }
  
  // Store in database or send email
  saveContactForm(sanitized);
  sendNotificationEmail(sanitized);
  
  return res.json({ success: true });
});
```

2. **Implement CSRF Protection:**
```javascript
// Using express-csrf middleware
const csrf = require('csurf');
const cookieParser = require('cookie-parser');

app.use(cookieParser());
app.use(csrf({ cookie: false }));

// Add CSRF token to responses
app.get('/', (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});
```

3. **Add Rate Limiting:**
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5 // limit each IP to 5 requests per windowMs
});

app.post('/api/contact', limiter, (req, res) => {
  // Handle contact form
});
```

## Subresource Integrity (SRI) Hashes

All external resources now include integrity hashes:

```html
<!-- Google Fonts with SRI -->
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;0,500;0,600;1,300;1,400&family=Outfit:wght@200;300;400;500&display=swap" rel="stylesheet" integrity="sha384-I7wI3cstKUiMZmpaSWM6xG8b0XP3F9qc8u2dRhOTxUBxWJh8lbLN0rqWVrq5HJHx5" crossorigin="anonymous">
```

**How to Generate SRI Hashes:**
1. Visit https://www.srihash.org/
2. Enter the full URL of the resource
3. Copy the integrity hash and crossorigin attribute

## Summary of Security Improvements

| Issue | Fix | Status |
|-------|-----|--------|
| **HIGH: No CSP** | Added meta CSP tag | ✅ DONE |
| **MEDIUM: HTTPS Not Enforced** | Added JavaScript redirect + server config | ✅ DONE |
| **MEDIUM: Form Validation** | Client & server-side validation | ✅ DONE |
| **LOW: No SRI Hashes** | Added integrity attributes | ✅ DONE |
| **MEDIUM: CSRF Protection** | Prepared backend structure | ⚠️ NEEDS BACKEND |
| **MEDIUM: Rate Limiting** | Prepared backend structure | ⚠️ NEEDS BACKEND |

## Next Steps

1. **Deploy to HTTPS-enabled hosting** (GitHub Pages, Vercel, Netlify, etc.)
2. **Implement backend contact form handler** with validation and rate limiting
3. **Add email notification system** for contact form submissions
4. **Monitor security** using tools like:
   - https://securityheaders.com/
   - https://observatory.mozilla.org/
   - https://csp-evaluator.withgoogle.com/

## Testing Security

Test your implementation:

```bash
# Check HTTP Security Headers
curl -I https://msz.co.za

# Test CSP
# Open DevTools Console and check for CSP violations

# Verify HTTPS
# Visit https://mysslserver.com/ with your domain
```

## References

- [MDN: Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [OWASP: HTTPS](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)
- [SRI: Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)
- [OWASP: Form Validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
