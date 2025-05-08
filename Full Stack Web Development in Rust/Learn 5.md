# Full-stack, AI-powered, desktop + web app


# 🛡️ 1. Add HTTPS, CORS, and Secure Headers

To deploy your Leptos + Axum app securely to production (e.g., Vercel, Cloudflare, or self-hosted), you’ll want:

- ✅ HTTPS
- ✅ Proper CORS setup
- ✅ Security headers like HSTS, CSP, etc.

## 🔒 Step 1: Enable HTTPS

For local dev, use `rustls` for TLS support.

Update `Cargo.toml`:

```toml
[dependencies]
axum = { version = "0.6", features = ["tls"] }
rustls = "0.20"
```

Generate a self-signed cert:

```bash
openssl req -x509 -newkey rsa:4096 -nodes -keyout key.pem -out cert.pem -days 365
```

Update `main.rs` to serve over HTTPS:

```rust
use axum::Router;
use std::net::SocketAddr;
use tokio::net::TcpListener;
use rustls::{ServerConfig, Certificate, PrivateKey};
use std::fs::File;
use std::io::BufReader;
use std::sync::Arc;

async fn start_https_server(app: Router) {
    let cert_file = &mut BufReader::new(File::open("cert.pem").unwrap());
    let key_file = &mut BufReader::new(File::open("key.pem").unwrap());

    let cert = rustls_pemfile::certs(cert_file)
        .collect::<Result<Vec<_>, _>>()
        .unwrap();

    let mut keys = rustls_pemfile::pkcs8_private_keys(key_file)
        .collect::<Result<Vec<_>, _>>()
        .unwrap();

    let config = ServerConfig::builder()
        .with_safe_defaults()
        .with_no_client_auth()
        .with_single_cert(cert, keys.remove(0))
        .unwrap();

    let tls = axum_server::tls::RustlsConfig::from_config(Arc::new(config));

    let addr = SocketAddr::from(([0, 0, 0, 0], 3000));
    println!("🚀 Listening on https://{}", addr);

    axum_server::bind_rustls(addr, tls)
        .serve(app.into_make_service())
        .await
        .unwrap();
}
```

> Note: For real deployment, use Let's Encrypt with something like [acme](https://github.com/ramosbugs/acme).

---

## 🔗 Step 2: Add CORS Middleware

Add CORS middleware in `main.rs`:

```rust
use axum::{
    http::HeaderValue,
    routing::get,
    Router,
};
use tower_http::cors::{CorsLayer, Any};

let cors = CorsLayer::new()
    .allow_origin(Any)
    .allow_methods([http::Method::GET, http::Method::POST])
    .allow_headers([http::header::CONTENT_TYPE]);

let app = Router::new().fallback(handler).layer(cors);
```

---

## 🧱 Step 3: Add Security Headers

Use `tower-http` to add security headers:

```bash
cargo add tower-http
```

Then in `main.rs`:

```rust
use tower_http::set_header::SetResponseHeaderLayer;
use http::header::{HeaderName, HeaderValue};

let hsts = SetResponseHeaderLayer::overriding(
    HeaderName::from_static("strict-transport-security"),
    HeaderValue::from_static("max-age=31536000; includeSubDomains"),
);

let csp = SetResponseHeaderLayer::overriding(
    HeaderName::from_static("content-security-policy"),
    HeaderValue::from_static("default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';"),
);

let x_content_type_options = SetResponseHeaderLayer::overriding(
    HeaderName::from_static("x-content-type-options"),
    HeaderValue::from_static("nosniff"),
);

let app = Router::new()
    .fallback(handler)
    .layer(cors)
    .layer(hsts)
    .layer(csp)
    .layer(x_content_type_options);
```

Now your app is secure for public deployment 🛡️

---

# 📈 2. Add Dashboard View with Charts

We'll build a dashboard that shows task completion stats using a chart library like Chart.js or ECharts via WebAssembly.

## 📦 Step 1: Add WASM-Compatible Chart Library

Use `leptos-use` and `web-sys`.

Install `wasm-bindgen`:

```bash
cargo add wasm-bindgen
```

Add JS charting library (Chart.js) in `index.html`:

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
```

## 📊 Step 2: Create Dashboard Component

In `lib.rs`, create a new component:

```rust
#[component]
fn Dashboard() -> impl IntoView {
    let db = use_context::<Db>().expect("to have found database");

    let stats = create_resource(|| (), move |_| {
        let db = db.clone();
        async move {
            let tasks = db.get_tasks().await.unwrap_or_default();
            let total = tasks.len();
            let completed = tasks.iter().filter(|t| t.completed).count();
            (total, completed)
        }
    });

    view! {
        <h1>"Dashboard"</h1>
        <Suspense fallback=move || view! { <p>"Loading stats..."</p> }>
            {move || {
                stats.with(|(total, completed)| {
                    let percent = if *total > 0 {
                        (*completed as f64 / *total as f64) * 100.0
                    } else {
                        0.0
                    };
                    view! {
                        <div class="stats shadow">
                            <div class="stat">
                                <div class="stat-title">"Total Tasks"</div>
                                <div class="stat-value">{*total}</div>
                            </div>
                            <div class="stat">
                                <div class="stat-title">"Completed"</div>
                                <div class="stat-value">{*completed}</div>
                                <div class="stat-desc">{format!("{:.1}%", percent)}</div>
                            </div>
                        </div>
                        <canvas id="chart" width="400" height="200"></canvas>
                    }
                })
            }}
        </Suspense>
    }
}
```

You can extend this with actual charts using `web-sys` bindings or JS interop.

---

# 💾 3. Add CSV Export / Import

Let users export their tasks as CSV.

## 📁 Step 1: Add CSV Crate

```bash
cargo add csv serde
```

## 📥 Step 2: Implement Export Function

In `lib.rs`:

```rust
use wasm_bindgen::prelude::*;
use web_sys::HtmlElement;

#[wasm_bindgen]
extern "C" {
    #[wasm_bindgen(js_namespace = ["window"], js_name = "URL")]
    fn url_create_object_url(blob: &web_sys::Blob) -> String;

    #[wasm_bindgen(js_namespace = ["window"], js_name = "document")]
    var document: HtmlElement;

    #[wasm_bindgen(js_namespace = ["window"], js_name = "location")]
    fn download(url: &str);
}

fn export_tasks(tasks: Vec<Task>) {
    let mut wtr = csv::Writer::from_writer(vec![]);
    for task in tasks {
        wtr.serialize(task).unwrap();
    }
    let data = wtr.into_inner().unwrap();
    let blob = web_sys::Blob::new_with_u8_array_sequence(&js_sys::Array::of1(&data.into_boxed_slice().into())).unwrap();
    let url = url_create_object_url(&blob);
    download(&url);
}
```

Call it from a button:

```rust
<button on:click=move |_| {
    let tasks = tasks.get_untracked();
    export_tasks(tasks.iter().cloned().collect())
}>Export CSV</button>
```

Add import functionality similarly by reading file input with `web-sys`.

---

# 🌐 4. Multi-User Support with PostgreSQL

Right now, all users share one global DB. Let’s make it per-user with PostgreSQL and JWT auth.

## 📦 Step 1: Update Dependencies

```toml
sqlx = { version = "0.6", features = ["postgres", "runtime-tokio-native-tls"] }
jsonwebtoken = "0.2"
serde = { version = "1.0", features = ["derive"] }
```

## 🧠 Step 2: Add User Model

```rust
#[derive(Serialize, Deserialize)]
pub struct User {
    pub id: i32,
    pub email: String,
}
```

Create login endpoint that returns JWT token:

```rust
#[post("/login")]
async fn login_user(db: Pool<Postgres>, body: LoginBody) -> Result<Json<String>, StatusCode> {
    // check user exists...
    let claims = Claims {
        sub: user.id.to_string(),
        exp: (Utc::now() + Duration::hours(1)).timestamp() as usize,
    };
    let token = encode(&Header::default(), &claims, &encoding_key)?;
    Ok(Json(token))
}
```

Now each request includes user ID, so you can scope tasks to them.

---

# 🎯 Final Thoughts

You’ve now built an **AI-enhanced, secure, multi-platform, full-stack Rust app** that:

✅ Is deployable anywhere  
✅ Works offline and on desktop  
✅ Uses modern UI and charts  
✅ Can scale to enterprise-grade apps  
