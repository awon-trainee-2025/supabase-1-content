## Table of Contents
1. [Introduction to Supabase](#introduction-to-supabase)
2. [Setting Up Supabase](#setting-up-supabase)
4. [Database Fundamentals](#database-fundamentals)
5. [Supabase with Flutter](#supabase-with-flutter)
6. [Authentication](#authentication)
7. [Database Operations](#database-operations)
8. [Advanced Features](#advanced-features)

---

## Introduction to Supabase

### What is Supabase?
Supabase is an open-source Firebase alternative that provides:
- PostgreSQL database
- Authentication
- Real-time subscriptions
- Storage
### Key Features
- PostgreSQL Database: Full SQL database capabilities
- Instant APIs: Auto-generated REST and GraphQL APIs
- Authentication: Built-in user management
- Realtime: Listen to database changes
- Storage: File storage with CDN

---

## Setting Up Supabase
### 1. Create Supabase Account
1. Go to [supabase.com](https://supabase.com)
2. Click "Start your project"
3. Sign up with GitHub or email
### 2. Create New Project
```markdown
1. Click "New Project"
2. Enter project name
3. Select region closest to your users
4. Set database password (save this securely)
5. Click "Create new project"
```
### 3. Get API Credentials
After project creation:
1. Go to Project Settings → API
2. Note:
     - Project URL
     - Public (anon) key

---

## Database Fundamentals
### Creating Tables
1. Go to Table Editor in Supabase dashboard
2. Click "Create a new table"
3. Define columns:
    - Name
    - Data type (text, int8, bool, timestamp, etc.)
    - Default value
    - Constraints (primary key, nullable, unique)
### Example: Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email TEXT NOT NULL UNIQUE,
  name TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```
### Defining Relationships
### One-to-Many:
```sql
CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  title TEXT NOT NULL,
  content TEXT
);
```
### Many-to-Many:
```sql
CREATE TABLE bookmarks (
  user_id UUID REFERENCES users(id),
  post_id INT REFERENCES posts(id),
  PRIMARY KEY (user_id, post_id)
);
```
### Indexing
Create indexes for frequently queried columns:
```sql
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_users_email ON users(email);
```

---

## Supabase with Flutter
1. Add Dependencies
```yaml
dependencies:
  supabase_flutter: ^2.0.0
  flutter_dotenv: ^5.1.0 # For environment variables
```
2. Initialize Supabase
```dart
import 'package:supabase_flutter/supabase_flutter.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  await Supabase.initialize(
    url: 'YOUR_SUPABASE_URL',
    anonKey: 'YOUR_SUPABASE_ANON_KEY',
  );
  
  runApp(MyApp());
}
```
3. Get Supabase Client
```dart
final supabase = Supabase.instance.client;
```

---

## Authentication
### User Registration
```dart
Future<void> signUp(String email, String password) async {
  final response = await supabase.auth.signUp(
    email: email,
    password: password,
  );
  
  if (response.user == null) {
    throw Exception('Signup failed');
  }
}
```

### User Login
```dart
Future<void> signIn(String email, String password) async {
  final response = await supabase.auth.signInWithPassword(
    email: email,
    password: password,
  );
  
  if (response.user == null) {
    throw Exception('Login failed');
  }
}
```

### Current User Session
```dart
final user = supabase.auth.currentUser;
if (user != null) {
  print('Logged in as ${user.email}');
} else {
  print('Not logged in');
}
```

### Sign Out
```dart
Future<void> signOut() async {
  await supabase.auth.signOut();
}
```

---

## Database Operations
### SELECT (Read Data)
```dart
// Get all posts
final data = await supabase
  .from('posts')
  .select('*')
  .order('created_at', ascending: false);

// Get posts with user info
final postsWithUsers = await supabase
  .from('posts')
  .select('*, users(name, avatar_url)')
  .eq('is_published', true);
```

### INSERT (Create Data)
```dart
await supabase
  .from('posts')
  .insert({
    'title': 'New Post',
    'content': 'This is my first post',
    'user_id': supabase.auth.currentUser!.id,
  });
```

### UPDATE (Modify Data)
```dart
await supabase
  .from('posts')
  .update({'title': 'Updated Title'})
  .eq('id', postId);
```

### DELETE (Remove Data)
```dart
await supabase
  .from('posts')
  .delete()
  .eq('id', postId);
```

---

## Advanced Features
### Realtime Subscriptions
```dart
final subscription = supabase
  .from('posts')
  .on(SupabaseEventTypes.all, (payload) {
    print('Change received: ${payload.newRecord}');
  })
  .subscribe();

// Later...
supabase.removeSubscription(subscription);
```
