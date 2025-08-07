# Carthus: Project Guide

## 1. Project Overview

Carthus, also known as "Elijeweb," is a web application designed as an AI sales assistant primarily for businesses operating on Meta platforms (Facebook, Instagram). It provides a dashboard for users to manage their e-commerce operations, including product inventory, customer orders, and sales. The system features user authentication, a RESTful API for data management, and integrates with a PostgreSQL database via Prisma ORM, aiming to streamline sales processes and stock management.

## 2. Key Technologies & Dependencies

The project leverages a modern JavaScript ecosystem, focusing on a full-stack Next.js approach.

*   **Languages**: TypeScript
*   **Framework**: Next.js (React Framework)
*   **UI Library**: React
*   **Styling**: Tailwind CSS, PostCSS, Autoprefixer
*   **Authentication**: NextAuth.js (with Credentials Provider and Prisma Adapter)
*   **Database ORM**: Prisma (for PostgreSQL)
*   **Database**: PostgreSQL
*   **Password Hashing**: bcrypt
*   **Linting**: ESLint

## 3. File Structure Analysis

The project follows the Next.js App Router structure, organizing code logically based on functionality and routing.

*   `app/`: Contains the core application logic, including pages, API routes, and the root layout.
    *   `app/api/`: Defines API endpoints.
        *   `app/api/auth/`: Handles NextAuth.js authentication routes (`[...nextauth]`) and custom user registration.
        *   `app/api/orders/`: Provides CRUD operations for customer orders, including stock management.
        *   `app/api/products/`: Provides CRUD operations for product inventory.
    *   `app/dashboard/`: The main authenticated user interface for managing products and orders.
    *   `app/login/`: User login page.
    *   `app/register/`: User registration page.
    *   `app/layout.tsx`: The root layout for the entire application, including metadata and `SessionProvider`.
    *   `app/page.tsx`: The public-facing landing/marketing page.
*   `components/`: Reusable React components used across different pages (e.g., `Navbar`, `Footer`, `SessionProvider`).
*   `lib/`: Contains utility functions and configurations.
    *   `lib/auth.ts`: NextAuth.js configuration, defining authentication providers and callbacks.
    *   `lib/prisma.ts`: Initializes the Prisma client for database interactions.
*   `prisma/`: Holds the Prisma schema definition.
    *   `prisma/schema.prisma`: Defines the database models (User, Account, Session, VerificationToken, Product, Order, OrderItem, OrderStatus enum) and their relationships.
*   `types/`: Custom TypeScript type declarations, specifically extending NextAuth.js types.
*   `public/`: (Implicit, standard Next.js) For static assets like images, fonts, etc.
*   `globals.css`: Global styles for the application.
*   `package.json`: Project metadata, scripts, and dependency declarations.
*   `postcss.config.js`: Configuration for PostCSS plugins (Tailwind CSS, Autoprefixer).

## 4. Setup & Build Instructions

To get the Carthus project up and running, follow these steps:

1.  **Clone the repository**:
    ```bash
    git clone <repository-url>
    cd elijeweb # or whatever the project folder is named
    ```

2.  **Install Dependencies**:
    ```bash
    npm install
    # or yarn install
    ```
    This command will also automatically run `prisma generate` due to the `postinstall` script, generating the Prisma client.

3.  **Set up Environment Variables**:
    Create a `.env` file in the project root and add the following:
    ```env
    DATABASE_URL="postgresql://user:password@host:port/database?schema=public"
    NEXTAUTH_URL="http://localhost:3000" # Or your deployment URL
    NEXTAUTH_SECRET="YOUR_VERY_LONG_RANDOM_SECRET_STRING" # Generate a strong, random string
    ```
    *   `DATABASE_URL`: Connection string for your PostgreSQL database.
    *   `NEXTAUTH_URL`: The base URL of your application.
    *   `NEXTAUTH_SECRET`: A secret key used by NextAuth.js for signing tokens.

4.  **Run Database Migrations**:
    Apply the Prisma schema to your database. This will create the necessary tables.
    ```bash
    npx prisma migrate dev --name init
    ```
    You can replace `init` with a descriptive name for your migration.

5.  **Start the Development Server**:
    ```bash
    npm run dev
    # or yarn dev
    ```
    The application will be accessible at `http://localhost:3000`.

6.  **Build for Production**:
    ```bash
    npm run build
    # or yarn build
    ```
    This command compiles the application for production deployment.

7.  **Run Production Build**:
    ```bash
    npm run start
    # or yarn start
    ```
    This command serves the production-ready application.

## 5. Proposed CI/CD Pipeline

A simple CI/CD pipeline using GitHub Actions can ensure code quality and automated deployments.

```yaml
name: Carthus CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-lint:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20' # Or the recommended Node.js version for Next.js 14

      - name: Install dependencies
        run: npm ci

      - name: Generate Prisma client
        run: npx prisma generate

      - name: Run ESLint
        run: npm run lint

      - name: Build project
        run: npm run build
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          NEXTAUTH_URL: ${{ secrets.NEXTAUTH_URL }}
          NEXTAUTH_SECRET: ${{ secrets.NEXTAUTH_SECRET }}

  # Optional: Add a deployment job here, e.g., to Vercel, Netlify, or a custom server
  # deploy:
  #   needs: build-and-lint
  #   runs-on: ubuntu-latest
  #   environment: production # Or staging
  #   steps:
  #     - name: Deploy to Vercel
  #       uses: amondnet/vercel-action@v25
  #       with:
  #         vercel-token: ${{ secrets.VERCEL_TOKEN }}
  #         vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
  #         vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
  #         vercel-args: '--prod' # For production deployment
  #         github-token: ${{ secrets.GITHUB_TOKEN }} # Required for Vercel integration
```

**Note on Secrets**: Ensure `DATABASE_URL`, `NEXTAUTH_URL`, `NEXTAUTH_SECRET`, and any deployment-specific tokens (e.g., `VERCEL_TOKEN`) are configured as secrets in your GitHub repository settings.

## 6. Architecture Diagram

The application follows a typical client-server architecture with a Next.js full-stack setup.

```mermaid
graph TD
    User[User] -->|Browser/Client| NextJS_UI[Next.js UI (Pages)]
    NextJS_UI -->|HTTP/S Requests| NextJS_API[Next.js API Routes]

    subgraph Next.js Application
        NextJS_UI
        NextJS_API
    end

    NextJS_API -->|Authentication| NextAuth[NextAuth.js]
    NextJS_API -->|ORM Queries| Prisma[Prisma ORM]
    NextAuth -->|User/Session Data| Prisma

    Prisma -->|SQL Commands| PostgreSQL[PostgreSQL Database]

    style NextJS_UI fill:#b3e0ff,stroke:#3399ff,stroke-width:2px
    style NextJS_API fill:#c2f0c2,stroke:#66cc66,stroke-width:2px
    style NextAuth fill:#ffcc99,stroke:#ff9933,stroke-width:2px
    style Prisma fill:#e6ccff,stroke:#9933ff,stroke-width:2px
    style PostgreSQL fill:#ffb3b3,stroke:#ff6666,stroke-width:2px
```

**Explanation:**
*   **User**: Interacts with the web application through their browser.
*   **Next.js UI (Pages)**: The frontend part of the application, rendered by Next.js, handling user interactions and displaying data.
*   **Next.js API Routes**: The backend part of the application, also within the Next.js project, handling business logic, authentication, and data persistence.
*   **NextAuth.js**: Manages user authentication, including login, registration, and session management. It integrates with Prisma to store user and session data.
*   **Prisma ORM**: An Object-Relational Mapper that provides a type-safe way for the Next.js API routes to interact with the database.
*   **PostgreSQL Database**: The primary data store for users, products, orders, and other application data.

## 7. Potential Improvements

1.  **Comprehensive Testing Suite**: Implement unit tests for utility functions and API handlers, integration tests for API endpoints, and end-to-end tests (e.g., using Playwright or Cypress) for critical user flows (registration, login, product/order management). This will significantly improve code reliability and maintainability.
2.  **Robust Input Validation & Error Handling**: Enhance server-side input validation beyond basic checks (e.g., using a schema validation library like Zod) for all API endpoints to prevent invalid data and potential security vulnerabilities. Additionally, standardize error responses and implement more granular error logging for easier debugging and monitoring.
3.  **Pagination and Filtering for Data Lists**: For the `/api/products` and `/api/orders` endpoints, implement pagination, sorting, and filtering capabilities. As the number of products or orders grows, fetching all data at once can lead to performance issues and a poor user experience. This would involve adding query parameters to the API and adjusting Prisma queries accordingly.