# Event Stock Manager

A self-hosted inventory management web app for tracking stock across multiple items, variants, and events. It runs entirely on a local machine, so it works without a cloud subscription and is reachable from any device on the same network.

Built as part of an IT internship project for a small non-profit organization.

## Features

- Add, edit, and remove stock items, including variants with unique product codes
- Bulk import and update stock from a standard Excel/CSV template
- Search and filter by item, category, and quantity
- Green-and-white interface with a dark-mode toggle
- Works from a laptop, phone, or tablet on the same WiFi
- Secure remote access through a private VPN mesh (NetBird), with the database kept off the open network

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React + Vite |
| Backend and database | PocketBase (single binary, SQLite) |
| Host | Linux Mint on a low-spec PC (3 GB RAM) |
| Web server | Nginx (serves the built app and proxies the API) |
| Remote access | NetBird |

## Architecture

```
Phone / laptop on WiFi  ──►  Nginx :80  ──►  built React app (static files)
                                     │
                                     └─ /api/ ──►  PocketBase 127.0.0.1:8090 (not exposed to the network)
Remote devices  ──►  NetBird tunnel  ──►  the same Nginx address
```

PocketBase listens only on `127.0.0.1`. The only thing exposed to the network is Nginx, which forwards API requests to PocketBase.

## Documentation

- [Installation guide](docs/INSTALL.md): set it up step by step on Linux Mint or Ubuntu
- [Cloud to local conversion](docs/CONVERSION.md): why and how the project moved off Vercel and Supabase to a local system

## What I learned

- Replacing a hosted backend-as-a-service with a self-hosted one, and mapping queries, auth, and data across
- Serving a Vite production build and reverse-proxying an API with Nginx
- Keeping a backend off the open network while still allowing secure remote access
- Running services reliably with systemd on low-memory hardware
- Cleaning and migrating messy spreadsheet data into a structured database

## License

MIT
