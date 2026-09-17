# Minimal Tasks

A modern, responsive to-do list built with Vue 3, Vite, and Tailwind CSS.

## Features

- Add, edit, complete, and delete tasks
- Filter tasks by All, Active, and Completed
- Category chips for Work, Personal, and Urgent tasks
- Progress bar with completed task count
- Dark mode toggle with saved preference
- Task persistence with browser `localStorage`
- Responsive layout with subtle transitions and empty states

## Tech Stack

- Vue 3
- Vite
- Tailwind CSS v4

## Getting Started

Install dependencies:

```bash
npm install
```

Start the development server:

```bash
npm run dev
```

Create a production build:

```bash
npm run build
```

Preview the production build:

```bash
npm run preview
```

## Project Structure

```text
src/
  App.vue
  main.js
  style.css
  components/
    HelloWorld.vue
```

The main application lives in `src/components/HelloWorld.vue`. Global Tailwind setup and dark-mode variant support live in `src/style.css`.
