# 🚀 5-Minute Quickstart

Get from zero to a live API in under 5 minutes.

Full detail behind every step below lives in the [docs site](https://websvcin.github.io/crudapi/getting-started/quickstart.html).

---

## Step 1 — Install (1 min)

```bash
docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/App_Data \
  --name crudapi \
  ghcr.io/websvcin/crudapi:latest
```

Visit http://localhost:8080 — landing page should load.

---

## Step 2 — Create your admin account (30 sec)

Click **"Admin Console"** on the landing page — you'll land on **`/setup`**. There's no default
login; this wizard creates the real first admin account, so pick your own username and password
here.

---

## Step 3 — Create a tenant (30 sec)

Everything else lives under a **tenant** — one customer/workspace, with its own data, keys and
people, completely separate from every other tenant's.

1. Sign in to `/admin`, click **Tenants** in the sidebar.
2. Click **+ Create Tenant**, give it an id, a name, and a slug (the slug is what appears in its
   sign-in address).
3. Open the tenant you just created and click **🔑 Console key** — copy it, shown once.

---

## Step 4 — Sign in to the tenant workspace and register a database (1 min)

1. Go to `/{your-slug}/login`, choose the **API Key** tab, and paste in the console key.
2. In the workspace, go to **Databases → + Register a database**.
3. Fill in the connection details for a MySQL database you already run:
   - **DB Key**: `master`
   - **Name**: `My First DB`
   - **Host**, **Port**, **DB Name**, **User**, **Password**
4. Click **Test connection** — green means ready — then **Register**. Nothing is stored until the
   test passes.

---

## Step 5 — Create an API key (30 sec)

Still in the tenant workspace, go to **Keys → + Create API key**:

1. Pick the database you just registered.
2. Pick a role — **Editor** gives full CRUD.
3. Click **Create** and **copy the key** — you only see it once. `drk_abc123...`

---

## Step 6 — Call your data (1 min)

```bash
curl http://localhost:8080/api/v1/your-tenant-slug/master/crud/customers \
  -H "X-API-Key: drk_yourkey..."
```

JSON with all rows. ✅ Note the URL shape: `/api/v1/{tenant}/{database}/crud/{table}` — the tenant
slug always comes first.

---

## Step 7 — Try filters and sorting

```bash
# Filter
curl "http://localhost:8080/api/v1/your-tenant-slug/master/crud/customers?filter=country%20eq%20'India'" \
  -H "X-API-Key: drk_..."

# Sort — column:asc or column:desc, comma-separate for more than one
curl "http://localhost:8080/api/v1/your-tenant-slug/master/crud/customers?orderBy=created_at:desc" \
  -H "X-API-Key: drk_..."

# Paginate
curl "http://localhost:8080/api/v1/your-tenant-slug/master/crud/customers?limit=10&offset=0" \
  -H "X-API-Key: drk_..."
```

Full operator list: [API reference → Filtering (OData-style)](https://websvcin.github.io/crudapi/api-reference/filtering-odata.html).

---

## 🎉 You're done

You now have:
- ✅ A REST API for any MySQL table, scoped to a tenant
- ✅ Authentication via API key
- ✅ Filtering, sorting, pagination
- ✅ Admin and tenant consoles for managing it all
- ✅ Rate limiting already guarding your endpoints — tune it from a database's own **Actions** menu
  in the tenant workspace

**Next steps:**
- Browse the [full docs site](https://websvcin.github.io/crudapi/) for every feature
- Set up production deployment with [DEPLOY.md](DEPLOY.md)
