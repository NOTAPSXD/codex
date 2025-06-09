Great! Let's proceed with setting up the backend for the **Code X Verse** SaaS platform using Node.js and Express. We’ll go through the following steps:

### Step 6: Backend Setup

#### 1. Initialize the Backend

Navigate to the root of your project (i.e., one level up from the `frontend` directory) and create a new folder named `backend`. Then, navigate into the `backend` folder and initialize a new Node.js project.

```bash
mkdir backend
cd backend
npm init -y
```

#### 2. Install Required Packages

Install the necessary packages for your backend:

```bash
npm install express firebase-admin cors body-parser axios dotenv
```

Here’s what each package does:
- **express**: Minimalist web framework to build the API.
- **firebase-admin**: Admin SDK for interacting with Firebase.
- **cors**: Middleware to enable CORS (Cross-Origin Resource Sharing).
- **body-parser**: Middleware to parse incoming request bodies in a middleware before your handlers.
- **axios**: Promise-based HTTP client for making API requests (for Discord webhook).
- **dotenv**: Loads environment variables from a `.env` file.

#### 3. Create Environment File

Create a `.env` file in the `backend` folder to store your environment variables:

```bash
touch .env
```

Fill it with the following content (replace placeholder values with your actual Firebase and Discord details):

```plaintext
FIREBASE_PROJECT_ID=your_firebase_project_id
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_API_KEY=your_google_api_key
DISCORD_WEBHOOK_URL=your_discord_webhook_url
FRONTEND_URL=http://your_frontend_url
```

#### 4. Set Up Firebase Admin SDK

Create a `firebase.js` file in the `backend` folder to initialize Firebase Admin SDK:

```javascript
// backend/firebase.js
const admin = require('firebase-admin');
const serviceAccount = require('./path/to/your/firebase/serviceAccountKey.json'); // Download from Firebase console

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: `https://${process.env.FIREBASE_PROJECT_ID}.firebaseio.com`,
});

const db = admin.firestore();

module.exports = { admin, db };
```

Make sure to replace `path/to/your/firebase/serviceAccountKey.json` with the path to your Firebase service account key file, which you can generate in the Firebase Console.

#### 5. Set Up Express Server

Create a `server.js` file to bootstrap the Express application:

```javascript
// backend/server.js
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const { authRoutes } = require('./routes/auth');
const { userRoutes } = require('./routes/users');
const { logRoutes } = require('./routes/logs');
const { productRoutes } = require('./routes/products');
const { chatRoutes } = require('./routes/chat');
const { ticketRoutes } = require('./routes/tickets');

const app = express();
app.use(cors());
app.use(bodyParser.json());

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);
app.use('/api/logs', logRoutes);
app.use('/api/products', productRoutes);
app.use('/api/chat', chatRoutes);
app.use('/api/tickets', ticketRoutes);

// Start the server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

### Step 7: Create API Routes

Now we will create the routes necessary for handling authentication, user management, logging, product management, chat, and tickets. 

#### 1. Create Routes Directories

Create a `routes` folder inside the `backend` directory:

```bash
mkdir routes
```

#### 2. Auth Routes

Create `auth.js` inside the `routes` folder:

```javascript
// backend/routes/auth.js
const express = require('express');
const { admin } = require('../firebase');
const router = express.Router();

router.post('/register', async (req, res) => {
  const { email, password } = req.body;

  try {
    const userRecord = await admin.auth().createUser({ email, password });
    res.status(201).send({ uid: userRecord.uid });
  } catch (error) {
    res.status(400).send({ error: error.message });
  }
});

router.post('/login', async (req, res) => {
  const { email, password } = req.body;

  // Handle login logic with Firebase
  // Using Firebase SDK for login
});

router.post('/google/callback', async (req, res) => {
  // Handle Google OAuth callback
});

module.exports = { authRoutes: router };
```

#### 3. User Routes

Create `users.js` inside the `routes` folder:

```javascript
// backend/routes/users.js
const express = require('express');
const { admin } = require('../firebase');
const router = express.Router();

// Middleware to verify admin access
const verifyAdmin = async (req, res, next) => {
  const user = await admin.auth().verifyIdToken(req.headers.authorization);
  if (user.role === 'admin') {
    next();
  } else {
    res.status(403).send('Forbidden');
  }
};

router.get('/', verifyAdmin, async (req, res) => {
  // List users for admin
});

router.post('/create', verifyAdmin, async (req, res) => {
  // Create user logic
});

router.post('/promote', verifyAdmin, async (req, res) => {
  // Promote user to admin
});

router.post('/delete', verifyAdmin, async (req, res) => {
  // Delete user logic
});

router.post('/changepass', verifyAdmin, async (req, res) => {
  // Change user password logic
});

module.exports = { userRoutes: router };
```

#### 4. Log Routes

Create `logs.js` inside the `routes` folder:

```javascript
// backend/routes/logs.js
const express = require('express');
const router = express.Router();

router.get('/', async (req, res) => {
  // Fetch logs logic (download, login, moderation)
});

module.exports = { logRoutes: router };
```

#### 5. Product Routes

Create `products.js` inside the `routes` folder:

```javascript
// backend/routes/products.js
const express = require('express');
const { db } = require('../firebase');
const router = express.Router();

router.post('/', async (req, res) => {
  const { name, price, type } = req.body; // type can be 'free' or 'paid'
  try {
    const productRef = await db.collection('products').add({ name, price, type });
    res.status(201).send({ id: productRef.id });
  } catch (error) {
    res.status(400).send({ error: error.message });
  }
});

router.get('/', async (req, res) => {
  // Fetch all products
});

module.exports = { productRoutes: router };
```

#### 6. Chat Routes

Create `chat.js` inside the `routes` folder:

```javascript
// backend/routes/chat.js
const express = require('express');
const axios = require('axios');
const router = express.Router();

router.post('/ai', async (req, res) => {
  const { message } = req.body;

  try {
    const response = await axios.post(`https://gemini.api.url`, { input: message }); // Replace with actual API call
    res.send({ response: response.data });
  } catch (error) {
    res.status(500).send({ error: 'Chat API error' });
  }
});

module.exports = { chatRoutes: router };
```

#### 7. Ticket Routes

Create `tickets.js` inside the `routes` folder:

```javascript
// backend/routes/tickets.js
const express = require('express');
const { db } = require('../firebase');
const router = express.Router();

router.post('/', async (req, res) => {
  const { userId, issue } = req.body; // Example data
  try {
    const ticketRef = await db.collection('tickets').add({ userId, issue });
    res.status(201).send({ id: ticketRef.id });
  } catch (error) {
    res.status(400).send({ error: error.message });
  }
});

module.exports = { ticketRoutes: router };
```

### Step 8: Discord Webhook Integration

We will now set up the Discord webhook for logging events.

Create a `discordWebhook.js` file in the `services` folder (create the `services` folder if it does not exist):

```javascript
// backend/services/discordWebhook.js
const axios = require('axios');

const logToDiscord = async (message) => {
  try {
    const embed = {
      content: message,
      // Customize with other embed parameters if needed
    };
    await axios.post(process.env.DISCORD_WEBHOOK_URL, { embeds: [embed] });
  } catch (error) {
    console.error('Error logging to Discord:', error);
  }
};

module.exports = { logToDiscord };
```

### Next Steps

Now we've built the foundational structure for both the frontend and backend of your **Code X Verse** platform. Here’s a summary of what we achieved:

- We set up the backend with Node.js and Express.
- We configured Firebase Admin SDK to manage authentication and Firestore for data storage.
- We created robust API routes to handle user authentication, user management, chat interactions, product management, and ticket creation.
- We integrated Discord webhook logging for certain events.

Would you like to proceed with deployment steps and logging functionality implementation, or is there anything specific you want to dive deeper into?
