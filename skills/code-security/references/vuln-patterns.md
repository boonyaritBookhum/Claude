# Vulnerability Patterns Reference

This file is pre-loaded into shared context by the orchestrator before agents start. Agents must NOT re-read this file — use the sections already in context.
Every finding MUST include a CWE ID from the mappings below each section.

---

## Secrets & Credentials — Patterns to Flag
**CWE:** CWE-798 (Hardcoded Credentials), CWE-321 (Hardcoded Crypto Key), CWE-312 (Cleartext Storage of Sensitive Info)

**Known key formats (any language):**
- AWS: `AKIA[A-Z0-9]{16}` (Access Key ID), `[A-Za-z0-9/+=]{40}` near "aws_secret"
- GCP: `AIza[0-9A-Za-z\-_]{35}`
- GitHub: `ghp_[a-zA-Z0-9]{36}`, `ghs_[a-zA-Z0-9]{36}`
- Slack: `xox[baprs]-[0-9A-Za-z\-]+`
- Stripe: `sk_live_[0-9a-zA-Z]{24}`, `pk_live_[0-9a-zA-Z]{24}`
- OpenAI: `sk-[a-zA-Z0-9]{48}`, `sk-proj-[a-zA-Z0-9_\-]{100,}`, `sk-svcacct-[a-zA-Z0-9_\-]{100,}`
- PEM: `-----BEGIN (RSA |EC |)PRIVATE KEY-----`

**Generic credential variable names assigned to a literal string** (flag if value is not a placeholder like `your_key_here`, `<SECRET>`, `$ENV_VAR`, `os.getenv`):
- Variable/key names: `password`, `passwd`, `pwd`, `secret`, `api_key`, `apikey`, `token`, `auth_token`, `access_token`, `private_key`, `client_secret`, `database_url`, `db_password`

**Connection strings with embedded credentials:**
- `mongodb://user:pass@`, `postgresql://user:pass@`, `mysql://user:pass@`, `redis://:pass@`
- `Server=...;Password=`, `Data Source=...;Password=`

**Config files to check:** `.env.example`, `config.yaml`, `appsettings.json`, `application.properties`, `settings.py` — flag actual values, not env-var references.

---

## Injection — Language-Specific Patterns
**CWE:** CWE-89 (SQL Injection), CWE-78 (OS Command Injection), CWE-79 (XSS), CWE-22 (Path Traversal), CWE-918 (SSRF), CWE-94 (Code Injection), CWE-1321 (Prototype Pollution), CWE-502 (Unsafe Deserialization), CWE-1336 (SSTI)

**Go:**
- SQL: `db.Query(fmt.Sprintf(`, `db.Exec(fmt.Sprintf(`, string concat with `+` in query string
- OS: `exec.Command(` with variable first arg or args from user input
- Path traversal: `os.Open(`, `os.ReadFile(`, `ioutil.ReadFile(` with unsanitized user path variable
- SSRF: `http.Get(userURL)`, `http.NewRequest(method, userURL` without URL allowlist

**TypeScript / JavaScript:**
- SQL: template literals in `.query(`, `.execute(`, `.raw(`, `.where(` — e.g. `` `SELECT * FROM ${table}` ``
- OS: `child_process.exec(`, `execSync(`, `spawnSync(` with user-controlled string
- XSS: `.innerHTML =`, `.outerHTML =`, `document.write(`, `dangerouslySetInnerHTML={{__html:`, `eval(`, `new Function(`
- Path traversal: `fs.readFile(`, `fs.readFileSync(`, `path.join(__dirname, req.params` or `req.query`
- Prototype pollution: `Object.assign({}, req.body)` into a prototype, `_.merge({}, req.body)`

**Python:**
- SQL: `cursor.execute(f"`, `cursor.execute("..." % `, `.execute("...".format(`, `.execute("..." + `
- OS: `os.system(`, `subprocess.run(..., shell=True`, `subprocess.Popen(..., shell=True`, `os.popen(`
- Unsafe deserialize: `pickle.loads(`, `yaml.load(data)` without `Loader=yaml.SafeLoader`, `marshal.loads(`
- SSTI: `render_template_string(user_input)`, `jinja2.Template(user_input).render(`
- Path traversal: `open(user_path)` without `os.path.basename` or directory restriction

**Java:**
- SQL: `Statement.executeQuery("..."+userInput)`, `createNativeQuery("..."+userInput)`, `createQuery("..."+`
- OS: `Runtime.getRuntime().exec(userInput)`, `new ProcessBuilder(userInput)`
- Deserialize: `ObjectInputStream.readObject(` without whitelist (class filtering)
- XSS: `response.getWriter().print(request.getParameter(` unescaped, outputs without `escapeHtml`
- Path traversal: `new File(baseDir + userInput)` without `.getCanonicalPath()` check

**.NET / C#:**
- SQL: `new SqlCommand("..." + userInput`, `string.Format("SELECT...{0}", userInput`  
- OS: `Process.Start(userInput)`, `new ProcessStartInfo(userInput)`
- Path traversal: `File.ReadAllText(Path.Combine(root, userInput))` without canonicalization
- Deserialize: `BinaryFormatter.Deserialize(`, `JavaScriptSerializer.Deserialize(`, `XmlSerializer` with external DTD
- XSS: `Response.Write(Request.QueryString[` unescaped

**Ruby:**
- SQL: string interpolation in `.where("...#{userInput}")`, `.find_by_sql("..." + userInput)`
- OS: `` `#{userInput}` ``, `system(userInput)`, `exec(userInput)`, `IO.popen(userInput)`, `eval(userInput)`
- Deserialize: `Marshal.load(untrusted_data)`
- Path traversal: `File.read(params[:file])`, `send_file(params[:path])` without path restriction

**Rust:**
- Unsafe blocks: `unsafe {` — note every occurrence, assess if necessary
- OS: `Command::new(user_input)`, `Command::new("sh").arg("-c").arg(user_input)`
- Path traversal: `fs::read_to_string(user_path)` without `canonicalize()` + prefix check

---

## Auth & Session — Patterns to Flag
**CWE:** CWE-287 (Improper Authentication), CWE-639 (IDOR), CWE-862 (Missing Authorization), CWE-614 (Insecure Cookie), CWE-384 (Session Fixation), CWE-916 (Weak Password Hash), CWE-347 (JWT Signature Not Verified), CWE-208 (Timing Attack)

- JWT `alg: none` / `algorithm: "none"` in decode options
- JWT decode without signature verification: `.decode(token)` without `verify=True`, `options={"verify_signature": True}`
- Token comparison with `==` instead of `hmac.compare_digest` / `crypto.timingSafeEqual` / `SecureEquals` (timing attack)
- `Set-Cookie` without `HttpOnly`, `Secure`, `SameSite` attributes
- Session with no `maxAge` / `expires` — indefinite sessions
- Route handler with no auth middleware and no inline permission check before accessing user data
- `md5(password)`, `sha1(password)`, `sha256(password)` without salt for password storage

---

## CSRF — Patterns to Flag
**CWE:** CWE-352 (Cross-Site Request Forgery)

- State-changing route handler (POST/PUT/PATCH/DELETE) with no CSRF token check, no `csurf` middleware, no `@CsrfProtect`, no Django `{% csrf_token %}`, no Rails `protect_from_forgery`
- Cookie set without `SameSite=Strict` or `SameSite=Lax` (combined with missing CSRF token significantly raises risk)
- Fetch/XHR that sends cookies but omits `X-Requested-With` or CSRF header for state-changing calls
- Note: if the API is stateless (JWT in Authorization header, no cookies) and `SameSite` is set, CSRF risk is low — downgrade to INFO

---

## Crypto — Patterns to Flag
**CWE:** CWE-327 (Broken Crypto Algorithm), CWE-328 (Weak Hash), CWE-329 (Hardcoded IV), CWE-330 (Insufficient Randomness), CWE-295 (Improper Certificate Validation)

- `MD5`, `md5`, `SHA1`, `sha1` used for password hashing, HMAC for auth tokens, or digital signatures
- `DES`, `3DES`, `TripleDES`, `RC4`, `AES/ECB` in encryption
- Hardcoded IV or nonce: `iv = b"\x00" * 16`, `iv = "0000000000000000"`, `nonce = 0`
- `Math.random()` / `rand.Intn()` / `random.random()` for security tokens, session IDs, or CSRF tokens (use `crypto.randomBytes` / `rand.Read` / `secrets.token_bytes` instead)
- `InsecureSkipVerify: true` (Go TLS config), `verify=False` (Python requests), `NODE_TLS_REJECT_UNAUTHORIZED=0`

---

## Docker & Container Security — Patterns to Flag
**CWE:** CWE-250 (Excessive Privileges), CWE-269 (Improper Privilege Management), CWE-1395 (Dependency on Vulnerable Component)

**Dockerfile:**
- Running as root: no `USER` instruction, or `USER root` without switching back
- Using `:latest` or no tag: `FROM node:latest`, `FROM python` (no pinned version)
- Copying everything: `COPY . .` or `ADD . .` without `.dockerignore` — may leak `.env`, `.git`, secrets
- Secrets in build: `ARG PASSWORD=`, `ENV API_KEY=`, `ENV DB_PASSWORD=` with literal values
- Unnecessary tools: installing `curl`, `wget`, `netcat`, `ssh` in production image
- No multi-stage build: final image contains build tools, source code, dev dependencies
- No `HEALTHCHECK` instruction
- Exposed sensitive ports: `EXPOSE 22` (SSH), `EXPOSE 3306` (MySQL), `EXPOSE 5432` (Postgres) directly
- Using `ADD` for remote URLs (prefer `COPY` + explicit download with checksum)
- `apt-get install` without `--no-install-recommends` (bloated image)

**docker-compose.yml:**
- `privileged: true` — full host access
- `network_mode: host` — no network isolation
- Secrets in `environment:` as plaintext (use `secrets:` or `.env` reference)
- Volumes mounting sensitive host paths: `/`, `/etc`, `/var/run/docker.sock`
- No resource limits (`mem_limit`, `cpus`) — resource exhaustion risk
- `restart: always` without health checks — crash loop risk

---

## API Security — Patterns to Flag
**CWE:** CWE-770 (No Rate Limit), CWE-200 (Information Exposure), CWE-284 (Improper Access Control), CWE-400 (Resource Exhaustion), CWE-942 (Overly Permissive CORS)

**Missing rate limiting:**
- Express/Koa/Fastify: no `express-rate-limit`, `@nestjs/throttler`, or equivalent middleware on auth/public endpoints
- Go: no rate limiter middleware (e.g. `golang.org/x/time/rate`, `tollbooth`)
- Django/Flask: no `django-ratelimit`, `flask-limiter`
- Spring: no `bucket4j`, `resilience4j` rate limiting
- Ruby/Rails: no `rack-attack`

**Data over-fetching:**
- Returning entire DB objects to client: `res.json(user)` without field selection (exposes password hash, internal IDs, PII)
- No DTO/serializer pattern — direct model-to-response
- GraphQL: introspection enabled in production (`introspection: true` or not explicitly disabled)
- GraphQL: no query depth/complexity limiting — allows deeply nested queries (DoS)

**Missing pagination:**
- `SELECT * FROM` or `.find({})` without `LIMIT`/`.limit()` — returns unbounded results
- API endpoint returning arrays without `page`/`limit`/`offset` parameters
- No max page size enforcement (client can request `?limit=999999`)

**Missing input validation on API:**
- No request body schema validation (no `joi`, `zod`, `class-validator`, `pydantic`, `marshmallow`)
- Accepting arbitrary JSON keys — mass assignment risk
- No `Content-Type` validation on file/multipart endpoints

**Insecure API patterns:**
- API versioning absent — breaking changes affect all clients
- `DELETE` endpoints without soft-delete or confirmation
- Bulk operations without size limits (`POST /api/bulk` accepting unbounded arrays)
- Sensitive data in URL query params (tokens, passwords in GET requests) — logged by servers/proxies

---

## File Upload Security — Patterns to Flag
**CWE:** CWE-434 (Unrestricted File Upload), CWE-400 (Resource Exhaustion), CWE-22 (Path Traversal via filename)

**No file type validation:**
- Accepting uploads without checking MIME type or extension
- Validating only by extension (not magic bytes) — easily spoofed
- Missing allowlist of accepted types (using blocklist instead, or no list at all)
- `multer()` / `express-fileupload` / `Busboy` without `fileFilter` or `limits`

**No file size limit:**
- Upload handler without `maxFileSize`, `limits.fileSize`, `MAX_CONTENT_LENGTH`, `upload_max_filesize`
- Streaming uploads without size cap — memory/disk exhaustion
- Multipart with no part count limit — zip bomb / billion laughs attack vector

**Dangerous storage:**
- Saving to web-accessible directory (`public/`, `static/`, `wwwroot/`) — direct execution risk
- Using original filename from client: `req.file.originalname` / `file.filename` without sanitization
- Path traversal via filename: `../../etc/cron.d/malicious` — must strip path separators
- No randomization of stored filename (collision + enumeration risk)

**Missing content validation:**
- Image uploads not validated (could be HTML/SVG with embedded JS → stored XSS)
- SVG uploads without sanitization (`<script>`, `onload=`, `xlink:href=` in SVG)
- No antivirus/malware scan on uploaded files
- Zip/archive uploads without bomb detection (nested zip, huge expansion ratio)

**Language-specific patterns:**
- **Node.js:** `multer({ dest: 'public/uploads' })` without fileFilter, `express-fileupload` with `useTempFiles: false` (memory), `fs.writeFile(req.body.filename, data)`
- **Python:** `request.files['file'].save(os.path.join('uploads', filename))` with unsanitized filename, Django `FileField` without `validators`, Flask without `secure_filename()`
- **Go:** `r.FormFile("upload")` without size limit (`r.Body = http.MaxBytesReader()`), saving to `./static/` directly
- **Java:** `MultipartFile.transferTo(new File(uploadDir + originalFilename))`, no `@Size` annotation, Spring without `max-file-size` config
- **C#:** `IFormFile` saved with `file.FileName` directly, no `[RequestSizeLimit]`, `Path.Combine(webRoot, fileName)` without sanitization
- **Ruby:** `params[:file].read` without size check, CarrierWave/ActiveStorage without content-type validation
