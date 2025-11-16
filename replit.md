# VoucherMax - Voucher Reselling Platform

## Overview

VoucherMax is a full-stack web application for managing and reselling vouchers. It enables administrators to manage voucher inventory, define package tiers, and track sales, while agents can sell vouchers to customers. The platform supports role-based authentication, real-time inventory updates, and comprehensive sales tracking, aiming to streamline voucher distribution and enhance operational efficiency for businesses involved in voucher sales. Key capabilities include hierarchical user management, a three-tier voucher package system, multi-currency support, and integrated sales tracking with SMS notifications.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

The application employs a modern full-stack architecture separating frontend and backend concerns.

- **Frontend**: React SPA with TypeScript, utilizing Vite for development and building. UI is built with Tailwind CSS and shadcn/ui components. State management is handled by Zustand for authentication and React Query for server state.
- **Backend**: Express.js REST API with TypeScript.
- **Database**: PostgreSQL with Drizzle ORM. Currently uses in-memory storage with plans for full database migration.
- **Authentication**: A three-tier system with Super Admin, Admin, and Agent roles, each having dedicated login interfaces and role-based dashboards. Super Admins manage admin accounts and system-wide settings, Admins handle voucher and system operations, and Agents focus on sales and customer management. Features include secure individual Super Admin accounts, admin credential generation by Super Admins, and persistent login state.
- **Voucher Management**: Supports a three-tier package system (Silver, Gold, Platinum) with multi-currency support (ISO 4217). Admins can create, read, update, delete vouchers, and perform bulk CSV uploads. Vouchers are linked to packages for pricing and categorization. Includes dedicated online voucher creation with Mikrotik router integration for WiFi access.
- **Sales System**: Agents sell vouchers by selecting codes, entering customer details, and specifying amounts. Sales are tracked with timestamps and agent attribution, with real-time inventory updates. Integrated SMS disbursement allows for client notifications with configurable providers, including international and East African services.
- **UI/UX Decisions**:
    - **Login System**: Main selection page with distinct, color-themed interfaces for Super Admin (purple), Admin (gray), and Agent (green) roles for clear visual distinction and navigation.
    - **Dashboards**: Role-specific dashboards provide tailored functionality and information.
    - **User Interfaces**: Professional UI for package management, multi-currency selection, SMS settings, and online voucher management.
    - **Online Vouchers**: Admin interface for creating online vouchers with Mikrotik integration; Agent interface for managing their activated online vouchers.
    - **Design**: Responsive design with Tailwind CSS and consistent use of shadcn/ui for accessibility. WiFi branding icons are used throughout the application.

## External Dependencies

- **Frontend**:
    - React 18, React DOM, Wouter (React Router)
    - @tanstack/react-query, Zustand
    - @radix-ui components, Lucide-react icons
    - React-hook-form with @hookform/resolvers
    - Tailwind CSS, Class-variance-authority, Clsx
    - Date-fns
- **Backend**:
    - Express
    - @neondatabase/serverless (Neon PostgreSQL)
    - Drizzle-orm, Drizzle-zod
    - Connect-pg-simple
    - Nanoid
- **Development Tools**:
    - Vite, Esbuild
    - TypeScript
    - Tsx
- **SMS Providers (Configurable)**:
    - International: Twilio, Nexmo, AWS SNS, MessageBird
    - East Africa: Africa's Talking, Safaricom, Airtel Africa, MTN, Tigo, Vodacom, Orange Africa, Hubtel