# DreamInsight

## Table of Contents

- [DreamInsight](#dreaminsight)
  - [Table of Contents](#table-of-contents)
  - [Project Description](#project-description)
  - [Tech Stack](#tech-stack)
  - [Getting Started Locally](#getting-started-locally)
  - [Available Scripts](#available-scripts)
  - [Project Scope](#project-scope)
    - [MVP Features](#mvp-features)
    - [Out of Scope for MVP](#out-of-scope-for-mvp)
  - [Project Status](#project-status)
  - [License](#license)

## Project Description

DreamInsight is an AI-powered web application for dream journaling. It allows users to securely record their dreams, tag emotions and symbols, generate on-demand interpretations from an AI model, and view simple trend visualizations on a dashboard. The MVP is designed to be minimalistic with a default dark theme, using Supabase for authentication and database storage. AI interpretations are generated manually and saved to the database to avoid recurring query costs.

The main goals of the project are:

- Quick and secure addition of dream entries (CRUD).
- AI-driven dream interpretations saved as part of the entry.
- A dashboard with aggregations of emotions and tags for 7/30 day periods.
- A simple user flow: registration → dream list → add dream → request interpretation → rate.

## Tech Stack

- **Framework**: Next.js (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **UI Components**: shadcn/ui
- **Backend & Database**: Supabase (Postgres + Auth + RLS)
- **AI**: OpenAI
- **E2E Testing**: Playwright

## Getting Started Locally

To get a local copy up and running, follow these simple steps.

### Prerequisites

- Node.js (version 20.x or later)
- npm, yarn, or pnpm

### Installation

1.  **Clone the repository:**

    ```sh
    git clone https://github.com/your-username/dream-insight.git
    cd dream-insight
    ```

2.  **Install dependencies:**

    ```sh
    npm install
    # or
    yarn install
    # or
    pnpm install
    ```

3.  **Set up environment variables:**
    Create a `.env.local` file in the root of your project and add the following environment variables. You can get these from your Supabase project dashboard.

    ```env
    NEXT_PUBLIC_SUPABASE_URL=your-supabase-project-url
    NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
    OPENAI_API_KEY=your-openai-api-key
    ```

4.  **Run the development server:**
    ```sh
    npm run dev
    ```
    Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

## Available Scripts

In the project directory, you can run:

- `npm run dev`: Runs the app in development mode.
- `npm run build`: Builds the app for production.
- `npm run start`: Starts a production server.
- `npm run lint`: Runs the linter to check for code quality issues.

## Project Scope

### MVP Features

- User registration and login (Supabase Auth).
- Full CRUD functionality for dream entries.
- On-demand generation and saving of AI interpretations.
- Dashboard with a 7/30 day summary of emotions and tags.
- Form validation and a dark theme UI.
- CI/CD pipeline with at least one E2E test.

### Out of Scope for MVP

- Reminders and a soft cap of 30 entries.
- Advanced visualizations on the dashboard.
- Advanced ML models or custom training.
- Integrations with wearables or automatic dream imports.
- Social features like sharing or discussions.
- Data export (PDF, calendar), native mobile apps, or voice input.
- Versioning of entries (edits overwrite the previous version in the MVP).

## Project Status

This project is currently in the **development phase**. The core features for the MVP are being built.

## License

This project is licensed under the MIT License. See the `LICENSE` file for more details.
