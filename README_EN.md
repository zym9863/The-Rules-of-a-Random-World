English | [中文](./README.md)

# The Rules of a Random World 🎲

An interactive probability theory teaching application built with Vue 3 + TypeScript + Vite, designed to help understand core probability theorems through visual demonstrations.

## ✨ Features

- 📊 **Law of Large Numbers Demo** - Dynamic charts showing the convergence process of the law of large numbers
- 📈 **Central Limit Theorem Visualization** - Real-time observation of the normalization process of sample mean distribution
- 🎨 **Modern UI Design** - Contemporary interface with glassmorphism effects and gradient backgrounds
- 📱 **Responsive Design** - Perfect adaptation for desktop and mobile devices
- ⚡ **High Performance Rendering** - Efficient component architecture based on Vue 3 Composition API

## 🚀 Tech Stack

- **Frontend Framework**: Vue 3 + TypeScript
- **Build Tool**: Vite
- **Styling System**: CSS3 + Design System Variables
- **Development Experience**: HMR Hot Reload + TypeScript Type Checking

## 📁 Project Structure

```
src/
├── App.vue                      # Main application component with navigation and layout
├── main.ts                      # Application entry point
├── style.css                    # Global styles and design system variables
├── vite-env.d.ts               # Vite type declarations
├── assets/                      # Static assets
└── components/                  # Components directory
    ├── LawOfLargeNumbers.vue    # Law of Large Numbers demo component
    └── CentralLimitTheorem.vue  # Central Limit Theorem demo component
```

## 🛠️ Local Development

### Requirements

- Node.js >= 16
- npm or yarn or pnpm

### Install Dependencies

```bash
npm install
```

### Start Development Server

```bash
npm run dev
```

Visit `http://localhost:5173` to view the application

### Build for Production

```bash
npm run build
```

### Preview Production Build

```bash
npm run preview
```

## 🎯 Usage Guide

1. **Select Theorem Type**: Use the top navigation bar to switch between different probability theorem demonstrations
2. **Law of Large Numbers**: Observe how the sample mean converges to the theoretical expected value as the number of experiments increases
3. **Central Limit Theorem**: Watch how the sample mean distribution gradually approaches a normal distribution

## 🎨 Design Features

- **Gradient Backgrounds**: Multi-layer gradients creating depth perception
- **Glassmorphism Effects**: Semi-transparent backgrounds with blur filters
- **Micro-interactions**: Smooth transition animations and hover effects
- **Modern Color System**: Unified color scheme based on design system
- **Responsive Layout**: Flexible layout adapting to various screen sizes

## 📚 Educational Value

This application aims to help students understand through visualization:

- **Law of Large Numbers**: The stability patterns of frequency in random experiments
- **Central Limit Theorem**: The normality convergence characteristics of sample mean distribution
- **Probability Convergence**: The relationship between theoretical and observed values
- **Statistical Inference**: Mathematical foundations for inferring populations from samples

## 🔧 Development Configuration

The project uses modern development configurations:

- **Vue 3 Composition API**: Better logic reuse and type inference
- **TypeScript**: Strict type checking ensuring code quality
- **Vite**: Fast hot reload and build experience
- **CSS Design System**: Unified variable management and responsive design

## 📄 License

MIT License

## 🤝 Contributing

Issues and Pull Requests are welcome to improve this project!

---

*Explore the wonderful world of probability theory and understand the deterministic laws behind randomness* ✨
