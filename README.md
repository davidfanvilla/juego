# Juego - Mobile-Optimized Canvas Game

A responsive, mobile-first game built with HTML5 Canvas and optimized for all devices.

## Features

- ✅ **Mobile Optimized**: Fully responsive design with proper viewport configuration
- ✅ **Safe Area Support**: Respects device notches and safe area insets
- ✅ **Centered Canvas**: Canvas properly centered on all screen sizes and orientations
- ✅ **Touch Enabled**: Full touch support for mobile devices
- ✅ **Responsive Layout**: Adapts seamlessly to landscape and portrait modes

## Mobile Compatibility

### Viewport Configuration
The game includes proper viewport settings for optimal mobile display:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=no">
```

### Canvas Centering
The canvas is centered both horizontally and vertically using:
- Flexbox layout for perfect centering
- Dynamic viewport height units (`dvh`) for mobile browser compatibility
- Safe area inset support for devices with notches/cutouts

### CSS Responsive Design
```css
body {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100dvh;
  margin: 0;
  padding: env(safe-area-inset-top) env(safe-area-inset-right) env(safe-area-inset-bottom) env(safe-area-inset-left);
  background-color: #000;
  overflow: hidden;
}

canvas {
  display: block;
  max-width: 100vw;
  max-height: 100dvh;
  object-fit: contain;
}
```

## Setup Instructions

1. Clone the repository:
```bash
git clone https://github.com/davidfanvilla/juego.git
cd juego
```

2. Open `index.html` in your browser or serve using a local server:
```bash
python -m http.server 8000
# or
npx http-server
```

3. For mobile testing, access via your device's IP address:
```
http://YOUR_IP:8000
```

## Browser Support

- ✅ Chrome/Edge 90+
- ✅ Firefox 88+
- ✅ Safari 14+
- ✅ Mobile Safari (iOS 14+)
- ✅ Chrome Mobile
- ✅ Firefox Mobile

## Technical Details

### Orientation Support
The game handles both portrait and landscape orientations with automatic canvas resizing and repositioning.

### Touch Events
Full touch support includes:
- Touch start/end detection
- Multi-touch handling
- Touch-optimized controls

### Performance
- Hardware-accelerated canvas rendering
- Optimized event listeners
- Minimal repaints and reflows

## Development

To modify the game, edit:
- `index.html` - Structure and viewport configuration
- `style.css` - Responsive styling with safe area support
- `script.js` - Game logic and canvas rendering

## Testing

Test on multiple devices and browsers:
- Desktop browsers (Chrome, Firefox, Safari, Edge)
- iOS devices (iPhone, iPad)
- Android devices (various screen sizes)
- Different orientations (portrait, landscape)

## License

MIT License - Feel free to use and modify as needed.

---

**Last Updated**: 2025-12-30

For mobile-first development best practices and viewport configuration, refer to [MDN Web Docs - Viewport Meta Tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Viewport_meta_tag)
