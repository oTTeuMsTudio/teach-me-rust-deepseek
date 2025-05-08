# An ecosystem in Rust


# 🧰 Build a CLI Tool to Manage Your SaaS Platform

We’ll create a **Rust CLI tool** that lets you:

✅ Create users  
✅ View tasks  
✅ Manage subscriptions  
✅ Seed data  
✅ Run migrations  
✅ Deploy the app  

This will make your SaaS platform **fully operable from the terminal**, perfect for DevOps, admin tools, or scripting.

---

## 📦 Step 1: Add Dependencies

In `Cargo.toml` (root project):

```toml
[dependencies]
structopt = "0.3"
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.6", features = ["postgres", "runtime-tokio-native-tls"] }
bcrypt = "0.11"
serde = { version = "1.0", features = ["derive"] }
dotenvy = "0.15"
clap = { version = "4.0", features = ["derive"] }
anyhow = "1.0"
```

---

## 🗂️ Step 2: Create CLI Structure

Create a new folder: `cli/`

Inside `cli/main.rs`:

```rust
use clap::Parser;
use std::env;
use anyhow::Result;
use sqlx::PgPool;
use model::Db;

mod model;

#[derive(Parser)]
#[command(name = "saasctl")]
#[command(about = "Admin CLI for managing the SaaS platform", long_about = None)]
enum Cli {
    /// User management commands
    User(UserCommand),

    /// Task management commands
    Task(TaskCommand),

    /// Subscription management commands
    Sub(SubscriptionCommand),

    /// Database operations
    Db(DbCommand),
}

#[derive(clap::Args)]
struct UserCommand {
    #[command(subcommand)]
    cmd: UserSubCommand,
}

#[derive(clap::Subcommand)]
enum UserSubCommand {
    /// Create a new user
    Create { email: String, password: String },

    /// List all users
    List,
}

#[derive(clap::Args)]
struct TaskCommand {
    #[command(subcommand)]
    cmd: TaskSubCommand,
}

#[derive(clap::Subcommand)]
enum TaskSubCommand {
    /// List all tasks for a user
    List { user_id: i32 },
}

#[derive(clap::Args)]
struct SubscriptionCommand {
    #[command(subcommand)]
    cmd: SubSubCommand,
}

#[derive(clap::Subcommand)]
enum SubSubCommand {
    /// Show subscription info for a user
    Info { user_id: i32 },
}

#[derive(clap::Args)]
struct DbCommand {
    #[command(subcommand)]
    cmd: DbSubCommand,
}

#[derive(clap::Subcommand)]
enum DbSubCommand {
    /// Run database migrations
    Migrate,

    /// Seed initial data
    Seed,
}

#[tokio::main]
async fn main() -> Result<()> {
    dotenv::dotenv().ok();

    let cli = Cli::parse();
    let db_url = env::var("DATABASE_URL")?;
    let pool = PgPool::connect(&db_url).await?;
    let db = Db::new(pool);

    match cli {
        Cli::User(cmd) => match cmd.cmd {
            UserSubCommand::Create { email, password } => {
                let hash = bcrypt::hash(password, bcrypt::DEFAULT_COST)?;
                let user = db.create_user(&email, &hash).await?;
                println!("Created user: {:?}", user);
            }
            UserSubCommand::List => {
                let users = db.get_users().await?;
                for user in users {
                    println!("{:?}", user);
                }
            }
        },
        Cli::Task(cmd) => match cmd.cmd {
            TaskSubCommand::List { user_id } => {
                let tasks = db.get_tasks_by_user(user_id).await?;
                for task in tasks {
                    println!("{:?}", task);
                }
            }
        },
        Cli::Sub(cmd) => match cmd.cmd {
            SubSubCommand::Info { user_id } => {
                if let Some(sub) = db.get_subscription_by_user(user_id).await? {
                    println!("Subscription: {:?}", sub);
                } else {
                    println!("No subscription found.");
                }
            }
        },
        Cli::Db(cmd) => match cmd.cmd {
            DbSubCommand::Migrate => {
                sqlx::migrate!("./db").run(&db.pool).await?;
                println!("✅ Migrated database");
            }
            DbSubCommand::Seed => {
                sqlx::query!(r#"INSERT INTO users (email, password_hash) VALUES ('admin@example.com', 'hashedpass')"#)
                    .execute(&db.pool)
                    .await?;
                println!("🌱 Seeded database");
            }
        },
    }

    Ok(())
}
```

Update `model.rs` with methods like:

```rust
impl Db {
    pub async fn create_user(&self, email: &str, hash: &str) -> Result<User> {
        sqlx::query_as!(
            User,
            "INSERT INTO users (email, password_hash) VALUES ($1, $2) RETURNING *",
            email,
            hash
        )
        .fetch_one(&self.pool)
        .await
    }

    pub async fn get_users(&self) -> Result<Vec<User>> {
        sqlx::query_as!(User, "SELECT * FROM users")
            .fetch_all(&self.pool)
            .await
    }

    pub async fn get_tasks_by_user(&self, user_id: i32) -> Result<Vec<Task>> {
        sqlx::query_as!(Task, "SELECT * FROM tasks WHERE user_id = $1", user_id)
            .fetch_all(&self.pool)
            .await
    }

    pub async fn get_subscription_by_user(&self, user_id: i32) -> Result<Option<Subscription>> {
        sqlx::query_as!(Subscription, "SELECT * FROM subscriptions WHERE user_id = $1", user_id)
            .fetch_optional(&self.pool)
            .await
    }
}
```

---

## 🧪 Step 3: Try It Out

Build and run:

```bash
cargo build --bin saasctl
./saasctl user create admin@example.com mypassword
./saasctl user list
./saasctl task list 1
./saasctl db migrate
./saasctl db seed
```

Now you have a **powerful CLI interface** for managing your SaaS platform!

---

## 🧱 Step 4: Publish as a Binary Template

You can publish this CLI tool on crates.io or distribute compiled binaries via GitHub Releases.

### Option A: Publish on crates.io

```bash
cargo login
cargo publish
```

Users install it with:

```bash
cargo install saasctl
```

### Option B: Build for Multiple Platforms

Use `cross` to build binaries for Linux/macOS/Windows:

```bash
cargo install cross
cross build --target x86_64-unknown-linux-gnu --release
cross build --target x86_64-apple-darwin --release
cross build --target x86_64-pc-windows-gnu --release
```

Then upload them to GitHub Releases.

---

## 🎉 Final Recap

We now have:

✅ A **Leptos web frontend** with Tailwind + DaisyUI  
✅ An **Axum backend** with PostgreSQL support  
✅ **JWT authentication**  
✅ **Stripe integration**  
✅ A **CLI tool** to manage everything from the terminal  
✅ A **deployable template** ready to be used by others  

This is a **complete SaaS stack in Rust** — fast, safe, scalable, and production-ready.
