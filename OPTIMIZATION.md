## Optimizations & Images

The `<Image />` component optimizes performance by lazy loading and resizing images.

- **Local Images:** Import the file; dimensions are handled automatically.
- **Remote Images:** Use the `fill` prop for responsive containers or provide `width/height`.
- **Cloudinary/Resizing:** Use an `imageLoader` to let external services handle the heavy lifting.

```js
'use client'
import Image from 'next/image'

const myImageLoader = ({ src, width, quality }) => {
  return `https://example.com/${src}?w=${width}&q=${quality || 75}`
}

export default function HeroImage() {
  return (
    /* The parent MUST have a height and position: relative */
    <div style={{ position: 'relative', width: '100%', height: '400px' }}>
      <Image
        loader={myImageLoader}
        src="images/hero-meal.jpg"
        alt="Hero meal image"
        fill // This replaces width and height
        style={{ objectFit: 'cover' }} // Maintains aspect ratio without stretching
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      />
    </div>
  )
}
```
