# 🚀 5-Minute Quickstart

Get from zero to a live API in under 5 minutes.

---

## Step 1 — Install (1 min)

```bash
docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/Data \
  --name crudapi \
  ghcr.io/websvcin/crudapi:latest
```

Visit http://localhost:8080 — landing page should load.

---

## Step 2 — Login to Admin (30 sec)

Click **"Admin Console"** on the landing page.

- Username: `admin`
- Password: `admin@123`

**Immediately change** the password at `/admin/adminusers`.

---

## Step 3 — Register a MySQL database (1 min)

1. Click **Databases** in left nav
2. Click **+ Register Database**
3. Fill in:
   - **DB Key**: `master`
   - **Name**: `My First DB`
   - Pick **MySQL** as the engine
   - **Host**, **Port**, **DB Name**, **User**, **Password**
4. Click **🔌 Test Connection** — green means ready
5. Click **Register**

---

## Step 4 — Create an API key (30 sec)

1. Click **API Clients** in left nav
2. Click **+ Create API Client**
3. Pick the database you just registered
4. Pick a role: **Editor** (full CRUD)
5. Click **Create**
6. **Copy the API key** — you only see it once. `drk_abc123...`

---

## Step 5 — Call your data (1 min)

```bash
curl http://localhost:8080/api/v1/master/crud/customers \
  -H "X-API-Key: drk_yourkey..."
```

JSON with all rows. ✅

---

## Step 6 — Try filters

```bash
# Filter
curl "http://localhost:8080/api/v1/master/crud/customers?filter=country%20eq%20'India'" \
  -H "X-API-Key: drk_..."

# Sort
curl "http://localhost:8080/api/v1/master/crud/customers?orderby=created_at%20desc" \
  -H "X-API-Key: drk_..."

# Paginate
curl "http://localhost:8080/api/v1/master/crud/customers?limit=10&offset=0" \
  -H "X-API-Key: drk_..."
```

Full operator list at http://localhost:8080/help#filters.

---

## 🎉 You're done

You now have:
- ✅ A REST API for any MySQL table
- ✅ Authentication via API key
- ✅ Filtering, sorting, pagination
- ✅ Admin UI for managing it all

**Next steps:**
- Browse http://localhost:8080/help for all features
- Set up production deployment with [DEPLOY.md](DEPLOY.md)
