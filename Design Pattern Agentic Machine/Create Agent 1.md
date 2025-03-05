To create an agent in Rust that writes an Actix-web server template, adds authentication, integrates a database, and implements complex business logic, we will break the task into several steps. Each step will build upon the previous one to create a comprehensive solution.

### Step 1: Set Up the Actix-Web Server Template

We'll start by creating a basic Actix-web server template. This will serve as the foundation for our application.

#### Code for Basic Actix-Web Server:
```rust
use actix_web::{web, App, HttpServer, HttpResponse, Responder};

async fn index() -> impl Responder {
    HttpResponse::Ok().body("Hello, world!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

This is a simple Actix-web server that listens on `http://127.0.0.1:8080` and responds with "Hello, world!" when you hit the root endpoint (`/`).

---

### Step 2: Add Authentication

For authentication, we can use JSON Web Tokens (JWT). We'll use the `jsonwebtoken` crate to handle JWT generation and validation.

#### Add Dependencies:
Add the following to your `Cargo.toml`:
```toml
[dependencies]
actix-web = "4"
serde = { version = "1.0", features = ["derive"] }
jsonwebtoken = "8"
chrono = "0.4"
```

#### Implement JWT Authentication:
```rust
use actix_web::{web, Error, HttpRequest, HttpResponse};
use jsonwebtoken::{decode, encode, DecodingKey, EncodingKey, Header, Validation};
use serde::{Deserialize, Serialize};
use chrono::Utc;

#[derive(Serialize, Deserialize)]
struct Claims {
    sub: String,
    exp: usize,
}

fn create_token(user_id: &str) -> Result<String, jsonwebtoken::errors::Error> {
    let expiration = Utc::now()
        .checked_add_signed(chrono::Duration::hours(1))
        .expect("valid timestamp")
        .timestamp();

    let claims = Claims {
        sub: user_id.to_owned(),
        exp: expiration as usize,
    };

    encode(
        &Header::default(),
        &claims,
        &EncodingKey::from_secret("secret".as_ref()),
    )
}

fn validate_token(token: &str) -> Result<Claims, jsonwebtoken::errors::Error> {
    decode::<Claims>(
        token,
        &DecodingKey::from_secret("secret".as_ref()),
        &Validation::default(),
    )
    .map(|data| data.claims)
}

async fn login() -> Result<HttpResponse, Error> {
    let token = create_token("user1").unwrap();
    Ok(HttpResponse::Ok().json(json!({ "token": token })))
}

async fn protected(req: HttpRequest) -> Result<HttpResponse, Error> {
    if let Some(auth_header) = req.headers().get("Authorization") {
        if let Ok(token_str) = auth_header.to_str() {
            if let Some(stripped_token) = token_str.strip_prefix("Bearer ") {
                if let Ok(claims) = validate_token(stripped_token) {
                    return Ok(HttpResponse::Ok().body(format!("Welcome {}", claims.sub)));
                }
            }
        }
    }
    Ok(HttpResponse::Unauthorized().body("Unauthorized"))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/login", web::post().to(login))
            .route("/protected", web::get().to(protected))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

Here, we have two endpoints:
- `/login`: Generates a JWT token for a user.
- `/protected`: Validates the token and allows access only if the token is valid.

---

### Step 3: Integrate Database

For database integration, we'll use the `sqlx` crate, which provides async SQL support for Rust. We'll use PostgreSQL as the database.

#### Add Dependencies:
Add the following to your `Cargo.toml`:
```toml
sqlx = { version = "0.6", features = ["runtime-tokio-native-tls", "postgres"] }
dotenv = "0.15"
```

#### Create Database Connection:
```rust
use sqlx::PgPool;
use dotenv::dotenv;
use std::env;

async fn establish_connection() -> PgPool {
    dotenv().ok();
    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    PgPool::connect(&database_url).await.expect("Failed to connect to the database")
}

#[derive(sqlx::FromRow, Debug, Serialize)]
struct User {
    id: i32,
    username: String,
    password_hash: String,
}

async fn get_user_by_id(pool: &PgPool, user_id: i32) -> Result<Option<User>, sqlx::Error> {
    sqlx::query_as::<_, User>("SELECT * FROM users WHERE id = $1")
        .bind(user_id)
        .fetch_optional(pool)
        .await
}
```

In this example, we define a `User` struct and a function to fetch a user by their ID from the database.

#### Update `main` Function:
```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let pool = establish_connection().await;

    HttpServer::new(move || {
        App::new()
            .data(pool.clone())
            .route("/login", web::post().to(login))
            .route("/protected", web::get().to(protected))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

Here, we pass the database connection pool to the Actix-web app.

---

### Step 4: Add Complex Business Logic

Let's add some business logic to our application. For example, we can implement a function that calculates the total number of posts for a user.

#### Example Business Logic:
```rust
#[derive(sqlx::FromRow, Debug, Serialize)]
struct Post {
    id: i32,
    title: String,
    content: String,
    user_id: i32,
}

async fn count_posts_for_user(pool: &PgPool, user_id: i32) -> Result<i64, sqlx::Error> {
    sqlx::query_scalar::<_, i64>("SELECT COUNT(*) FROM posts WHERE user_id = $1")
        .bind(user_id)
        .fetch_one(pool)
        .await
}

async fn user_posts_count(req: HttpRequest, pool: web::Data<PgPool>) -> Result<HttpResponse, Error> {
    if let Some(auth_header) = req.headers().get("Authorization") {
        if let Ok(token_str) = auth_header.to_str() {
            if let Some(stripped_token) = token_str.strip_prefix("Bearer ") {
                if let Ok(claims) = validate_token(stripped_token) {
                    if let Ok(Some(user)) = get_user_by_id(&pool, claims.sub.parse::<i32>().unwrap()).await {
                        let post_count = count_posts_for_user(&pool, user.id).await.unwrap();
                        return Ok(HttpResponse::Ok().json(json!({ "username": user.username, "post_count": post_count })));
                    }
                }
            }
        }
    }
    Ok(HttpResponse::Unauthorized().body("Unauthorized"))
}
```

Here, we define a new endpoint `/user/posts/count` that returns the number of posts for the authenticated user.

#### Update `main` Function:
```rust
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let pool = establish_connection().await;

    HttpServer::new(move || {
        App::new()
            .data(pool.clone())
            .route("/login", web::post().to(login))
            .route("/protected", web::get().to(protected))
            .route("/user/posts/count", web::get().to(user_posts_count))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

---

### Conclusion

You now have a Rust Actix-web server with:
1. A basic server setup.
2. JWT-based authentication.
3. Database integration using `sqlx`.
4. Complex business logic for counting user posts.

This structure can be expanded further to include more endpoints, additional business logic, and more sophisticated error handling.
