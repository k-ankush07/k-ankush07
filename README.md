
"""
Banner image ko gradient border + rounded corners ke saath SVG me wrap karta hai,
taaki GitHub README me same look dikhe (README me CSS nahi chalti).

Use:
    python make_banner_svg.py                      # default URL se image le lega
    python make_banner_svg.py main-github.png      # ya local PNG file se

Sirf Python standard library chahiye, kuch install nahi karna.
"""
import base64
import struct
import sys
import urllib.request

SRC = sys.argv[1] if len(sys.argv) > 1 else "https://ankush.bio/assets1/main-github.png"
OUT = "main-github-rounded.svg"

BORDER = 1   # CSS wala padding: 1px
RADIUS = 10  # CSS wala border-radius: 10px

# image load karo
if SRC.startswith("http"):
    req = urllib.request.Request(SRC, headers={"User-Agent": "Mozilla/5.0"})
    data = urllib.request.urlopen(req).read()
else:
    with open(SRC, "rb") as f:
        data = f.read()

if data[:8] != b"\x89PNG\r\n\x1a\n":
    sys.exit("Ye PNG file nahi hai. PNG use karo.")

# PNG header se width/height nikalo
w, h = struct.unpack(">II", data[16:24])
W, H = w + 2 * BORDER, h + 2 * BORDER
b64 = base64.b64encode(data).decode()

svg = f"""<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="{W}" height="{H}" viewBox="0 0 {W} {H}">
  <defs>
    <linearGradient id="g" x1="0" y1="0" x2="1" y2="0">
      <stop offset="0%" stop-color="#8f74bf"/>
      <stop offset="32.21%" stop-color="#d76d77"/>
      <stop offset="100%" stop-color="#ffaf7b"/>
    </linearGradient>
    <clipPath id="c">
      <rect x="{BORDER}" y="{BORDER}" width="{w}" height="{h}" rx="{RADIUS - BORDER}"/>
    </clipPath>
  </defs>
  <rect width="{W}" height="{H}" rx="{RADIUS}" fill="url(#g)"/>
  <image x="{BORDER}" y="{BORDER}" width="{w}" height="{h}" clip-path="url(#c)" xlink:href="data:image/png;base64,{b64}"/>
</svg>
"""

with open(OUT, "w", encoding="utf-8") as f:
    f.write(svg)

print(f"Done: {OUT}  (image size {w}x{h})")
