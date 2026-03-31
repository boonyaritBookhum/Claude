# Vulnerability Patterns Reference

Read this file before starting Phase 1 scan agents.

---

## Secrets & Credentials — Patterns to Flag

**Known key formats (any language):**
- AWS: `AKIA[A-Z0-9]{16}` (Access Key ID), `[A-Za-z0-9/+=]{40}` near "aws_secret"
- GCP: `AIza[0-9A-Za-z\-_]{35}`
- GitHub: `ghp_[a-zA-Z0-9]{36}`, `ghs_[a-zA-Z0-9]{36}`
- Slack: `xox[baprs]-[0-9A-Za-z\-]+`
- Stripe: `sk_live_[0-9a-zA-Z]{24}`, `pk_live_[0-9a-zA-Z]{24}`
- OpenAI: `sk-[a-zA-Z0-9]{48}`
- PEM: `-----BEGIN (RSA |EC |)PRIVATE KEY-----`

**Generic credential variable names assigned to a literal string** (flag if value is not a placeholder like `your_key_here`, `<SECRET>`, `$ENV_VAR`, `os.getenv`):
- Variable/key names: `password`, `passwd`, `pwd`, `secret`, `api_key`, `apikey`, `token`, `auth_token`, `access_token`, `private_key`, `client_secret`, `database_url`, `db_password`

**Connection strings with embedded credentials:**
- `mongodb://user:pass@`, `postgresql://user:pass@`, `mysql://user:pass@`, `redis://:pass@`
- `Server=...;Password=`, `Data Source=...;Password=`

**Config files to check:** `.env.example`, `config.yaml`, `appsettings.json`, `application.properties`, `settings.py` — flag actual values, not env-var references.

---

## Injection — Language-Specific Patterns

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

- JWT `alg: none` / `algorithm: "none"` in decode options
- JWT decode without signature verification: `.decode(token)` without `verify=True`, `options={"verify_signature": True}`
- Token comparison with `==` instead of `hmac.compare_digest` / `crypto.timingSafeEqual` / `SecureEquals` (timing attack)
- `Set-Cookie` without `HttpOnly`, `Secure`, `SameSite` attributes
- Session with no `maxAge` / `expires` — indefinite sessions
- Route handler with no auth middleware and no inline permission check before accessing user data
- `md5(password)`, `sha1(password)`, `sha256(password)` without salt for password storage

---

## CSRF — Patterns to Flag

- State-changing route handler (POST/PUT/PATCH/DELETE) with no CSRF token check, no `csurf` middleware, no `@CsrfProtect`, no Django `{% csrf_token %}`, no Rails `protect_from_forgery`
- Cookie set without `SameSite=Strict` or `SameSite=Lax` (combined with missing CSRF token significantly raises risk)
- Fetch/XHR that sends cookies but omits `X-Requested-With` or CSRF header for state-changing calls
- Note: if the API is stateless (JWT in Authorization header, no cookies) and `SameSite` is set, CSRF risk is low — downgrade to INFO

## Crypto — Patterns to Flag

- `MD5`, `md5`, `SHA1`, `sha1` used for password hashing, HMAC for auth tokens, or digital signatures
- `DES`, `3DES`, `TripleDES`, `RC4`, `AES/ECB` in encryption
- Hardcoded IV or nonce: `iv = b"\x00" * 16`, `iv = "0000000000000000"`, `nonce = 0`
- `Math.random()` / `rand.Intn()` / `random.random()` for security tokens, session IDs, or CSRF tokens (use `crypto.randomBytes` / `rand.Read` / `secrets.token_bytes` instead)
- `InsecureSkipVerify: true` (Go TLS config), `verify=False` (Python requests), `NODE_TLS_REJECT_UNAUTHORIZED=0`
