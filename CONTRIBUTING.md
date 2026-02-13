# Contributing to Farmly AI

Thank you for your interest in contributing to Farmly AI! This document provides guidelines and instructions for contributing.

---

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Development Workflow](#development-workflow)
4. [Code Style](#code-style)
5. [Testing](#testing)
6. [Pull Request Process](#pull-request-process)
7. [Project Structure](#project-structure)
8. [Common Tasks](#common-tasks)

---

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. Please:

- Be respectful and constructive in all interactions
- Focus on what is best for the farming community
- Show empathy towards other contributors
- Accept constructive criticism gracefully

---

## Getting Started

### Prerequisites

- **Node.js** 20+ and **pnpm** 8+
- **Python** 3.11+
- **Git**
- MongoDB Atlas account
- Redis Cloud account

### Setup Development Environment

1. **Fork the repository** on GitHub

2. **Clone your fork**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/farmly-ai.git
   cd farmly-ai
   ```

3. **Add upstream remote**:
   ```bash
   git remote add upstream https://github.com/ORIGINAL_OWNER/farmly-ai.git
   ```

4. **Install dependencies**:
   ```bash
   pnpm install
   ```

5. **Setup environment variables**:
   ```bash
   # Backend
   cp backend/.env.example backend/.env
   # Edit backend/.env with your credentials

   # Frontend
   cp frontend/.env.example frontend/.env
   # Edit frontend/.env

   # ML Service
   cp ml-service/.env.example ml-service/.env
   # Edit ml-service/.env
   ```

6. **Initialize database**:
   ```bash
   cd backend
   pnpm run init:db
   pnpm run seed:schemes
   pnpm run seed:demo
   ```

7. **Run locally**:
   ```bash
   # Terminal 1 - Backend
   cd backend && pnpm run dev

   # Terminal 2 - Frontend
   cd frontend && pnpm run dev

   # Terminal 3 - ML Service
   cd ml-service && uvicorn app.main:app --reload
   ```

---

## Development Workflow

### 1. Create a Branch

Always create a new branch for your work:

```bash
git checkout -b feature/your-feature-name
# Or for bug fixes:
git checkout -b fix/bug-description
```

Branch naming conventions:
- `feature/` - New features
- `fix/` - Bug fixes
- `docs/` - Documentation changes
- `refactor/` - Code refactoring
- `test/` - Adding tests
- `chore/` - Maintenance tasks

### 2. Make Your Changes

- Follow the [code style guidelines](#code-style)
- Write clear, self-documenting code
- Add comments for complex logic
- Update documentation if needed

### 3. Test Your Changes

```bash
# Frontend
cd frontend
pnpm run typecheck
pnpm run lint
pnpm run build

# Backend
cd backend
pnpm run typecheck
pnpm run lint
pnpm run build

# Verify functionality manually
# Test all affected features
```

### 4. Commit Your Changes

Use [Conventional Commits](https://www.conventionalcommits.org/) format:

```bash
git add .
git commit -m "feat: add disease history filtering"
git commit -m "fix: resolve OTP expiration issue"
git commit -m "docs: update API documentation"
```

Commit message format:
```
<type>(<scope>): <subject>

<body>

<footer>
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Maintenance tasks

### 5. Keep Your Branch Updated

```bash
git fetch upstream
git rebase upstream/main
```

### 6. Push to Your Fork

```bash
git push origin feature/your-feature-name
```

---

## Code Style

### TypeScript/JavaScript

- Use **TypeScript** for all new code
- Follow **ESLint** rules (configured in `.eslintrc.cjs`)
- Use **Prettier** for formatting
- Prefer **functional components** and **hooks** in React
- Use **async/await** over promises

**Example**:
```typescript
// ✅ Good
export const fetchUserProfile = async (userId: string): Promise<User> => {
  try {
    const response = await axios.get(`/api/user/${userId}`);
    return response.data;
  } catch (error) {
    logger.error('Failed to fetch user profile', { userId, error });
    throw error;
  }
};

// ❌ Avoid
export function fetchUserProfile(userId) {
  return axios.get('/api/user/' + userId).then(res => res.data);
}
```

### Python

- Follow **PEP 8** style guide
- Use **type hints**
- Document functions with docstrings

**Example**:
```python
# ✅ Good
def preprocess_image(image: np.ndarray, target_size: tuple[int, int]) -> np.ndarray:
    """
    Preprocess image for disease detection model.
    
    Args:
        image: Input image as numpy array
        target_size: Target dimensions (height, width)
    
    Returns:
        Preprocessed image ready for inference
    """
    image = cv2.resize(image, target_size)
    image = image / 255.0
    return image
```

### File Organization

```typescript
// Component file structure
import React from 'react';           // External imports
import { useNavigate } from 'react-router-dom';

import { Button } from '@/components/ui/button';  // Internal imports
import { useAuth } from '@/hooks/useAuth';
import { User } from '@/types';      // Types

import './styles.css';               // Styles

// Types and interfaces
interface ComponentProps {
  // ...
}

// Component
export const Component: React.FC<ComponentProps> = ({ ... }) => {
  // ...
};
```

---

## Testing

### Manual Testing Checklist

Before submitting a PR, test:

- [ ] Feature works as expected
- [ ] No console errors (browser or terminal)
- [ ] Responsive design (mobile, tablet, desktop)
- [ ] Works in multiple browsers (Chrome, Safari, Firefox)
- [ ] Works in selected language (at least Hindi + English)
- [ ] No TypeScript errors
- [ ] No ESLint warnings

### Running Tests

```bash
# TypeScript type checking
pnpm run typecheck

# Linting
pnpm run lint

# Fix lint issues automatically
pnpm run lint --fix

# Format code
pnpm run format

# Build (catches compilation errors)
pnpm run build
```

### Testing Specific Features

**Authentication**:
```bash
# Enable demo mode
# In backend/.env: DEMO_MODE=true
# Use demo phone numbers with OTP: 123456
```

**Disease Detection**:
```bash
# Test with sample images from:
# Crop disese classification/Tomato Disease Clssifier/test/
```

**Voice Interface**:
```bash
# Test microphone permissions
# Record audio and check transcription accuracy
# Test in Hindi, Tamil, and English
```

---

## Pull Request Process

### Before Submitting

1. Ensure your code follows style guidelines
2. Run all tests and ensure they pass
3. Update documentation if needed
4. Rebase on latest `main` branch
5. Write clear commit messages

### Submitting a PR

1. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```

2. **Create Pull Request** on GitHub with:
   - **Clear title**: Summarize the change in <50 characters
   - **Description**:
     ```markdown
     ## What does this PR do?
     Brief description of changes
     
     ## Why is this needed?
     Problem being solved or feature being added
     
     ## How was this tested?
     Steps to test the changes
     
     ## Screenshots (if applicable)
     Before/after screenshots for UI changes
     
     ## Checklist
     - [ ] Code follows style guidelines
     - [ ] Tests pass
     - [ ] Documentation updated
     - [ ] No breaking changes (or documented)
     ```

3. **Link related issues**:
   - Use keywords: "Closes #123" or "Fixes #456"

### Review Process

1. Maintainers will review your PR
2. Address feedback by pushing new commits
3. Once approved, maintainers will merge

### After Merge

1. **Delete your branch**:
   ```bash
   git branch -d feature/your-feature-name
   git push origin --delete feature/your-feature-name
   ```

2. **Sync your fork**:
   ```bash
   git checkout main
   git pull upstream main
   git push origin main
   ```

---

## Project Structure

```
farmly-ai/
├── frontend/          # React PWA
│   ├── src/
│   │   ├── components/   # Reusable UI components
│   │   ├── pages/        # Page components (routes)
│   │   ├── services/     # API clients
│   │   ├── store/        # State management (Zustand)
│   │   ├── hooks/        # Custom React hooks
│   │   ├── locales/      # i18n translations
│   │   └── types/        # TypeScript types
│
├── backend/           # Express.js API
│   ├── src/
│   │   ├── controllers/  # Route handlers
│   │   ├── middleware/   # Auth, error handling, etc.
│   │   ├── models/       # MongoDB schemas
│   │   ├── routes/       # API routes
│   │   ├── services/     # Business logic
│   │   ├── utils/        # Helper functions
│   │   └── scripts/      # Seed scripts
│
├── ml-service/        # FastAPI ML service
│   ├── app/
│   │   ├── services/     # ML inference logic
│   │   └── main.py       # FastAPI app
│   └── models/           # Pre-trained models
│
└── docs/              # Documentation
```

---

## Common Tasks

### Adding a New Translation

1. **Add translation key** to all language files:
   ```bash
   frontend/src/locales/en.json
   frontend/src/locales/hi.json
   frontend/src/locales/ta.json
   frontend/src/locales/ml.json
   frontend/src/locales/te.json
   frontend/src/locales/kn.json
   ```

2. **Use in component**:
   ```typescript
   import { useTranslation } from 'react-i18next';
   
   const { t } = useTranslation();
   return <h1>{t('your.translation.key')}</h1>;
   ```

3. **Verify completeness**:
   ```bash
   cd frontend
   pnpm run i18n:check
   ```

### Adding a New API Endpoint

1. **Define route** in `backend/src/routes/`:
   ```typescript
   // backend/src/routes/yourFeature.routes.ts
   import { Router } from 'express';
   import { yourController } from '../controllers/yourFeature.controller.js';
   import { authenticate } from '../middleware/auth.middleware.js';
   
   const router = Router();
   router.get('/endpoint', authenticate, yourController);
   
   export default router;
   ```

2. **Create controller**:
   ```typescript
   // backend/src/controllers/yourFeature.controller.ts
   import { Request, Response } from 'express';
   
   export const yourController = async (req: Request, res: Response) => {
     try {
       // Your logic
       res.json({ success: true, data: {} });
     } catch (error) {
       res.status(500).json({ success: false, error: 'Error message' });
     }
   };
   ```

3. **Register route** in `backend/src/index.ts`:
   ```typescript
   import yourFeatureRoutes from './routes/yourFeature.routes.js';
   app.use('/api/yourFeature', yourFeatureRoutes);
   ```

4. **Update API documentation** in `docs/API.md`

### Adding a New MongoDB Model

1. **Create schema** in `backend/src/models/`:
   ```typescript
   import mongoose from 'mongoose';
   
   const yourSchema = new mongoose.Schema({
     field1: { type: String, required: true },
     field2: { type: Number, default: 0 },
     createdAt: { type: Date, default: Date.now }
   });
   
   export const YourModel = mongoose.model('YourModel', yourSchema);
   ```

2. **Add to database initialization** in `backend/scripts/init-db.ts`

### Updating Dependencies

```bash
# Check outdated packages
pnpm outdated

# Update all dependencies
pnpm update

# Update specific package
pnpm update package-name

# Test after updating
pnpm run typecheck
pnpm run lint
pnpm run build
```

---

## Questions?

- Check [README.md](./README.md) for project overview
- See [docs/QUICK_START.md](./docs/QUICK_START.md) for setup
- See [docs/TROUBLESHOOTING.md](./docs/TROUBLESHOOTING.md) for common issues
- Open an issue on GitHub for questions

---

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for contributing to Farmly AI and helping farmers!** 🌾
