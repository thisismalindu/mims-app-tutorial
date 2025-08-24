# MIMS - Full-Stack Development Tutorial

## Table of Contents
1.  [Prerequisites & Setup](#1-prerequisites--setup)
2.  [Project Structure & Git](#2-project-structure--git)
3.  [Backend: Building the API](#3-backend-building-the-api)
4.  [Frontend: Building the UI](#4-frontend-building-the-ui)
5.  [Connecting Frontend to Backend](#5-connecting-frontend-to-backend)
6.  [Deployment (Hosting)](#6-deployment-hosting)
7.  [Development Workflow](#7-development-workflow)
8.  [FAQ & Troubleshooting](#8-faq--troubleshooting)

---

## 1. Prerequisites & Setup

Before you start, every team member must install these on their computer:

1.  **Node.js (v18 or higher):** Download and run the installer from [nodejs.org](https://nodejs.org/). This also installs `npm`.
    *   *Verify:* Open a terminal/command prompt and run `node --version` and `npm --version`.

2.  **Git:** Download and install from [git-scm.com](https://git-scm.com/).
    *   *Verify:* Run `git --version`.

3.  **A Code Editor:** We recommend **Visual Studio Code** ([code.visualstudio.com](https://code.visualstudio.com/)).

4.  **PostgreSQL (Optional for Local Dev):** It's best to have a local database.
    *   **Windows/macOS:** Download and install from [postgresql.org](https://www.postgresql.org/download/). Remember the password you set for the `postgres` user.
    *   **Easier Alternative (Recommended):** Use a hosted database from the start (Railway) as described in the Deployment section. You can skip local PostgreSQL setup if it's complex.

---

## 2. Project Structure & Git

### Initialize the Project and Git
```bash
# Navigate to your project folder
cd path/to/mims-app

# Initialize a main Git repository (this is optional but good for organizing)
git init

# Navigate into the backend folder and initialize it as a Node.js project
cd backend
npm init -y

# Navigate into the frontend folder and create a React app using Vite
cd ../frontend
npm create vite@latest . -- --template react
# Follow the prompts. Select 'React' and 'JavaScript'.

# Install frontend dependencies
npm install
```

### Project Structure
Your folder should now look like this:
```
mims-app/
├── .git/                  # Main Git repository
├── backend/               # Node.js/Express API
│   ├── node_modules/
│   ├── package.json       # Backend dependencies and scripts
│   └── ... (other files will be created)
└── frontend/              # React Application
    ├── node_modules/
    ├── public/
    ├── src/
    ├── index.html
    ├── package.json       # Frontend dependencies and scripts
    └── vite.config.js
```

### Basic Git Setup
Create a `.gitignore` file in the root `mims-app/` folder with this content:
```
# Dependencies
backend/node_modules/
frontend/node_modules/

# Environment variables
backend/.env
frontend/.env

# Database
*.db
*.sqlite

# Logs
*.log

# Runtime data
pids
*.pid
*.seed
*.pid.lock

# Coverage directory used by tools like istanbul
coverage/
```
**First Commit:**
```bash
# Add all files to staging
git add .

# Make the first commit
git commit -m "Initial project structure with backend and frontend folders"
```

---

## 3. Backend: Building the API

**All commands are run from the `/backend` directory.**

### Step 1: Install Dependencies
```bash
# Core framework and server
npm install express
npm install -D nodemon # Tool to auto-restart server on changes

# Database ORM and connector
npm install prisma
npm install -D @prisma/client
npx prisma init # This creates the prisma folder and schema

# Authentication and security
npm install bcryptjs jsonwebtoken

# Other utilities
npm install cors dotenv
```

### Step 2: Configure Prisma and Database
1.  Open `prisma/schema.prisma`. This file defines your database structure.
2.  **Define Your Data Model:** Replace the content with your tables. Here's a starter for the `Customer` and `Agent` models. You will add more (Account, Transaction, etc.) later.
    ```prisma
    // prisma/schema.prisma
    generator client {
      provider = "prisma-client-js"
    }

    datasource db {
      provider = "postgresql"
      url      = env("DATABASE_URL") // We'll set this in a .env file
    }

    model Branch {
      id      Int      @id @default(autoincrement())
      name    String
      location String
      agents  Agent[]
      // Add other fields later
      @@map("branches")
    }

    model Agent {
      id        Int      @id @default(autoincrement())
      name      String
      branchId  Int
      branch    Branch   @relation(fields: [branchId], references: [id], onDelete: Cascade)
      customers Customer[]

      @@map("agents")
    }

    model Customer {
      id        Int      @id @default(autoincrement())
      name      String
      nic       String   @unique // National Identity Card Number
      phone     String?
      assignedAgentId Int
      agent     Agent    @relation(fields: [assignedAgentId], references: [id], onDelete: Cascade)
      // We'll add 'accounts' relation later

      @@map("customers")
    }
    ```
3.  **Create a `.env` file** in the `/backend` folder:
    ```bash
    # For Local PostgreSQL (if you installed it)
    DATABASE_URL="postgresql://postgres:your_password@localhost:5432/mimsdb?schema=public"

    # For Hosted PostgreSQL (on Railway - get this URL later)
    # DATABASE_URL="postgresql://johndoe:abc123@@server.railway.com:5432/railway"

    JWT_SECRET="your_super_secret_jwt_key_make_this_very_long_and_random"
    ```
    *Replace `your_password` with the password you set for PostgreSQL.*

### Step 3: Create the Database and Tables
```bash
# This command reads your schema.prisma and creates SQL commands to create the tables.
npx prisma migrate dev --name init

# This generates the Prisma Client code, which we use to query the database.
npx prisma generate
```

### Step 4: Build the Express Server
1.  Create the main server file: `backend/src/index.js`
    ```javascript
    // backend/src/index.js
    const express = require('express');
    const cors = require('cors');
    require('dotenv').config();

    const app = express();
    const PORT = process.env.PORT || 3001;

    // Middleware
    app.use(cors()); // Allows our React app to call this API
    app.use(express.json()); // Lets us parse JSON from requests

    // Basic health check route
    app.get('/api/health', (req, res) => {
      res.json({ message: 'MIMS Backend Server is running!' });
    });

    // TODO: Import and use routes here
    // app.use('/api/customers', customerRoutes);

    // Start the server
    app.listen(PORT, () => {
      console.log(`Server is listening on port ${PORT}`);
    });
    ```
2.  **Update `package.json` scripts** to run the server easily:
    ```json
    "scripts": {
      "start": "node src/index.js",
      "dev": "nodemon src/index.js"
    }
    ```
3.  **Test the server:** Run `npm run dev`. Open your browser and go to `http://localhost:3001/api/health`. You should see the JSON message.

### Step 5: Create Your First API Route
1.  Create a folder: `backend/src/routes`
2.  Create a file: `backend/src/routes/customerRoutes.js`
    ```javascript
    // backend/src/routes/customerRoutes.js
    const express = require('express');
    const { PrismaClient } = require('@prisma/client');

    const router = express.Router();
    const prisma = new PrismaClient();

    // GET /api/customers - Get all customers
    router.get('/', async (req, res) => {
      try {
        const customers = await prisma.customer.findMany({
          include: { agent: true }, // Include agent data in the response
        });
        res.json(customers);
      } catch (error) {
        res.status(500).json({ error: 'Failed to fetch customers' });
      }
    });

    // POST /api/customers - Create a new customer
    router.post('/', async (req, res) => {
      const { name, nic, phone, assignedAgentId } = req.body;
      try {
        const newCustomer = await prisma.customer.create({
          data: { name, nic, phone, assignedAgentId },
        });
        res.status(201).json(newCustomer);
      } catch (error) {
        res.status(400).json({ error: 'Failed to create customer' });
      }
    });

    module.exports = router;
    ```
3.  **Import the route in `index.js`:**
    ```javascript
    // Add near the top of index.js
    const customerRoutes = require('./routes/customerRoutes');

    // Add this line where the TODO comment was
    app.use('/api/customers', customerRoutes);
    ```
4.  **Test the API:** Use a tool like **Thunder Client (VSCode extension)** or **Postman** to send a `POST` request to `http://localhost:3001/api/customers` with a JSON body:
    ```json
    {
      "name": "John Doe",
      "nic": "199712345678",
      "phone": "0771234567",
      "assignedAgentId": 1
    }
    ```
    Then send a `GET` request to the same URL to see your created customer.

**Next Steps for Backend:** Repeat this process for other routes: `agentRoutes.js`, `accountRoutes.js`, `transactionRoutes.js`. Implement all the business logic (e.g., in `transactionRoutes.js`, check balance before allowing a withdrawal).

---

## 4. Frontend: Building the UI

**All commands are run from the `/frontend` directory.**

### Step 1: Install Additional Dependencies
```bash
# For making API calls to our backend
npm install axios

# For routing between different pages (e.g., Login, Dashboard, Customer List)
npm install react-router-dom
```

### Step 2: Clean Up and Structure
1.  Remove unnecessary files: `src/App.css`, `src/index.css`. You can keep them if you want, but we'll use Tailwind.
2.  **Install and Configure Tailwind CSS (Optional but Recommended):**
    ```bash
    npm install -D tailwindcss postcss autoprefixer
    npx tailwindcss init -p
    ```
    Configure `tailwind.config.js`:
    ```js
    /** @type {import('tailwindcss').Config} */
    export default {
      content: [
        "./index.html",
        "./src/**/*.{js,ts,jsx,tsx}",
      ],
      theme: {
        extend: {},
      },
      plugins: [],
    }
    ```
    Replace the content of `src/index.css` with:
    ```css
    @tailwind base;
    @tailwind components;
    @tailwind utilities;
    ```

### Step 3: Create Components and Pages
Create a logical structure inside `src/`:
```
src/
├── components/     # Reusable pieces (Navbar, CustomerCard)
├── pages/         # Full pages (Dashboard, CustomersPage)
├── services/      # Code for API calls (api.js)
├── App.jsx
└── main.jsx
```

1.  **Create an API service:** `src/services/api.js`
    ```javascript
    // src/services/api.js
    import axios from 'axios';

    // Create an axios instance pointing to our backend URL
    // In development, it's localhost. After deployment, we'll change this.
    const API = axios.create({
      baseURL: 'http://localhost:3001/api', // Your backend URL
    });

    export default API;
    ```

2.  **Create a Page:** `src/pages/CustomersPage.jsx`
    ```jsx
    // src/pages/CustomersPage.jsx
    import { useState, useEffect } from 'react';
    import API from '../services/api';

    function CustomersPage() {
      const [customers, setCustomers] = useState([]);

      useEffect(() => {
        // Fetch customers from the backend when the component loads
        const fetchCustomers = async () => {
          try {
            const response = await API.get('/customers');
            setCustomers(response.data);
          } catch (error) {
            console.error('Error fetching customers:', error);
          }
        };
        fetchCustomers();
      }, []);

      return (
        <div className="p-4">
          <h1 className="text-2xl font-bold mb-4">Customers</h1>
          <ul>
            {customers.map((customer) => (
              <li key={customer.id} className="border p-2 my-2">
                {customer.name} - {customer.nic}
              </li>
            ))}
          </ul>
        </div>
      );
    }
    export default CustomersPage;
    ```

3.  **Set Up Routing:** Modify `src/App.jsx`
    ```jsx
    // src/App.jsx
    import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
    import CustomersPage from './pages/CustomersPage';

    function App() {
      return (
        <Router>
          <div>
            <nav className="bg-blue-600 p-4 text-white">
              <Link to="/" className="mr-4">Dashboard</Link>
              <Link to="/customers">Customers</Link>
            </nav>
            <Routes>
              <Route path="/" element={<h1 className="p-4">Dashboard</h1>} />
              <Route path="/customers" element={<CustomersPage />} />
            </Routes>
          </div>
        </Router>
      );
    }
    export default App;
    ```

4.  **Test the Frontend:** Run `npm run dev`. Go to `http://localhost:5173` and click on "Customers". You should see the customer you created via the API!

---

## 5. Connecting Frontend to Backend

This is already done in the previous step! The key is the `baseURL` in `src/services/api.js`. The frontend runs on `http://localhost:5173` and the backend on `http://localhost:3001`. The `cors()` middleware in the backend allows this cross-origin request.

**Important:** Before deployment, you will need to change the `baseURL` to point to your live backend URL (e.g., `https://mims-backend.railway.app/api`).

---

## 6. Deployment (Hosting)

### Part A: Deploy Database & Backend to Railway
1.  Create accounts on [GitHub](https://github.com/), [Railway](https://railway.app/), and [Vercel](https://vercel.com/).
2.  Push your code to GitHub. Create two separate repositories: one for `backend` and one for `frontend`.
3.  **On Railway:**
    *   Create a new Project.
    *   Click "New" and choose "PostgreSQL". This creates your database. Go to the "Variables" tab and copy the `DATABASE_URL`.
    *   Now, connect your `backend` GitHub repo to Railway.
    *   In the Railway project settings, add a variable: `JWT_SECRET` with a long random string.
    *   Railway will automatically detect it's a Node.js app and run `npm install` and `npm start`. Your backend is now live! Railway will give you a URL like `https://mims-backend.up.railway.app`.

### Part B: Deploy Frontend to Vercel
1.  **On Vercel:** Click "Add New..." -> "Project".
2.  Import your `frontend` GitHub repository.
3.  In the "Build and Output Settings":
    *   **Framework Preset:** Vite
    *   **Root Directory:** `frontend` (if your repo is just the frontend code)
4.  **Crucial Step:** Add an **Environment Variable** for the build process.
    *   **Name:** `VITE_API_BASE_URL`
    *   **Value:** The URL of your live backend on Railway (e.g., `https://mims-backend.up.railway.app/api`)
5.  Click "Deploy". Your frontend is now live!
6.  **Update your frontend code:** In `src/services/api.js`, change the `baseURL` to use the environment variable:
    ```javascript
    const API = axios.create({
      baseURL: import.meta.env.VITE_API_BASE_URL, // This will now use the variable from Vercel
    });
    ```
    Push this change to GitHub. Vercel will automatically redeploy.

---

## 7. Development Workflow

1.  **Start Working:** Always pull the latest changes from the team: `git pull origin main`
2.  **Create a Feature Branch:** `git checkout -b feature/add-new-report`
3.  **Do Your Work:** Write code for your specific task (e.g., adding a report page).
4.  **Test Locally:** Make sure your backend (`npm run dev`) and frontend (`npm run dev`) are running and working together.
5.  **Commit Your Changes:**
    ```bash
    git add .
    git commit -m "feat: add monthly interest report page and API endpoint"
    ```
6.  **Push and Create a Pull Request:**
    ```bash
    git push origin feature/add-new-report
    ```
    Then go to GitHub, and you'll see a prompt to create a Pull Request (PR) to merge your branch into `main`.
7.  **Review & Merge:** Another team member reviews your code. Once approved, the PR is merged. This triggers automatic deployment on Railway and Vercel.

---

## 8. FAQ & Troubleshooting

**Q: I get a `CORS` error in the browser when my frontend tries to talk to the backend.**
**A:** Double-check you have `app.use(cors());` in your backend `index.js` file.

**Q: I get a `prisma` command not found error.**
**A:** You installed Prisma locally in the project. Always use `npx prisma ...` instead of just `prisma ...`.

**Q: My changes in the frontend/backend aren't showing up.**
**A:** 1) Did you save the file? 2) Is your dev server still running? 3) For backend changes, did `nodemon` restart? 4) Hard refresh your browser (Ctrl+F5).

**Q: How do I see my database data?**
**A:** Run `npx prisma studio` in your backend folder. It opens a browser window to view and edit your database tables.

**Q: Deployment failed on Railway/Vercel.**
**A:** Check the build logs on their websites. The error is almost always there. Common issues:
*   Missing environment variables (`DATABASE_URL`, `JWT_SECRET`).
*   A syntax error in your code.

**Q: Why are we using two separate GitHub repos?**
**A:** It simplifies deployment. Vercel is designed to deploy a single React app, and Railway is designed to deploy a single Node.js app. Managing them separately is much easier for this project structure.
