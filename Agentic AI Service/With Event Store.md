# Conversational, contextual, stateful AI Servise with Event Store

Creating a conversational, contextual, and stateful AI service in Rust that uses an Event Store for historical data is a multi-faceted task. Lets go through the process step-by-step, covering the key components: designing the architecture, implementing the stateful orchestration, integrating with an Event Store, and writing the conversational logic.

---

### **1. Understanding the Architecture**

The system can be broken down into the following components:

1. **Event Store**:
   - A database or storage system that records all events (e.g., user inputs, system responses, context updates) in an append-only manner.
   - This allows you to replay events to reconstruct the state of the conversation at any point in time.

2. **Stateful Orchestration**:
   - Maintains the current state of the conversation (e.g., user preferences, context, session data).
   - Uses the Event Store to retrieve historical data and update the state based on new events.

3. **Conversational Logic**:
   - Implements the AI's ability to understand and respond to user inputs.
   - Uses the state to provide context-aware responses.

4. **API Layer**:
   - Exposes endpoints for user interaction (e.g., HTTP or WebSocket).

---

### **2. Setting Up the Environment**

To implement this in Rust, you'll need the following crates:

- **`tokio`**: For asynchronous programming.
- **`serde`**: For serialization/deserialization of data.
- **`sqlx`**: For interacting with a relational database (e.g., PostgreSQL) as the Event Store.
- **`uuid`**: For generating unique IDs for events.
- **`actix-web` or `warp`**: For building the API layer.

Add these dependencies to your `Cargo.toml`:

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
sqlx = { version = "0.6", features = ["postgres", "runtime-tokio-native-tls"] }
uuid = { version = "1.0", features = ["v4"] }
actix-web = "4"
```

---

### **3. Designing the Event Store**

The Event Store will store all events related to the conversation. Each event will have the following structure:

```rust
#[derive(Debug, Clone, serde::Serialize, serde::Deserialize)]
pub struct Event {
    pub id: uuid::Uuid,
    pub session_id: uuid::Uuid,
    pub event_type: String,
    pub payload: serde_json::Value,
    pub timestamp: chrono::DateTime<chrono::Utc>,
}
```

#### SQL Schema for the Event Store

```sql
CREATE TABLE events (
    id UUID PRIMARY KEY,
    session_id UUID NOT NULL,
    event_type TEXT NOT NULL,
    payload JSONB NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

---

### **4. Implementing Stateful Orchestration**

The orchestrator will manage the state of the conversation by reading from the Event Store and applying events to reconstruct the state.

#### Example: State Struct

```rust
#[derive(Debug, Default, Clone)]
pub struct ConversationState {
    pub session_id: uuid::Uuid,
    pub context: serde_json::Value, // Stores contextual information
}

impl ConversationState {
    pub fn apply_event(&mut self, event: &Event) {
        match event.event_type.as_str() {
            "user_input" => {
                // Update context based on user input
                if let Some(new_context) = event.payload.get("context") {
                    self.context = new_context.clone();
                }
            }
            "system_response" => {
                // Handle system response events
            }
            _ => {}
        }
    }
}
```

#### Reconstructing State from Events

```rust
pub async fn reconstruct_state(
    pool: &sqlx::PgPool,
    session_id: &uuid::Uuid,
) -> Result<ConversationState, sqlx::Error> {
    let events = sqlx::query_as!(
        Event,
        "SELECT * FROM events WHERE session_id = $1 ORDER BY timestamp ASC",
        session_id
    )
    .fetch_all(pool)
    .await?;

    let mut state = ConversationState {
        session_id: *session_id,
        ..Default::default()
    };

    for event in events {
        state.apply_event(&event);
    }

    Ok(state)
}
```

---

### **5. Writing Conversational Logic**

The conversational logic will use the state to generate responses. You can integrate an external NLP model (e.g., via an API) or implement simple rule-based logic.

#### Example: Simple Rule-Based Response

```rust
pub fn generate_response(state: &ConversationState, user_input: &str) -> String {
    if user_input.contains("hello") {
        "Hello! How can I assist you today?".to_string()
    } else if user_input.contains("thank you") {
        "You're welcome!".to_string()
    } else {
        "I'm sorry, I didn't understand that.".to_string()
    }
}
```

---

### **6. Building the API Layer**

Use `actix-web` to expose an endpoint for user interaction.

#### Example: API Endpoint

```rust
use actix_web::{web, App, HttpServer, Responder};
use sqlx::PgPool;

async fn handle_message(
    pool: web::Data<PgPool>,
    message: web::Json<serde_json::Value>,
) -> impl Responder {
    let session_id = message.get("session_id").and_then(|v| v.as_str()).unwrap_or("");
    let user_input = message.get("text").and_then(|v| v.as_str()).unwrap_or("");

    let session_id = uuid::Uuid::parse_str(session_id).unwrap_or_else(|_| uuid::Uuid::new_v4());
    let mut state = reconstruct_state(&pool, &session_id).await.unwrap();

    let response = generate_response(&state, user_input);

    // Save the user input and system response as events
    let user_event = Event {
        id: uuid::Uuid::new_v4(),
        session_id,
        event_type: "user_input".to_string(),
        payload: serde_json::json!({ "text": user_input }),
        timestamp: chrono::Utc::now(),
    };

    let system_event = Event {
        id: uuid::Uuid::new_v4(),
        session_id,
        event_type: "system_response".to_string(),
        payload: serde_json::json!({ "text": response }),
        timestamp: chrono::Utc::now(),
    };

    sqlx::query!(
        "INSERT INTO events (id, session_id, event_type, payload, timestamp) VALUES ($1, $2, $3, $4, $5)",
        user_event.id,
        user_event.session_id,
        user_event.event_type,
        user_event.payload,
        user_event.timestamp
    )
    .execute(&pool)
    .await
    .unwrap();

    sqlx::query!(
        "INSERT INTO events (id, session_id, event_type, payload, timestamp) VALUES ($1, $2, $3, $4, $5)",
        system_event.id,
        system_event.session_id,
        system_event.event_type,
        system_event.payload,
        system_event.timestamp
    )
    .execute(&pool)
    .await
    .unwrap();

    web::Json(serde_json::json!({ "response": response }))
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    let database_url = "postgres://user:password@localhost/dbname";
    let pool = PgPool::connect(database_url).await.unwrap();

    HttpServer::new(move || {
        App::new()
            .app_data(web::Data::new(pool.clone()))
            .route("/message", web::post().to(handle_message))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

---

### **7. Running the Service**

1. Set up a PostgreSQL database and create the `events` table.
2. Run the Rust application using `cargo run`.
3. Send POST requests to `http://127.0.0.1:8080/message` with JSON payloads like:

```json
{
  "session_id": "your-session-id",
  "text": "Hello!"
}
```

---

### **Final Notes**

This implementation provides a foundation for a conversational, stateful AI service. You can extend it by:

- Adding more sophisticated NLP models.
- Implementing advanced state management (e.g., using CQRS/ES patterns).
- Scaling the service with distributed systems.
