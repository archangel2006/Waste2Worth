# Project Overview

This is a full-stack web application built on a modern, serverless technology stack. It's designed to be a platform where users can donate surplus food, which is then categorized by AI and made available for pickup by organizations or volunteers.

---

# Core Technology Stack
The application is built with a set of popular and robust technologies:

- **Framework**: **Next.js** (using the App Router) with **React**. This provides server-side rendering, routing, and a powerful component-based architecture.
- **Language**: TypeScript. This adds static typing to JavaScript, which helps prevent bugs and improves developer experience.
- **UI Components**: ShadCN UI. This is a collection of reusable UI components (like buttons, cards, and dialogs) that are built on top of Radix UI for accessibility.
- **Styling:** Tailwind CSS. A utility-first CSS framework that allows for rapid styling directly in the HTML. The theme and colors are configured in src/app/globals.css.
- **Fonts**: PT Sans for body text and Playfair Display for headlines, loaded from Google Fonts.

---

# AI-Powered Categorization

The core AI feature is the automatic categorization of food donations. This is handled by Genkit, an open-source framework for building AI-powered applications.

## How it Works:
1. When a donor fills out the "Create a New Donation" form and submits it, the data (including the title, food type, storage condition, and a photo) is sent to a server-side AI flow.
2. This flow is defined in src/ai/flows/categorize-donations.ts.
3. It uses a Google AI model (specifically gemini-2.5-flash, as configured in src/ai/genkit.ts) to analyze the provided text and image.
4. The AI has been prompted to act as a specialized assistant that determines if the donation is "Edible", "Usable" (for animal feed, etc.), or only suitable for "Compost".
5. The model returns a structured JSON object containing the category and a reason for its decision, which is then saved with the donation.

---

# Hosting And Deployement
The application is configured for easy and scalable cloud deployment.

1. Hosting Provider: Firebase App Hosting. The configuration for this is in the apphosting.yaml file, which sets up the backend infrastructure on Google Cloud.
2. Deployment: The project is set up for continuous deployment. When changes are pushed to your repository, Firebase App Hosting automatically builds and deploys the new version of your Next.js application.
  
# Authentication And Data
1. Authentication: The login and user system is currently a mock implementation.
- The form in src/app/login/login-form.tsx simulates a login/signup process.
- The useCurrentUser hook in src/hooks/use-current-user.ts provides mock user data, allowing you to test the app from different user perspectives (donor, volunteer, organization).
- This setup is designed to be easily replaced with a real authentication provider like Firebase Authentication without needing to change the UI significantly.

2. Database: Similar to authentication, the data is currently using mock placeholder data located in src/lib/placeholder-data.ts. This file acts as a temporary, in-memory database for donations, users, and community posts. This allows the application to be fully interactive for prototyping before connecting to a live database like Firebase Firestore.


