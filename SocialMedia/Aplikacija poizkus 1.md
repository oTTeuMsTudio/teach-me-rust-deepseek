# Socia media application
Creating a social media application in pure Rust is an ambitious and exciting project. Rust, known for its safety, performance, and concurrency features, is an excellent choice for building robust applications. Below, I will guide you through the process of designing and implementing a simple social media application in Rust. This will include backend development, database integration, and API design.

---

### **1. Understanding the Requirements**
Before diving into the code, let's outline the basic features of the social media application:
- **User Management**: Users can register, log in, and manage their profiles.
- **Posts**: Users can create, read, update, and delete posts.
- **Follow System**: Users can follow/unfollow other users.
- **Feed**: Users can see posts from the people they follow.
- **API**: A RESTful or GraphQL API to interact with the application.

For simplicity, we'll focus on building a backend server with RESTful APIs using Rust. Frontend development (e.g., React, Vue.js) is outside the scope of this guide.

---

### **2. Setting Up the Project**
1. **Install Rust**: If you haven't already, install Rust by following the instructions at [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install).
2. **Create a New Project**:
   ```bash
   cargo new social_media_app --bin
   cd social_media_app
   ```
3. **Add Dependencies**: Open `Cargo.toml` and add the necessary crates:
   ```toml
   [dependencies]
   actix-web = "4.0" # Web framework
   serde = { version = "1.0", features = ["derive"] } # Serialization/Deserialization
   serde_json = "1.0" # JSON handling
   sqlx = { version = "0.6", features = ["runtime-tokio-native-tls", "postgres"] } # Database ORM
   dotenv = "0.15" # Environment variable management
   bcrypt = "0.13" # Password hashing
   uuid = { version = "1.3", features = ["v4"] } # UUID generation
   chrono = "0.4" # Date and time handling
   ```

Run `cargo build` to download and compile the dependencies.

---

### **3. Designing the Database**
We'll use PostgreSQL as our database. Define the schema for the application:

#### Tables:
1. **Users**:
   - `id`: UUID (Primary Key)
   - `username`: String
   - `email`: String
   - `password_hash`: String
   - `created_at`: Timestamp

2. **Posts**:
   - `id`: UUID (Primary Key)
   - `user_id`: UUID (Foreign Key referencing Users.id)
   - `content`: Text
   - `created_at`: Timestamp

3. **Follows**:
   - `follower_id`: UUID (Foreign Key referencing Users.id)
   - `followed_id`: UUID (Foreign Key referencing Users.id)

You can create these tables using SQL scripts or migrations with tools like `sqlx-cli`.

---

### **4. Backend Implementation**
We'll use `actix-web` for the web framework and `sqlx` for database interactions.

#### **a. Database Connection**
Set up a connection pool for PostgreSQL in `src/main.rs`:
```rust
use sqlx::PgPool;
use dotenv::dotenv;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    dotenv().ok(); // Load environment variables
    let database_url = std::env::var("DATABASE_URL").expect("DATABASE_URL must be set");
    let pool = PgPool::connect(&database_url).await.expect("Failed to connect to DB");

    // Start the server
    HttpServer::new(move || {
        App::new()
            .data(pool.clone()) // Pass the database pool to handlers
            .service(register_user)
            .service(login_user)
            .service(create_post)
            .service(get_feed)
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

#### **b. User Registration**
Create a handler for user registration:
```rust
use actix_web::{web, HttpResponse};
use serde::Deserialize;
use bcrypt::{hash, DEFAULT_COST};

#[derive(Deserialize)]
struct RegisterUser {
    username: String,
    email: String,
    password: String,
}

async fn register_user(
    pool: web::Data<PgPool>,
    form: web::Json<RegisterUser>,
) -> HttpResponse {
    let hashed_password = hash(&form.password, DEFAULT_COST).unwrap();
    let user_id = uuid::Uuid::new_v4();

    sqlx::query!(
        r#"
        INSERT INTO users (id, username, email, password_hash, created_at)
        VALUES ($1, $2, $3, $4, NOW())
        "#,
        user_id,
        form.username,
        form.email,
        hashed_password
    )
    .execute(pool.get_ref())
    .await
    .map_err(|_| HttpResponse::InternalServerError().finish())?;

    HttpResponse::Ok().body("User registered successfully")
}
```

#### **c. User Login**
Implement a login endpoint with password verification:
```rust
use bcrypt::verify;

#[derive(Deserialize)]
struct LoginUser {
    email: String,
    password: String,
}

async fn login_user(
    pool: web::Data<PgPool>,
    form: web::Json<LoginUser>,
) -> HttpResponse {
    let user = sqlx::query!(
        r#"
        SELECT id, password_hash FROM users WHERE email = $1
        "#,
        form.email
    )
    .fetch_optional(pool.get_ref())
    .await
    .map_err(|_| HttpResponse::InternalServerError().finish())?;

    if let Some(user) = user {
        if verify(&form.password, &user.password_hash).unwrap() {
            return HttpResponse::Ok().body("Login successful");
        }
    }

    HttpResponse::Unauthorized().body("Invalid credentials")
}
```

#### **d. Creating Posts**
Allow users to create posts:
```rust
#[derive(Deserialize)]
struct CreatePost {
    user_id: uuid::Uuid,
    content: String,
}

async fn create_post(
    pool: web::Data<PgPool>,
    form: web::Json<CreatePost>,
) -> HttpResponse {
    let post_id = uuid::Uuid::new_v4();

    sqlx::query!(
        r#"
        INSERT INTO posts (id, user_id, content, created_at)
        VALUES ($1, $2, $3, NOW())
        "#,
        post_id,
        form.user_id,
        form.content
    )
    .execute(pool.get_ref())
    .await
    .map_err(|_| HttpResponse::InternalServerError().finish())?;

    HttpResponse::Ok().body("Post created successfully")
}
```

#### **e. Fetching Feed**
Fetch posts from users that the current user follows:
```rust
async fn get_feed(
    pool: web::Data<PgPool>,
    path: web::Path<uuid::Uuid>,
) -> HttpResponse {
    let user_id = path.into_inner();

    let posts = sqlx::query!(
        r#"
        SELECT p.id, p.content, p.created_at, u.username
        FROM posts p
        JOIN follows f ON f.followed_id = p.user_id
        JOIN users u ON u.id = p.user_id
        WHERE f.follower_id = $1
        ORDER BY p.created_at DESC
        "#,
        user_id
    )
    .fetch_all(pool.get_ref())
    .await
    .map_err(|_| HttpResponse::InternalServerError().finish())?;

    HttpResponse::Ok().json(posts)
}
```

---

### **5. Running the Application**
1. Set up your `.env` file:
   ```env
   DATABASE_URL=postgres://username:password@localhost/social_media_db
   ```
2. Run the application:
   ```bash
   cargo run
   ```

---

### **6. Testing the API**
Use tools like Postman or `curl` to test the endpoints:
- **Register User**: `POST /register`
- **Login User**: `POST /login`
- **Create Post**: `POST /posts`
- **Get Feed**: `GET /feed/{user_id}`

---

### **7. Next Steps**
- Add authentication (e.g., JWT tokens).
- Implement pagination for the feed.
- Optimize database queries.
- Add unit tests and integration tests.
- Deploy the application using Docker or cloud platforms like AWS or Heroku.

---

This guide provides a foundational structure for building a social media application in Rust. You can expand it further by adding more features, improving error handling, and enhancing performance. Let me know if you need help with any specific part!
