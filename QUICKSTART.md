# 🚀 5-Minute Quickstart

Get from zero to a live API in under 5 minutes.

---

## Step 1 — Install (1 min)

```bash
docker run -d -p 8080:80 \
  -v $(pwd)/data:/app/Data \
  --name crudapi \
  ghcr.io/web-svc/crudapi:latest

Visit http://localhost:8080 — landing page should load.

Step 2 — Login to Admin (30 sec)
Click "Admin Console" on the landing page.

Username: admin
Password: admin@123

Immediately change the password at /admin/adminusers.

Step 3 — Register a MySQL database (1 min)

Click Databases in left nav
Click + Register Database
Fill in:

DB Key: master (short alias used in API URLs)
Name: My First DB
Pick MySQL as the engine
Host, Port, DB Name, User, Password


Click 🔌 Test Connection — green means ready
Click Register


Step 4 — Create an API key (30 sec)

Click API Clients in left nav
Click + Create API Client
Pick the database you just registered
Pick a role: Editor (full CRUD) for now
Click Create
Copy the API key — you only see it once. Looks like drk_abc123...


Step 5 — Call your data (1 min)
Pick any table in your database. Let's say you have a customers table.
Shellcurl http://localhost:8080/api/v1/master/crud/customers \  -H "X-API-Key: drk_yourkey..."Show more lines
You get back JSON with all rows. ✅

Step 6 — Try filters
Shell# Filtercurl "http://localhost:8080/api/v1/master/crud/customers?filter=country%20eq%20'India'" \  -H "X-API-Key: drk_..."# Sortcurl "http://localhost:8080/api/v1/master/crud/customers?orderby=created_at%20desc" \  -H "X-API-Key: drk_..."# Paginatecurl "http://localhost:8080/api/v1/master/crud/customers?limit=10&offset=0" \  -H "X-API-Key: drk_..."Show more lines
Full operator list at http://localhost:8080/help#filters.

🎉 You're done
You now have:

✅ A REST API for any MySQL table
✅ Authentication via API key
✅ Filtering, sorting, pagination
✅ Admin UI for managing it all

Next steps:

Browse the in-app help for all features
Set up DEPLOY.md
Configure http://localhost:8080/help#cors if you're calling from a browser
