pwa-icon-generator.skill
YAML
name: pwa-icon-generator
description: Generate PWA icons programmatically using Python (Pillow) and set up PWA manifest/service worker. Use for creating PWA assets, generating icons for web apps, setting up offline capabilities for websites.
PWA Icon Generator & Setup
This skill automates the creation of PWA (Progressive Web App) icons and provides the necessary configuration files to make a web application installable.
Workflow
1. Generate Icons
Use the provided Python script to generate high-quality, programmatically drawn icons in 192x192 and 512x512 sizes.
Bash
python3 /home/ubuntu/skills/pwa-icon-generator/scripts/generate_icons.py --name "MYAPP" --theme "#c8502a" --bg "#0d0d0f" --outdir ./public
Programmatic Design: The script uses Pillow to draw a techy, orbital-style icon.
Scaling: All elements are scaled based on the target size to ensure sharpness.
Maskable Support: Designed with a safe zone to work well as a "maskable" icon on Android.
2. Configure PWA Files
Copy and customize the templates from /home/ubuntu/skills/pwa-icon-generator/templates/pwa_configs.md:
manifest.json: Defines the app name, colors, and icon paths.
sw.js: A basic Service Worker with a "Cache First" strategy for offline support.
index.html: Integration code for the <head> and Service Worker registration.
3. Verification
Use Chrome DevTools > Application tab to verify the Manifest and Service Worker.
Ensure the site is served over HTTPS (required for PWA installation).
Key Principles
No Hardcoding: Always calculate positions as a percentage of size (e.g., int(size * 0.22)).
Safe Zone: Keep important icon content within the center 80% to support various OS masking shapes.
Offline Ready: Always include a Service Worker to enable the "Add to Home Screen" prompt.
