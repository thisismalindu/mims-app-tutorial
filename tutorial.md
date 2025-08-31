# MIMS - Microbanking & Interest Management System
## The Ultimate Next.js Full-Stack Guide

**Audience:** This guide is written for absolute beginners. If you have never built a web application before, you are in the right place. We will walk through every concept and step together.

## Table of Contents
1.  [What Are We Building?](#1-what-are-we-building)
2.  [Understanding the Architecture](#2-understanding-the-architecture)
3.  [Prerequisites & Setup](#3-prerequisites--setup)
4.  [Project Setup & First Run](#4-project-setup--first-run)
5.  [Database Design & Setup](#5-database-design--setup)
6.  [Building the Backend (API Routes)](#6-building-the-backend-api-routes)
7.  [Building the Frontend (Pages & Components)](#7-building-the-frontend-pages--components)
8.  [Connecting the Frontend to the Backend](#8-connecting-the-frontend-to-the-backend)
9.  [Authentication & Security](#9-authentication--security)
10. [Deployment to Vercel](#10-deployment-to-vercel)
11. [Development Workflow](#11-development-workflow)
12. [FAQ & Troubleshooting](#12-faq--troubleshooting)

---

## 1. What Are We Building?

We are building a web application for B-Trust Microfinance Bank. The system must allow:
- Managing branches, agents, and customers.
- Customers to open savings accounts under different plans (Children, Teen, etc.).
- Recording deposits and withdrawals.
- Managing Fixed Deposits (FDs) and calculating their interest.
- Generating various reports (transactions, balances, FDs, etc.).

This is a **Full-Stack Application**, meaning it has:
- A **Frontend** (what users see and interact with in the browser).
- A **Backend** (the server that does the logic, calculations, and talks to the database).
- A **Database** (where all the data is stored permanently).

## 2. Understanding the Architecture

We are using **Next.js**, which is a **Full-Stack React Framework**. This is a key point:

- **Traditional Way:** You might build a separate frontend (e.g., with React) and a separate backend (e.g., with Express.js). They communicate over the internet via API calls. This is more complex to set up.
- **Next.js Way:** Next.js lets us build both the frontend and backend **in a single project**. This is simpler and more powerful for our needs.

**Our Tech Stack:**
- **Next.js:** The main framework (handles both frontend and backend).
- **React:** The library for building user interfaces (part of Next.js).
- **PostgreSQL:** Our database (we'll use a free cloud-hosted one).
- **Tailwind CSS:** A CSS framework for styling our app quickly.
- **Vercel:** The platform where we will host our app for free.

## 3. Prerequisites & Setup

Every team member must install these:

1.  **Node.js (v18 or higher):**
    - **What it is:** The runtime that allows us to run JavaScript on our computers (for the backend) and to build our project.
    - **How to install:** Download the "LTS" installer from [nodejs.org](https://nodejs.org/).
    - **Verify:** Open your terminal (Command Prompt on Windows, Terminal on Mac) and run:
      ```bash
      node --version
      npm --version
      ```
      You should see version numbers, not an error.

2.  **Git:**
    - **What it is:** Version control software. It helps us track changes to our code and collaborate as a team.
    - **How to install:** Download from [git-scm.com](https://git-scm.com/).
    - **Verify:** Run `git --version` in your terminal.

3.  **A Code Editor: Visual Studio Code (VSCode)**
    - **Why:** It has excellent support for JavaScript and Next.js.
    - **Download:** [code.visualstudio.com](https://code.visualstudio.com/).
    - **Recommended VSCode Extensions:** After installing, install these extensions within VSCode:
        - `ES7+ React/Redux/React-Native snippets`
        - `Tailwind CSS IntelliSense`
        - `PostgreSQL` (for database management)

4.  **A GitHub Account:** Create a free account on [github.com](https://github.com/). We will use this to store our code.

## 4. Project Setup & First Run

### Step 1: Create the Next.js Project

1.  Open your terminal.
2.  Navigate to the folder where you want to create your project (e.g., `Desktop` or `Documents`).
3.  Run the following command to create a new Next.js app named `mims-app`:

    ```bash
    npx create-next-app@latest mims-app
    ```

4.  You will be asked some questions. Answer them as follows:
    ```
    Would you like to use TypeScript? ... No (*We'll use JavaScript for simplicity*)
    Would you like to use ESLint? ... Yes
    Would you like to use Tailwind CSS? ... Yes (*This is highly recommended*)
    Would you like to use `src/` directory? ... Yes (*Keeps our code organized*)
    Would you like to use App Router? ... Yes (*Uses the modern Next.js structure*)
    Would you like to customize the default import alias? ... No
    ```
5.  Navigate into your new project folder and open it in VSCode:
    ```bash
    cd mims-app
    code .
    ```

### Step 2: Explore the Project Structure

Your project folder will look like this:
```
mims-app/
├── src/
│   ├── app/          # This is where our main pages and layouts live
│   │   ├── globals.css
│   │   ├── layout.js  # Root layout for the entire app
│   │   └── page.js    # The home page (localhost:3000)
│   └── ...
├── public/           # Static files like images
├── next.config.js    # Configuration file for Next.js
├── package.json      # Lists our project's dependencies and scripts
└── tailwind.config.js # Configuration for Tailwind CSS
```

### Step 3: Run the Development Server

1.  In your terminal, make sure you are in the `mims-app` directory.
2.  Run the following command:
    ```bash
    npm run dev
    ```
3.  Open your browser and go to `http://localhost:3000`.
4.  You should see the default Next.js welcome page. Congratulations! Your development server is running.

**Keep this terminal window open.** This server will automatically update your browser whenever you save changes to your code.

## 5. Database Design & Setup

### Part A: Design the Database Tables

Based on the project requirements, we need tables for `branches`, `agents`, `customers`, `accounts`, `transactions`, and `fixed_deposits`.

We will write SQL commands to create these tables. Create a new file in your project called `database-schema.sql`. This file is just for planning; we will run these commands later.

**File: `database-schema.sql`**
```sql
-- Create a database named 'mimsdb' (we will do this on the hosting platform)
-- CREATE DATABASE mimsdb;

-- Table for branches
CREATE TABLE branches (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    location VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table for agents
CREATE TABLE agents (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL, -- We will store hashed passwords, never plain text!
    branch_id INTEGER REFERENCES branches(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table for customers
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    nic VARCHAR(20) UNIQUE NOT NULL, -- National Identity Card number
    phone VARCHAR(15),
    assigned_agent_id INTEGER REFERENCES agents(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table for account types (e.g., Children, Teen, Adult, etc.)
CREATE TABLE account_types (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL, -- e.g., 'Children', 'Teen'
    interest_rate DECIMAL(5, 2) NOT NULL, -- e.g., 12.00
    min_balance DECIMAL(15, 2) NOT NULL -- e.g., 0.00
);

-- Table for savings accounts
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    account_number VARCHAR(20) UNIQUE NOT NULL, -- Should be generated automatically
    current_balance DECIMAL(15, 2) DEFAULT 0.00,
    customer_id INTEGER REFERENCES customers(id) ON DELETE CASCADE,
    account_type_id INTEGER REFERENCES account_types(id) ON DELETE CASCADE,
    is_joint BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table for all transactions (deposits, withdrawals, interest)
CREATE TABLE transactions (
    id SERIAL PRIMARY KEY,
    amount DECIMAL(15, 2) NOT NULL,
    transaction_type VARCHAR(10) NOT NULL, -- 'deposit', 'withdrawal', 'interest'
    reference_number VARCHAR(100),
    account_id INTEGER REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Table for fixed deposits (FDs)
CREATE TABLE fixed_deposits (
    id SERIAL PRIMARY KEY,
    account_id INTEGER REFERENCES accounts(id) ON DELETE CASCADE,
    amount DECIMAL(15, 2) NOT NULL,
    tenure_months INTEGER NOT NULL, -- 6, 12, 36
    interest_rate DECIMAL(5, 2) NOT NULL,
    start_date DATE NOT NULL,
    next_interest_date DATE NOT NULL, -- Calculated: start_date + 30 days
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Part B: Set Up a Cloud Database

We will use **Neon** (https://neon.tech) for a free, cloud-hosted PostgreSQL database.

1.  Go to [Neon.tech](https://neon.tech) and sign up for a free account.
2.  Click "Create a project".
3.  Name your project `mims-database`.
4.  Neon will create a new database for you. On the dashboard, find the **Connection String**. It should look like this:
    `postgresql://username:password@ep-cool-water-123456.us-east-2.aws.neon.tech/database-name?sslmode=require`
5.  Copy this connection string. We will need it soon.

### Part C: Connect Next.js to the Database

1.  In your project, install the PostgreSQL library for Node.js:
    ```bash
    npm install pg
    ```
2.  Create a folder `src/lib` and inside it, create a file `database.js`. This file will handle the connection to our database.

    **File: `src/lib/database.js`**
    ```javascript
    import { Pool } from 'pg';

    // Create a new Pool instance. A Pool manages multiple database connections.
    const pool = new Pool({
      connectionString: process.env.DATABASE_URL, // We will set this environment variable next
      ssl: {
        rejectUnauthorized: false, // Often required for cloud databases like Neon
      },
    });

    // Export a helper function to query the database
    export async function query(text, params) {
      try {
        const client = await pool.connect();
        const result = await client.query(text, params);
        client.release(); // Release the client back to the pool
        return result;
      } catch (err) {
        console.error('Database query error', err);
        throw err; // Re-throw the error so the API route can handle it
      }
    }

    export default pool;
    ```

3.  **Environment Variables:** We need to store our sensitive database connection string securely.
    - In the root of your project, create a file called `.env.local`.
    - Add the following line to it, pasting your Neon connection string:
      ```
      DATABASE_URL="your_neon_connection_string_here"
      ```
    - **CRUCIAL:** Add `.env.local` to your `.gitignore` file. This prevents you from accidentally committing your password to GitHub.

4.  **Create the Tables:** Let's create a simple API route to run our SQL schema. This is a one-time setup task.
    - Create a file `src/app/api/setup-database/route.js`.
    - **Important:** In Next.js App Router, API routes are defined in `app/api/route.js`. The folder name (`setup-database`) becomes the API endpoint (`/api/setup-database`).

    **File: `src/app/api/setup-database/route.js`**
    ```javascript
    import { query } from '@/lib/database'; // We'll use our helper function

    export async function GET() {
      try {
        // Run the SQL commands from our database-schema.sql file
        await query(`
          CREATE TABLE IF NOT EXISTS branches (id SERIAL PRIMARY KEY, name VARCHAR(255) NOT NULL, location VARCHAR(255) NOT NULL, created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
          -- ... Paste the rest of your CREATE TABLE statements here ...
          CREATE TABLE IF NOT EXISTS fixed_deposits (id SERIAL PRIMARY KEY, account_id INTEGER REFERENCES accounts(id) ON DELETE CASCADE, amount DECIMAL(15, 2) NOT NULL, tenure_months INTEGER NOT NULL, interest_rate DECIMAL(5, 2) NOT NULL, start_date DATE NOT NULL, next_interest_date DATE NOT NULL, created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
        `);
        return Response.json({ message: 'Database tables created successfully!' });
      } catch (error) {
        return Response.json({ error: error.message }, { status: 500 });
      }
    }
    ```
5.  **Run the Setup:**
    - Make sure your dev server is running (`npm run dev`).
    - Open your browser and go to `http://localhost:3000/api/setup-database`.
    - You should see a JSON message: `{"message":"Database tables created successfully!"}`.
    - **Note:** You can delete this API route after running it once, or keep it for future resets.

## 6. Building the Backend (API Routes)

The backend is built using **API Routes**. Each file in `src/app/api/` becomes an endpoint. For example, `src/app/api/customers/route.js` is accessible at `http://localhost:3000/api/customers`.

We will use HTTP methods:
- `GET` to retrieve data.
- `POST` to create new data.
- `PUT` to update data.
- `DELETE` to delete data.

### Example: Customers API

Let's create an API to get all customers and add a new customer.

1.  Create the file: `src/app/api/customers/route.js`.

2.  Implement the `GET` and `POST` methods:

**File: `src/app/api/customers/route.js`**
```javascript
import { query } from '@/lib/database';
import { NextResponse } from 'next/server';

// Handle GET requests to /api/customers
export async function GET() {
  try {
    // SQL query to get all customers and their agent's name
    const result = await query(`
      SELECT c.*, a.name AS agent_name 
      FROM customers c 
      INNER JOIN agents a ON c.assigned_agent_id = a.id
    `);
    return NextResponse.json(result.rows);
  } catch (error) {
    return NextResponse.json({ error: error.message }, { status: 500 });
  }
}

// Handle POST requests to /api/customers
export async function POST(request) {
  try {
    // Get the data sent from the frontend
    const { name, nic, phone, assignedAgentId } = await request.json();

    // Validate the data (simple check)
    if (!name || !nic || !assignedAgentId) {
      return NextResponse.json(
        { error: 'Name, NIC, and assigned agent are required.' },
        { status: 400 }
      );
    }

    // Insert the new customer into the database
    const result = await query(
      'INSERT INTO customers (name, nic, phone, assigned_agent_id) VALUES ($1, $2, $3, $4) RETURNING *',
      [name, nic, phone, assignedAgentId]
    );

    // Return the newly created customer
    return NextResponse.json(result.rows[0], { status: 201 });
  } catch (error) {
    return NextResponse.json({ error: error.message }, { status: 500 });
  }
}
```

**Test your API:**
You can test this using a tool like **Thunder Client** (a VSCode extension) or **Postman**.
- **GET Test:** Send a `GET` request to `http://localhost:3000/api/customers`.
- **POST Test:** Send a `POST` request to the same URL with a JSON body:
  ```json
  {
    "name": "Sanduni Perera",
    "nic": "199712345678",
    "phone": "0771234567",
    "assignedAgentId": 1
  }
  ```

**Repeat this process** for all your other entities: `agents`, `accounts`, `transactions`, `fixed_deposits`. Create files like:
- `src/app/api/agents/route.js`
- `src/app/api/accounts/route.js`
- `src/app/api/accounts/[id]/transactions/route.js` (for transactions of a specific account)

## 7. Building the Frontend (Pages & Components)

The frontend is built using **React Components** in the `src/app/` directory. Each `.js` file here becomes a page. `page.js` is the home page (`/`), `customers/page.js` is the customers page (`/customers`).

### Step 1: Create a Layout with Navigation

First, let's modify the root layout to include a navigation bar.

**File: `src/app/layout.js`**
```jsx
import { Inter } from 'next/font/google';
import './globals.css';
import Link from 'next/link';

const inter = Inter({ subsets: ['latin'] });

export const metadata = {
  title: 'MIMS - Microbanking System',
  description: 'Manage microbanking operations',
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>
        {/* Navigation Bar */}
        <nav className="bg-blue-600 p-4 text-white">
          <div className="container mx-auto flex justify-between items-center">
            <Link href="/" className="text-xl font-bold">MIMS</Link>
            <div className="space-x-4">
              <Link href="/" className="hover:underline">Dashboard</Link>
              <Link href="/customers" className="hover:underline">Customers</Link>
              <Link href="/agents" className="hover:underline">Agents</Link>
              <Link href="/transactions" className="hover:underline">Transactions</Link>
            </div>
          </div>
        </nav>

        {/* Main Content */}
        <main className="container mx-auto p-4">
          {children}
        </main>
      </body>
    </html>
  );
}
```

### Step 2: Create the Customers Page

Now, let's create a page to display and add customers. We will use **React Server Components** to fetch data directly on the server.

**File: `src/app/customers/page.js`**
```jsx
import Link from 'next/link';

// This function runs on the SERVER before the page is sent to the user.
async function getCustomers() {
  // This is a server-side fetch to our own API route. It's fast and secure.
  const res = await fetch('http://localhost:3000/api/customers', {
    cache: 'no-store', // Don't cache, always get the latest data
  });

  if (!res.ok) {
    throw new Error('Failed to fetch customers');
  }
  return res.json();
}

export default async function CustomersPage() {
  // This await happens on the server. The user sees nothing until it's done.
  const customers = await getCustomers();

  return (
    <div>
      <h1 className="text-2xl font-bold mb-4">Customers</h1>
      <Link href="/customers/new" className="bg-blue-500 text-white px-4 py-2 rounded mb-4 inline-block">
        Add New Customer
      </Link>
      <table className="min-w-full table-auto">
        <thead>
          <tr className="bg-gray-200">
            <th className="px-4 py-2">ID</th>
            <th className="px-4 py-2">Name</th>
            <th className="px-4 py-2">NIC</th>
            <th className="px-4 py-2">Phone</th>
            <th className="px-4 py-2">Assigned Agent</th>
          </tr>
        </thead>
        <tbody>
          {customers.map((customer) => (
            <tr key={customer.id} className="border-b">
              <td className="px-4 py-2">{customer.id}</td>
              <td className="px-4 py-2">{customer.name}</td>
              <td className="px-4 py-2">{customer.nic}</td>
              <td className="px-4 py-2">{customer.phone}</td>
              <td className="px-4 py-2">{customer.agent_name}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

### Step 3: Create a Form to Add a Customer

We need a form for adding new customers. We'll use a **Client Component** because we need user interaction (typing, submitting a form).

1.  Create a new folder: `src/app/customers/new`.
2.  Inside it, create a file `page.js`.

**File: `src/app/customers/new/page.js`**
```jsx
'use client'; // This directive marks this component as a Client Component

import { useState } from 'react';
import { useRouter } from 'next/navigation';

export default function NewCustomerPage() {
  const [name, setName] = useState('');
  const [nic, setNic] = useState('');
  const [phone, setPhone] = useState('');
  const [assignedAgentId, setAssignedAgentId] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const router = useRouter();

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsLoading(true);

    // Send a POST request to our API route
    const response = await fetch('/api/customers', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ name, nic, phone, assignedAgentId: parseInt(assignedAgentId) }),
    });

    if (response.ok) {
      router.push('/customers'); // Navigate back to the customers list
      router.refresh(); // Refresh the server data for the customers list page
    } else {
      setIsLoading(false);
      // Handle error (e.g., show an error message)
      const errorData = await response.json();
      alert(`Error: ${errorData.error}`);
    }
  };

  return (
    <div>
      <h1 className="text-2xl font-bold mb-4">Add New Customer</h1>
      <form onSubmit={handleSubmit} className="max-w-md">
        <div className="mb-4">
          <label htmlFor="name" className="block text-gray-700">Name</label>
          <input
            type="text"
            id="name"
            value={name}
            onChange={(e) => setName(e.target.value)}
            required
            className="w-full p-2 border border-gray-300 rounded"
          />
        </div>
        {/* Add similar fields for NIC, Phone, and Assigned Agent ID */}
        <div className="mb-4">
          <label htmlFor="nic" className="block text-gray-700">NIC</label>
          <input
            type="text"
            id="nic"
            value={nic}
            onChange={(e) => setNic(e.target.value)}
            required
            className="w-full p-2 border border-gray-300 rounded"
          />
        </div>
        <div className="mb-4">
          <label htmlFor="phone" className="block text-gray-700">Phone</label>
          <input
            type="tel"
            id="phone"
            value={phone}
            onChange={(e) => setPhone(e.target.value)}
            className="w-full p-2 border border-gray-300 rounded"
          />
        </div>
        <div className="mb-4">
          <label htmlFor="agentId" className="block text-gray-700">Assigned Agent ID</label>
          <input
            type="number"
            id="agentId"
            value={assignedAgentId}
            onChange={(e) => setAssignedAgentId(e.target.value)}
            required
            className="w-full p-2 border border-gray-300 rounded"
          />
        </div>
        <button
          type="submit"
          disabled={isLoading}
          className="bg-blue-500 text-white px-4 py-2 rounded disabled:bg-blue-300"
        >
          {isLoading ? 'Adding...' : 'Add Customer'}
        </button>
      </form>
    </div>
  );
}
```

## 8. Connecting the Frontend to the Backend

You've already done it! The connection happens in two ways:

1.  **Server Components:** Using `fetch` inside `src/app/customers/page.js` to get data on the server.
2.  **Client Components:** Using `fetch` inside a event handler (like `handleSubmit`) to send data from the browser to the API route.

The key is the URL used in the `fetch` call:
- `fetch('/api/customers')` - This works because our frontend and backend are hosted on the same domain. Next.js routes the request to the correct API route.

## 9. Authentication & Security

This is a crucial part. We must ensure only authorized users can access the system.

### Step 1: Hash Passwords

Never store plain text passwords. We will use the `bcrypt` library to hash them.

1.  Install `bcrypt`:
    ```bash
    npm install bcrypt
    ```
2.  When creating an agent, hash their password before storing it:

**Example in `src/app/api/agents/route.js` (POST handler):**
```javascript
import bcrypt from 'bcrypt';
// ...
const { name, email, password, branch_id } = await request.json();
const passwordHash = await bcrypt.hash(password, 10); // 10 is the "salt rounds"

await query(
  'INSERT INTO agents (name, email, password_hash, branch_id) VALUES ($1, $2, $3, $4)',
  [name, email, passwordHash, branch_id]
);
// ...
```

### Step 2: Create Login API and Use JWT

1.  Install `jsonwebtoken`:
    ```bash
    npm install jsonwebtoken
    ```
2.  Create a secret key for JWT in your `.env.local` file:
    ```
    JWT_SECRET="your_super_secret_key_here_make_it_very_long"
    ```
3.  Create a login API route: `src/app/api/auth/login/route.js`

**File: `src/app/api/auth/login/route.js`**
```javascript
import { query } from '@/lib/database';
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';
import { NextResponse } from 'next/server';

export async function POST(request) {
  const { email, password } = await request.json();

  // 1. Find the agent by email
  const result = await query('SELECT * FROM agents WHERE email = $1', [email]);
  const agent = result.rows[0];

  if (!agent) {
    return NextResponse.json({ error: 'Invalid credentials' }, { status: 401 });
  }

  // 2. Compare the provided password with the stored hash
  const isPasswordValid = await bcrypt.compare(password, agent.password_hash);
  if (!isPasswordValid) {
    return NextResponse.json({ error: 'Invalid credentials' }, { status: 401 });
  }

  // 3. Create a JWT token
  const token = jwt.sign(
    { agentId: agent.id, email: agent.email }, // Payload (data to store in token)
    process.env.JWT_SECRET, // Secret key
    { expiresIn: '1h' } // Token expires in 1 hour
  );

  // 4. Return the token to the frontend
  const response = NextResponse.json({ message: 'Login successful' });
  // Set the token as an HTTP-only cookie (more secure than local storage)
  response.cookies.set('auth_token', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production', // Secure in production (HTTPS)
    sameSite: 'strict',
    maxAge: 60 * 60, // 1 hour in seconds
  });

  return response;
}
```

### Step 3: Protect API Routes

Create a middleware to check for a valid JWT on protected routes.

1.  Create a file `src/middleware.js`:

**File: `src/middleware.js`**
```javascript
import { NextResponse } from 'next/server';
import jwt from 'jsonwebtoken';

export function middleware(request) {
  // Get the token from the cookie
  const token = request.cookies.get('auth_token')?.value;

  // Define paths that require authentication
  const protectedPaths = ['/api/agents', '/api/customers', '/api/accounts']; // Add your protected API routes
  const isProtectedApiRoute = protectedPaths.some(path => request.nextUrl.pathname.startsWith(path));

  if (isProtectedApiRoute) {
    if (!token) {
      return NextResponse.json({ error: 'Access denied. No token provided.' }, { status: 401 });
    }

    try {
      // Verify the token
      jwt.verify(token, process.env.JWT_SECRET);
      // If valid, let the request continue
      return NextResponse.next();
    } catch (error) {
      return NextResponse.json({ error: 'Invalid token.' }, { status: 401 });
    }
  }

  // For all other requests, just continue
  return NextResponse.next();
}

// Specify which paths this middleware should run on
export const config = {
  matcher: ['/api/:path*'], // Run on all API routes
};
```

Now, any request to `/api/customers`, `/api/agents`, etc., will require a valid JWT cookie, which is only set after a successful login.

## 10. Deployment to Vercel

This is the easiest part!

1.  **Push your code to GitHub:**
    - Create a new repository on GitHub.
    - Follow the instructions to push your local code to it.
    - **Make sure your `.env.local` file is in `.gitignore`!**

2.  **Deploy on Vercel:**
    - Go to [vercel.com](https://vercel.com) and sign up with your GitHub account.
    - Click "Add New..." -> "Project".
    - Import your GitHub repository.
    - Vercel will automatically detect it's a Next.js app.

3.  **Add Environment Variables:**
    - In your Vercel project dashboard, go to "Settings" -> "Environment Variables".
    - Add your `DATABASE_URL` and `JWT_SECRET` here, with the same values you had in your `.env.local` file.
    - Click "Deploy". Vercel will build and deploy your application.

4.  **You're Live!** Vercel will give you a URL (e.g., `https://mims-app.vercel.app`). Your full-stack application is now on the internet!

## 11. Development Workflow

1.  **Plan:** Discuss with your team what feature to build next (e.g., "Interest Calculation").
2.  **Code:**
    - `git pull origin main` to get the latest code.
    - `git checkout -b feature/interest-calculation` to create a new branch for your feature.
    - Write your code. Start the dev server with `npm run dev` and test locally.
3.  **Test:** Test your feature thoroughly.
4.  **Commit and Push:**
    ```bash
    git add .
    git commit -m "feat: add monthly interest calculation API"
    git push origin feature/interest-calculation
    ```
5.  **Pull Request (PR):** On GitHub, create a PR from your feature branch to the `main` branch. Another team member should review your code.
6.  **Merge and Deploy:** Once approved, merge the PR into `main`. This will automatically trigger a new deployment on Vercel.

## 12. FAQ & Troubleshooting

**Q: I get an error `Module not found: Can't resolve 'pg'` or `bcrypt`.**
**A:** You probably installed the package but are getting this error in the browser. Remember, `pg` and `bcrypt` are server-side libraries. They will cause errors if you import them into a Client Component. Only use them in API Routes or Server Components.

**Q: My API route returns a 404 error.**
**A:** Check the file path. API routes must be in `src/app/api/your-route/route.js`. The folder name must be `api`.

**Q: How do I see my database data?**
**A:** You can use the SQL Shell in your Neon dashboard, or use a tool like DBeaver or VSCode's PostgreSQL extension to connect to your Neon database using the connection string.

**Q: I get a "Failed to fetch" error on the frontend.**
**A:** Check if your backend API is working. Test the API route directly in the browser or Thunder Client. The URL in your `fetch` call might be wrong.

**Q: How do I handle images or file uploads?**
**A:** For this project, you likely won't need it. But if you do, you would store the files in a service like AWS S3 or Vercel Blob and only store the file URL in your database.

---
**This guide provides a solid foundation.** As you build, you will encounter specific challenges. Use the official Next.js documentation (https://nextjs.org/docs) and don't hesitate to search for solutions online. Good luck with your project
