# Sozialindex Düsseldorf – Interaktive Karte

Interaktive Choroplethenkarte der 170 Sozialräume Düsseldorfs mit Daten aus dem Quartiersatlas 2024 und einem Quality-of-Life-Index.

## Funktionen

- Auswahl des angezeigten Indikators: 3 Indizes (Sozialer Handlungsbedarf, Fluktuation, Quality of Life) und 37 Einzelindikatoren in 6 Kategorien (Bevölkerung, Wanderung, Haushalte, Arbeit & Soziales, Bildung, Wohnen & Fläche)
- Tooltip und Popup je Sozialraum mit Vergleich zum Düsseldorfer Durchschnitt (Ø)
- Optional einblendbare Stadtteilgrenzen und Stadtteilnamen
- Export der aktuellen Karte als PDF (A4 quer) oder als Bild (1080×1350, 4:5)

Reine statische Seite (HTML/CSS/JS mit Leaflet 1.9.4 und jsPDF 2.5.1 per CDN), kein Build-Schritt. Die Kartendaten werden per `fetch()` aus `data/sozialraeume.geojson` und `data/stadtteile.geojson` geladen; lokal muss die Seite daher über einen Webserver ausgeliefert werden, z.B.:

```bash
python3 -m http.server 8000
```

## Projektstruktur

```
sozialindex/
├── index.html
├── css/style.css
├── js/app.js
└── data/
    ├── sozialraeume.geojson   # Sozialräume inkl. Indikatoren, Indizes und Durchschnittswerten
    ├── stadtteile.geojson     # Stadtteilgrenzen
    └── averages.json          # Düsseldorfer Durchschnittswerte (die App nutzt die in sozialraeume.geojson eingebettete Kopie)
```

## Deployment (Hetzner / Ubuntu)

### 1. Server vorbereiten

```bash
apt update && apt install -y nginx certbot python3-certbot-nginx git
```

### 2. Repo klonen

```bash
git clone https://github.com/lukasziesemer/sozialindex.git /var/www/sozialindex
```

### 3. Nginx konfigurieren

```bash
cat > /etc/nginx/sites-available/sozialindex <<'EOF'
server {
    listen 80;
    server_name sozialindex.lukas-ziesemer.de;

    root /var/www/sozialindex;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # GeoJSON caching
    location ~* \.geojson$ {
        add_header Cache-Control "public, max-age=86400";
    }

    # Static assets caching
    location ~* \.(css|js|json)$ {
        add_header Cache-Control "public, max-age=604800";
    }

    gzip on;
    gzip_types application/json application/javascript text/css;
}
EOF

ln -sf /etc/nginx/sites-available/sozialindex /etc/nginx/sites-enabled/
rm -f /etc/nginx/sites-enabled/default
nginx -t && systemctl reload nginx
```

### 4. DNS

Einen A-Record für `sozialindex.lukas-ziesemer.de` auf die Server-IP setzen.

### 5. SSL mit Let's Encrypt

```bash
certbot --nginx -d sozialindex.lukas-ziesemer.de
```

### 6. Updates deployen

```bash
cd /var/www/sozialindex && git pull
```

## Datenquellen

- [Quartiersatlas Düsseldorf 2024](https://www.duesseldorf.de/statistik-und-wahlen/statistik-und-stadtforschung/quartiersatlas)
- [Open Data Düsseldorf – Sozialräume GeoJSON](https://opendata.duesseldorf.de/) (Datenlizenz Deutschland – Zero)
- Kartenhintergrund: © OpenStreetMap contributors, © CARTO
- Quality-of-Life-Index: eigene Berechnung
