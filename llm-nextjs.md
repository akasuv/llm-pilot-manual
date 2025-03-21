# LLM Pilot Manual - Next.js

## PROJECT STRUCTURE PRINCIPLES
The goal of this structure is to provide a clear, scalable organization that leverages Next.js App Router features while maintaining good separation of concerns.

## DIRECTORY STRUCTURE

```
project
├── app/                     # Next.js App Router root
│   ├── api/                 # API routes
│   ├── components/          # Components specific to root layout
│   ├── page.tsx             # Root page component
│   ├── layout.tsx           # Root layout component
│   ├── [page]/              # Route groups (e.g., tools/, auth/)
│   │   ├── components/      # Page-specific components
│   │   ├── actions.ts       # Server actions for this route
│   │   ├── page.tsx         # Page component
│   │   └── layout.tsx       # Optional layout for this route
├── components/              # Shared components
│   ├── ui/                  # UI primitives and design system
│   └── shared/              # Reusable feature components
├── lib/                     # Utility libraries
│   ├── hooks/               # React hooks
│   └── utils/               # Utility functions
├── middleware/              # Modular middleware components
│   ├── auth.ts              # Authentication middleware
│   ├── logging.ts           # Logging middleware
│   ├── metrics.ts           # Metrics collection middleware
│   └── index.ts             # Re-exports middleware components
├── middleware.ts            # Main Next.js middleware entry point
├── services/                # External service integrations
│   └── supabase/            # Database and API integrations
├── types/                   # TypeScript type definitions
└── public/                  # Static assets
```

## PATH CONFIGURATION

Configure absolute imports in `tsconfig.json` to use the '@' alias for cleaner imports:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

This allows you to import from the project root using the '@' symbol:

```typescript
// Import from app directory
import { SomeComponent } from '@/app/components/some-component';

// Import from components directory
import { Button } from '@/components/ui/button';

// Import from lib directory
import { formatDate } from '@/lib/utils/date-formatter';

// Import from services directory
import { supabaseClient } from '@/services/supabase/client';
```

## MIDDLEWARE ORGANIZATION

Next.js supports a special `middleware.ts` file at the root of your project that runs before requests are processed. While only one middleware.ts file is supported per project, you can still organize your middleware logic modularly:

```typescript
// middleware/auth.ts
export function authMiddleware(req) {
  // Authentication logic
}

// middleware/logging.ts
export function loggingMiddleware(req) {
  // Logging logic
}

// middleware/index.ts
export { authMiddleware } from './auth';
export { loggingMiddleware } from './logging';
export { metricsMiddleware } from './metrics';

// Root middleware.ts
import { NextResponse } from 'next/server';
import { authMiddleware, loggingMiddleware, metricsMiddleware } from './middleware';

export function middleware(request) {
  // Call modular middleware functions in order
  authMiddleware(request);
  loggingMiddleware(request);
  metricsMiddleware(request);
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/api/:path*', '/dashboard/:path*'],
};
```

Note: While only one middleware.ts file is supported per project, you can still organize your middleware logic modularly. Break out middleware functionalities into separate .ts or .js files and import them into your main middleware.ts file. This allows for cleaner management of route-specific middleware, aggregated in the middleware.ts for centralized control. By enforcing a single middleware file, it simplifies configuration, prevents potential conflicts, and optimizes performance by avoiding multiple middleware layers.

## NAMING CONVENTIONS

1. **All Directories** (including React component directories): Use kebab-case for directories (e.g., `user-settings/`).

2. **Files**:
   - All Files (including React component Files): Use kebab-case for all file names (e.g., `button.tsx`, `user-profile.tsx`, `date-utils.ts`).
   - Next.js Special Files: Use the standard Next.js naming (e.g., `page.tsx`, `layout.tsx`, `loading.tsx`).

## COMPONENT ORGANIZATION

1. **Component Placement**:
   - Page-specific components: Place in `/app/[page]/components/` if only used within that page.
   - Layout components: Place in `/app/components/` if only used in the root layout.
   - Shared components: Place in `/components/shared/` if used across multiple pages.
   - UI primitives: Place in `/components/ui/` for base UI components.

2. **Component Directory Structure**: For complex components, use a directory structure:

```
components/shared/data-table/
├── data-table.tsx         # Main component
├── data-table-header.tsx  # Sub-component
├── data-table-row.tsx     # Sub-component
└── index.ts               # Re-export the main component
```

## DATA HANDLING

1. **Server Actions**: Place page-specific server actions in `app/[page]/actions.ts`.
2. **API Routes**: Place in `app/api/[route]/route.ts`.
3. **Database Logic**: Place in `services/supabase/` or appropriate data integration directory.

## BEST PRACTICES

1. Use the App Router features like server components, loading states, and error boundaries.
2. Minimize client components - use 'use client' only when necessary.
3. Keep components focused and composable.
4. Prioritize co-location of files that change together.
5. For very large applications, consider route groups to further organize the codebase.
6. **Always use absolute imports with the '@' alias** (configured in tsconfig.json):
   ```typescript
   // ✅ Good - Using absolute imports
   import { Button } from '@/components/ui/button';
   import { getUserById } from '@/lib/utils/users';
   import { authService } from '@/services/supabase/auth';
   
   // ❌ Bad - Using relative imports
   import { Button } from '../../../components/ui/button';
   import { getUserById } from '../../lib/utils/users';
   ```

## MIGRATION NOTES

When refactoring existing code to match this structure:
1. Start with moving components to their appropriate locations.
2. Update imports using search and replace.
3. Test each page after refactoring to ensure functionality is preserved. 