# Plataforma Visible - Arquitectura Modular Clean Architecture

## Descripción General

La Plataforma Visible es un sistema de gestión de contenido público e institucional del Ministerio de la Mujer y Poblaciones Vulnerables (MIMP) que cuenta con:

### **Portal Público**
- Hero Section con buscador global
- 6 módulos temáticos: Niños/Niñas/Adolescentes, Violencia contra la Mujer, Discapacidad, Familia, Acoso Político, Adulto Mayor
- Storytelling narrativo por módulo
- Sistema de navegación y búsqueda inteligente

### **Backoffice Administrativo**
- **3 roles diferenciados:**
  - **Administrador**: Acceso completo, gestión de usuarios
  - **Editor General**: Moderación de todos los módulos
  - **Editor Observatorio**: Gestión específica por módulo
- **6 submódulos de contenido:**
  - Estadísticas
  - Buenas Prácticas
  - Servicios Institucionales
  - Opiniones de Expertos
  - Publicaciones
  - Derechos

## Stack Tecnológico

- **Framework**: Next.js 14+ (App Router)
- **Lenguaje**: TypeScript
- **Estilos**: Tailwind CSS
- **Estado**: Zustand
- **Data Fetching**: SWR/TanStack Query
- **Validación**: Zod
- **UI Components**: Radix UI / shadcn/ui

## Arquitectura Clean por Módulos

```
/src
├── /shared              # Componentes y utilidades compartidas
├── /core               # Lógica de negocio central
├── /modules            # Módulos independientes (Modulith)
└── /app                # Next.js App Router pages
```

---

## PROMPTS DE IMPLEMENTACIÓN MODULAR

### **PROMPT 1: Configuración Base y Arquitectura Core**

**Objetivo**: Establecer la base técnica del proyecto con configuración optimizada.

```markdown
## Tarea: Configuración Base Next.js + TypeScript

### 1. Inicialización del Proyecto
```bash
npx create-next-app@latest sistema-visible --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
```

### 2. Dependencias Principales
```json
{
  "dependencies": {
    "@radix-ui/react-*": "^1.0.0",
    "zustand": "^4.4.0",
    "swr": "^2.2.0",
    "zod": "^3.22.0",
    "clsx": "^2.0.0",
    "tailwind-merge": "^2.0.0"
  },
  "devDependencies": {
    "@types/node": "^20",
    "prettier": "^3.0.0",
    "prettier-plugin-tailwindcss": "^0.5.0"
  }
}
```

### 3. Configuraciones Base

#### next.config.js
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    typedRoutes: true,
  },
  images: {
    domains: ['localhost', 'api.visible.gob.pe'],
  },
}

module.exports = nextConfig
```

#### tailwind.config.js
```javascript
// Tema institucional MIMP con colores corporativos
module.exports = {
  darkMode: ["class"],
  content: ["./src/**/*.{js,ts,jsx,tsx,mdx}"],
  theme: {
    extend: {
      colors: {
        mimp: {
          primary: '#8B5CF6',    // Morado institucional
          secondary: '#EC4899',   // Rosa MIMP
          accent: '#10B981',      // Verde estados
          neutral: '#64748B',     // Grises
        }
      }
    }
  }
}
```

#### tsconfig.json (paths optimizados)
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/shared/*": ["./src/shared/*"],
      "@/core/*": ["./src/core/*"],
      "@/modules/*": ["./src/modules/*"]
    }
  }
}
```
```

---

### **PROMPT 2: Sistema de Autenticación y Roles**

**Objetivo**: Implementar autenticación robusta con sistema de roles granular.

```markdown
## Tarea: Sistema de Autenticación Multi-Rol

### 1. Estructura de Archivos
```
/src/core/auth
├── providers/
│   ├── AuthProvider.tsx
│   └── AuthContext.ts
├── hooks/
│   ├── useAuth.ts
│   ├── useRole.ts
│   └── usePermissions.ts
├── guards/
│   ├── ProtectedRoute.tsx
│   └── RoleGuard.tsx
├── types/
│   └── auth.types.ts
├── services/
│   └── auth.service.ts
└── utils/
    └── permissions.utils.ts
```

### 2. Interfaces de Usuario y Permisos
```typescript
// auth.types.ts
export interface User {
  id: string
  email: string
  name: string
  role: UserRole
  modulePermissions: ModulePermission[]
  lastLogin: Date
}

export enum UserRole {
  ADMIN = 'administrator',
  EDITOR_GENERAL = 'editor_general', 
  EDITOR_OBSERVATORIO = 'editor_observatorio'
}

export interface ModulePermission {
  moduleId: ModuleType
  permissions: Permission[]
}

export enum ModuleType {
  NNA = 'nna',
  VIOLENCIA = 'violencia',
  DISCAPACIDAD = 'discapacidad',
  FAMILIA = 'familia',
  ACOSO_POLITICO = 'acoso_politico',
  ADULTO_MAYOR = 'adulto_mayor'
}

export enum Permission {
  READ = 'read',
  WRITE = 'write',
  MODERATE = 'moderate',
  PUBLISH = 'publish',
  DELETE = 'delete'
}
```

### 3. AuthProvider con Zustand
```typescript
// providers/AuthProvider.tsx
interface AuthState {
  user: User | null
  token: string | null
  isAuthenticated: boolean
  login: (email: string, password: string) => Promise<void>
  logout: () => void
  checkPermission: (module: ModuleType, permission: Permission) => boolean
}
```

### 4. Componentes de Autenticación
- **LoginForm**: Validación con Zod, estados de carga
- **DashboardLayout**: Navegación diferenciada por rol
- **PermissionGate**: Mostrar/ocultar según permisos
```

---

### **PROMPT 3: Módulo Portal Público**

**Objetivo**: Crear la experiencia pública de navegación y búsqueda.

```markdown
## Tarea: Portal Público Interactivo

### 1. Estructura del Módulo
```
/src/modules/public
├── domain/
│   ├── entities/
│   │   ├── Module.ts
│   │   └── SearchResult.ts
│   └── repositories/
│       └── PublicRepository.interface.ts
├── infrastructure/
│   ├── api/
│   │   └── public.api.ts
│   └── adapters/
│       └── public.adapter.ts
├── application/
│   ├── usecases/
│   │   ├── GetVisibleModules.ts
│   │   └── SearchContent.ts
│   └── services/
│       └── SearchService.ts
└── presentation/
    ├── components/
    │   ├── HeroSection.tsx
    │   ├── ModuleGrid.tsx
    │   ├── SearchBar.tsx
    │   ├── ModuleCard.tsx
    │   └── StorytellingViewer.tsx
    ├── pages/
    │   ├── HomePage.tsx
    │   └── ModulePage.tsx
    └── hooks/
        ├── usePublicModules.ts
        └── useGlobalSearch.ts
```

### 2. Componentes Clave

#### HeroSection.tsx
```typescript
interface HeroSectionProps {
  onSearch: (query: string) => void
  featuredModules: Module[]
}

// Funcionalidades:
// - Buscador global con autocompletado
// - Destacados dinámicos
// - Call-to-action institucional
```

#### ModuleCard.tsx
```typescript
interface ModuleCardProps {
  module: Module
  isVisible: boolean
  onClick: () => void
}

// Estados visuales:
// - Verde: Módulo visible al público
// - Rojo: Módulo oculto (solo backoffice)
// - Contador de contenido disponible
```

#### StorytellingViewer.tsx
```typescript
interface StorytellingViewerProps {
  moduleId: ModuleType
  canSkip: boolean
  onComplete: () => void
}

// Características:
// - Barra de progreso visual
// - Navegación anterior/siguiente
// - Opción "Saltar" storytelling
// - Contenido multimedia (imágenes/videos)
```

### 3. Navegación y Búsqueda
- **Búsqueda inteligente** con sugerencias contextuales
- **Filtros por módulo** y tipo de contenido
- **Resultados enlazados** a secciones específicas
- **Historial de búsquedas** para analytics
```

---

### **PROMPT 4: Módulo Backoffice - Configuración**

**Objetivo**: Interface administrativa para gestión de visibilidad y configuración.

```markdown
## Tarea: Sistema de Configuración Administrativa

### 1. Estructura del Módulo
```
/src/modules/backoffice/configuration
├── domain/
│   ├── entities/
│   │   ├── ModuleConfig.ts
│   │   └── VisibilitySettings.ts
│   └── repositories/
│       └── ConfigRepository.interface.ts
├── infrastructure/
│   └── api/
│       └── config.api.ts
├── application/
│   ├── usecases/
│   │   ├── UpdateModuleVisibility.ts
│   │   └── GetModuleSettings.ts
│   └── services/
│       └── ConfigService.ts
└── presentation/
    ├── components/
    │   ├── ModuleToggleTree.tsx
    │   ├── SubmoduleManager.tsx
    │   ├── VisibilityMatrix.tsx
    │   └── PreviewPanel.tsx
    ├── pages/
    │   └── ConfigurationPage.tsx
    └── hooks/
        ├── useModuleConfig.ts
        └── useVisibilitySettings.ts
```

### 2. Componentes de Configuración

#### ModuleToggleTree.tsx
```typescript
interface ModuleNode {
  id: string
  name: string
  isVisible: boolean
  submodules: SubmoduleNode[]
  canToggle: boolean // Según rol del usuario
}

// Funcionalidades:
// - Vista en árbol expandible/colapsable
// - Switches On/Off por módulo y submódulo
// - Estados de carga durante guardado
// - Validación de dependencias (ej: no ocultar padre con hijos visibles)
```

#### VisibilityMatrix.tsx
```typescript
// Tabla matricial: Módulos × Submódulos × Visibilidad
// Ideal para usuarios Administrador
// Vista compacta con acciones en lote
```

#### PreviewPanel.tsx
```typescript
// Preview en tiempo real de cómo se ve el portal público
// Iframe o vista simulada
// Botón "Ver cambios en vivo"
```

### 3. Validaciones y Reglas de Negocio
- **Dependencias entre módulos**: Estadísticas requiere al menos un módulo visible
- **Permisos granulares**: Editor Observatorio solo su módulo asignado
- **Guardado automático** con debounce
- **Historial de cambios** para auditoría
```

---

### **PROMPT 5: Módulos de Contenido (CRUD)**

**Objetivo**: Sistema CRUD para gestión de contenido por submódulos.

```markdown
## Tarea: Sistema de Gestión de Contenido Multi-Submódulo

### 1. Arquitectura por Submódulo
```
/src/modules/content
├── shared/
│   ├── components/
│   │   ├── ContentEditor.tsx      # WYSIWYG universal
│   │   ├── ContentTable.tsx       # Tabla reutilizable
│   │   ├── ModerationPanel.tsx    # Interface de moderación
│   │   ├── ContentViewer.tsx      # Vista pública
│   │   └── FileUploader.tsx       # Carga de archivos
│   ├── hooks/
│   │   ├── useContent.ts
│   │   └── useModeration.ts
│   └── types/
│       └── content.types.ts
├── statistics/
│   ├── domain/entities/Statistic.ts
│   ├── presentation/pages/StatisticsPage.tsx
│   └── infrastructure/api/statistics.api.ts
├── good-practices/
│   ├── domain/entities/GoodPractice.ts
│   ├── presentation/pages/GoodPracticesPage.tsx
│   └── infrastructure/api/good-practices.api.ts
├── services/
├── expert-opinions/
├── publications/
└── rights/
```

### 2. Interfaces Comunes de Contenido
```typescript
// content.types.ts
interface BaseContent {
  id: string
  title: string
  description: string
  content: string // HTML/Markdown
  moduleId: ModuleType
  status: ContentStatus
  author: User
  createdAt: Date
  updatedAt: Date
  publishedAt?: Date
  attachments: FileAttachment[]
  tags: string[]
  metadata: Record<string, any>
}

enum ContentStatus {
  DRAFT = 'draft',
  PENDING_REVIEW = 'pending_review',
  APPROVED = 'approved',
  REJECTED = 'rejected',
  PUBLISHED = 'published',
  ARCHIVED = 'archived'
}

interface FileAttachment {
  id: string
  filename: string
  url: string
  mimeType: string
  size: number
}
```

### 3. Componentes de Gestión de Contenido

#### ContentEditor.tsx
```typescript
interface ContentEditorProps {
  content?: BaseContent
  moduleId: ModuleType
  onSave: (content: BaseContent) => Promise<void>
  onCancel: () => void
}

// Características:
// - Editor WYSIWYG (TinyMCE/Tiptap)
// - Vista previa en tiempo real
// - Autoguardado como borrador
// - Validación de campos requeridos
// - Carga de imágenes/documentos
// - SEO metadata
```

#### ContentTable.tsx
```typescript
interface ContentTableProps {
  moduleId?: ModuleType
  status?: ContentStatus
  canModerate: boolean
  onEdit: (content: BaseContent) => void
  onModerate: (id: string, action: 'approve' | 'reject') => void
}

// Funcionalidades:
// - Filtros por estado, autor, fecha, módulo
// - Ordenación por columnas
// - Paginación eficiente
// - Selección múltiple para acciones en lote
// - Export CSV/PDF
```

#### ModerationPanel.tsx
```typescript
// Panel lateral para revisión de contenido
// - Vista del contenido original
// - Comentarios de revisión
// - Historial de cambios
// - Acciones: Aprobar, Rechazar, Solicitar cambios
```

### 4. Flujo de Moderación
1. **Editor** crea/edita contenido → estado DRAFT
2. **Editor** envía a revisión → estado PENDING_REVIEW  
3. **Moderador** revisa → estado APPROVED/REJECTED
4. **Contenido aprobado** → estado PUBLISHED (visible públicamente)
```

---

### **PROMPT 6: Sistema de Moderación y Analytics**

**Objetivo**: Dashboard de moderación centralizada y analytics de uso.

```markdown
## Tarea: Sistema de Moderación Centralizada + Analytics

### 1. Estructura del Módulo
```
/src/modules/moderation
├── domain/
│   ├── entities/
│   │   ├── ModerationQueue.ts
│   │   ├── Review.ts
│   │   └── Analytics.ts
│   └── repositories/
│       ├── ModerationRepository.interface.ts
│       └── AnalyticsRepository.interface.ts
├── infrastructure/
│   ├── api/
│   │   ├── moderation.api.ts
│   │   └── analytics.api.ts
│   └── adapters/
│       └── analytics.adapter.ts
├── application/
│   ├── usecases/
│   │   ├── GetPendingContent.ts
│   │   ├── ModerateContent.ts
│   │   └── GenerateReport.ts
│   └── services/
│       ├── ModerationService.ts
│       └── AnalyticsService.ts
└── presentation/
    ├── components/
    │   ├── ModerationQueue.tsx
    │   ├── ReviewInterface.tsx
    │   ├── AnalyticsDashboard.tsx
    │   ├── UserActivityLog.tsx
    │   └── SystemHealthMonitor.tsx
    ├── pages/
    │   ├── ModerationPage.tsx
    │   └── AnalyticsPage.tsx
    └── hooks/
        ├── useModeration.ts
        ├── useAnalytics.ts
        └── useSystemHealth.ts
```

### 2. Dashboard de Moderación

#### ModerationQueue.tsx
```typescript
interface ModerationQueueProps {
  userRole: UserRole
  moduleFilter?: ModuleType
}

// Funcionalidades:
// - Cola priorizada por fecha/urgencia
// - Filtros: módulo, tipo contenido, autor
// - Vista previa rápida sin salir de la cola
// - Acciones rápidas: Aprobar/Rechazar/Asignar
// - Notificaciones en tiempo real de nuevo contenido
// - Métricas: tiempo promedio de revisión
```

#### ReviewInterface.tsx
```typescript
interface ReviewInterfaceProps {
  contentId: string
  onComplete: (action: ModerationAction) => void
}

interface ModerationAction {
  action: 'approve' | 'reject' | 'request_changes'
  comment?: string
  suggestedChanges?: string[]
}

// Características:
// - Vista split: contenido original vs preview público
// - Sistema de comentarios con threading
// - Checklist de criterios de calidad
// - Historial de revisiones anteriores
// - Notificación automática al autor
```

### 3. Analytics y Reportes

#### AnalyticsDashboard.tsx
```typescript
// Métricas principales:
// - Contenido más visitado por módulo
// - Tendencias de búsqueda
// - Engagement por tipo de contenido
// - Tiempo de permanencia
// - Descargas de documentos
// - Usuarios activos vs nuevos

// Visualizaciones:
// - Gráficos de líneas (tendencias temporales)
// - Gráficos de barras (comparativas por módulo)
// - Heatmaps (actividad por hora/día)
// - Métricas en tiempo real
```

#### UserActivityLog.tsx
```typescript
interface ActivityLogEntry {
  userId: string
  action: string
  resourceType: string
  resourceId: string
  timestamp: Date
  ipAddress: string
  userAgent: string
  details: Record<string, any>
}

// Funcionalidades:
// - Log detallado de todas las acciones
// - Filtros: usuario, acción, fecha, módulo  
// - Búsqueda full-text
// - Export para auditorías
// - Detección de patrones sospechosos
```

### 4. Monitoreo del Sistema

#### SystemHealthMonitor.tsx
```typescript
// Métricas técnicas:
// - Estado de APIs externas
// - Tiempo de respuesta de endpoints
// - Errores 4xx/5xx
// - Uso de memoria/CPU (si disponible)
// - Tamaño de base de datos
// - Backups exitosos

// Alertas automáticas:
// - Email/Slack cuando métricas exceden umbrales
// - Dashboard de estado tipo "semáforo"
```
```

---

### **PROMPT 7: Design System y Componentes Compartidos**

**Objetivo**: Sistema de diseño consistente y componentes reutilizables.

```markdown
## Tarea: Design System Institucional MIMP

### 1. Estructura del Design System
```
/src/shared/components
├── ui/                 # Componentes base (atomic design)
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.stories.tsx
│   │   └── Button.test.tsx
│   ├── Input/
│   ├── Select/
│   ├── Modal/
│   ├── Table/
│   └── index.ts        # Barrel exports
├── layout/             # Componentes de layout
│   ├── Header/
│   ├── Footer/
│   ├── Sidebar/
│   └── PageContainer/
├── forms/              # Componentes de formularios
│   ├── FormField/
│   ├── SearchBar/
│   └── FileDropzone/
└── feedback/           # Feedback al usuario
    ├── Loading/
    ├── EmptyState/
    ├── ErrorBoundary/
    └── Toast/
```

### 2. Tokens de Diseño (Tailwind Config)
```javascript
// tailwind.config.js - Tema institucional MIMP
module.exports = {
  theme: {
    extend: {
      colors: {
        // Paleta institucional MIMP
        mimp: {
          primary: {
            50: '#f5f3ff',
            500: '#8b5cf6',  // Morado principal
            600: '#7c3aed',
            900: '#4c1d95',
          },
          secondary: {
            500: '#ec4899',  // Rosa MIMP
          },
          success: '#10b981',
          warning: '#f59e0b',
          danger: '#ef4444',
        },
        // Estados de contenido
        content: {
          draft: '#64748b',     // Gris
          pending: '#f59e0b',   // Amarillo
          approved: '#10b981',  // Verde
          rejected: '#ef4444',  // Rojo
        }
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        display: ['Poppins', 'sans-serif'], // Para títulos
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      },
      animation: {
        'fade-in': 'fadeIn 0.2s ease-in-out',
        'slide-up': 'slideUp 0.3s ease-out',
      }
    }
  }
}
```

### 3. Componentes Base UI

#### Button.tsx
```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger'
  size: 'sm' | 'md' | 'lg' | 'xl'
  loading?: boolean
  disabled?: boolean
  leftIcon?: React.ReactNode
  rightIcon?: React.ReactNode
  children: React.ReactNode
  onClick?: () => void
}

// Variantes visuales según Design System
// Estados: hover, active, disabled, loading
// Accesibilidad: ARIA labels, keyboard navigation
```

#### DataTable.tsx
```typescript
interface DataTableProps<T> {
  data: T[]
  columns: Column<T>[]
  loading?: boolean
  pagination?: PaginationConfig
  sorting?: SortingConfig
  filtering?: FilteringConfig
  selection?: SelectionConfig
  actions?: ActionConfig<T>[]
}

// Funcionalidades:
// - Ordenación por columnas
// - Filtros por tipo de dato
// - Paginación server-side/client-side
// - Selección múltiple con checkboxes
// - Acciones contextuales por fila
// - Estados vacíos y de carga
// - Export CSV/PDF
// - Responsive (colapsa columnas en mobile)
```

### 4. Layout Components

#### Header.tsx
```typescript
interface HeaderProps {
  user: User | null
  currentModule?: ModuleType
  notifications: Notification[]
  onLogout: () => void
}

// Elementos:
// - Logo institucional MIMP
// - Navegación principal (según rol)
// - Buscador global (solo portal público)
// - Menú de usuario con avatar
// - Notificaciones con badge de count
// - Breadcrumb navigation
```

#### Sidebar.tsx
```typescript
interface SidebarProps {
  userRole: UserRole
  currentPath: string
  modulePermissions: ModulePermission[]
  collapsed?: boolean
  onToggle: (collapsed: boolean) => void
}

// Navegación adaptiva por rol:
// - Administrador: todas las secciones
// - Editor General: moderación + configuración
// - Editor Observatorio: solo su módulo asignado
// - Estados activo/inactivo por ruta
// - Colapsar/expandir en mobile
```

### 5. Estados de Carga y Feedback

#### Loading.tsx
```typescript
// Tipos de loading:
// - Skeleton screens (para tablas/listas)
// - Spinners (para botones/acciones)
// - Progress bars (para uploads)
// - Overlay loaders (para formularios)
```

#### EmptyState.tsx
```typescript
interface EmptyStateProps {
  title: string
  description?: string
  icon?: React.ReactNode
  action?: {
    label: string
    onClick: () => void
  }
}

// Estados vacíos contextuales:
// - "No hay contenido pendiente de moderación" 
// - "No se encontraron resultados"
// - "Módulo sin contenido publicado"
// - Ilustraciones SVG institucionales
```

### 6. Accesibilidad y UX
- **ARIA labels** en todos los componentes interactivos
- **Navegación por teclado** (Tab, Enter, Escape)
- **Contraste WCAG AA** en todos los colores
- **Focus indicators** visibles
- **Screen reader** friendly
- **Responsive design** mobile-first
- **Loading states** para todas las acciones async
```

---

### **PROMPT 8: Estado Global y Data Management**

**Objetivo**: Gestión eficiente del estado y sincronización de datos.

```markdown
## Tarea: Arquitectura de Estado y Data Fetching

### 1. Estructura de Stores (Zustand)
```
/src/core/store
├── auth/
│   ├── authStore.ts
│   └── authStore.types.ts
├── modules/
│   ├── moduleStore.ts
│   └── moduleStore.types.ts  
├── content/
│   ├── contentStore.ts
│   └── contentStore.types.ts
├── ui/
│   ├── uiStore.ts
│   └── uiStore.types.ts
└── index.ts              # Store composition
```

### 2. Auth Store (Zustand + Persist)
```typescript
interface AuthState {
  // Estado
  user: User | null
  token: string | null
  isAuthenticated: boolean
  isLoading: boolean
  error: string | null

  // Acciones síncronas  
  setUser: (user: User) => void
  setToken: (token: string) => void
  clearAuth: () => void
  setError: (error: string | null) => void

  // Acciones asíncronas
  login: (credentials: LoginCredentials) => Promise<void>
  logout: () => Promise<void>
  refreshToken: () => Promise<void>
  
  // Computed (usando subscriptors)
  hasPermission: (module: ModuleType, permission: Permission) => boolean
  canAccessModule: (moduleId: ModuleType) => boolean
}

// Persistencia automática en localStorage
// Hydration en SSR/SSG
```

### 3. Module Store (Configuración de Visibilidad)
```typescript
interface ModuleState {
  // Estado
  modules: ModuleConfig[]
  activeModule: ModuleType | null
  visibilitySettings: VisibilitySettings
  isLoading: boolean

  // Acciones
  setActiveModule: (moduleId: ModuleType) => void
  updateVisibility: (moduleId: ModuleType, visible: boolean) => void
  updateSubmoduleVisibility: (moduleId: ModuleType, submoduleId: string, visible: boolean) => void
  
  // Server sync
  fetchModuleConfig: () => Promise<void>
  saveVisibilitySettings: () => Promise<void>

  // Computed
  getVisibleModules: () => ModuleConfig[]
  getModulePermissions: (moduleId: ModuleType) => Permission[]
}
```

### 4. Content Store (Gestión de Contenido)
```typescript
interface ContentState {
  // Estado
  contentByModule: Record<ModuleType, BaseContent[]>
  moderationQueue: BaseContent[]
  selectedContent: BaseContent | null
  filters: ContentFilters
  isLoading: boolean

  // Actions
  setSelectedContent: (content: BaseContent | null) => void
  updateContentStatus: (contentId: string, status: ContentStatus) => void
  addToModerationQueue: (content: BaseContent) => void
  removeFromModerationQueue: (contentId: string) => void
  
  // Optimistic updates
  createContentOptimistic: (content: Partial<BaseContent>) => string
  updateContentOptimistic: (contentId: string, updates: Partial<BaseContent>) => void
  deleteContentOptimistic: (contentId: string) => void
  
  // Server sync  
  syncContent: (moduleId: ModuleType) => Promise<void>
  moderateContent: (contentId: string, action: ModerationAction) => Promise<void>
}
```

### 5. UI Store (Estado de Interfaz)
```typescript
interface UIState {
  // Modals y overlays
  activeModal: string | null
  modalProps: Record<string, any>
  
  // Notifications
  notifications: Notification[]
  
  // Loading states globales
  globalLoading: boolean
  loadingStates: Record<string, boolean>
  
  // Sidebar y layout
  sidebarCollapsed: boolean
  currentPage: string
  
  // Actions
  openModal: (modalId: string, props?: Record<string, any>) => void
  closeModal: () => void
  addNotification: (notification: Omit<Notification, 'id'>) => void
  removeNotification: (id: string) => void
  setLoading: (key: string, loading: boolean) => void
  toggleSidebar: () => void
}
```

### 6. Data Fetching con SWR/TanStack Query

#### API Client Setup
```typescript
// /src/core/api/client.ts
class ApiClient {
  private baseURL: string
  private token: string | null = null

  constructor() {
    this.baseURL = process.env.NEXT_PUBLIC_API_URL!
  }

  setToken(token: string) {
    this.token = token
  }

  private async request<T>(
    endpoint: string, 
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.baseURL}${endpoint}`
    
    const config: RequestInit = {
      headers: {
        'Content-Type': 'application/json',
        ...(this.token && { Authorization: `Bearer ${this.token}` }),
        ...options.headers,
      },
      ...options,
    }

    const response = await fetch(url, config)
    
    if (!response.ok) {
      throw new ApiError(response.status, await response.text())
    }

    return response.json()
  }

  // Métodos públicos
  get = <T>(endpoint: string) => this.request<T>(endpoint)
  post = <T>(endpoint: string, data: any) => 
    this.request<T>(endpoint, { method: 'POST', body: JSON.stringify(data) })
  put = <T>(endpoint: string, data: any) => 
    this.request<T>(endpoint, { method: 'PUT', body: JSON.stringify(data) })
  delete = <T>(endpoint: string) => 
    this.request<T>(endpoint, { method: 'DELETE' })
}
```

#### Custom Hooks para Data Fetching
```typescript
// /src/shared/hooks/useContent.ts
export function useContent(moduleId: ModuleType, filters?: ContentFilters) {
  return useSWR(
    ['content', moduleId, filters],
    () => apiClient.get<BaseContent[]>(`/content/${moduleId}`, { params: filters }),
    {
      revalidateOnFocus: false,
      revalidateOnReconnect: true,
      dedupingInterval: 5000, // 5 segundos
    }
  )
}

// /src/shared/hooks/useMutation.ts  
export function useCreateContent() {
  return useSWRMutation(
    'content',
    async (key, { arg }: { arg: Partial<BaseContent> }) => {
      const newContent = await apiClient.post<BaseContent>('/content', arg)
      
      // Invalidar cache relacionado
      mutate(key => Array.isArray(key) && key[0] === 'content', undefined, { revalidate: true })
      
      return newContent
    }
  )
}
```

### 7. Error Handling y Loading States

#### Error Boundaries por Módulo
```typescript
// /src/shared/components/ErrorBoundary.tsx
interface ErrorBoundaryProps {
  fallback?: React.ComponentType<{ error: Error; retry: () => void }>
  onError?: (error: Error, errorInfo: ErrorInfo) => void
  children: React.ReactNode
}

// Error boundaries específicos:
// - AuthErrorBoundary: errores de autenticación → redirect login
// - ModuleErrorBoundary: errores de módulo → vista de error del módulo
// - GlobalErrorBoundary: errores no capturados → página de error general
```

#### Suspense y Loading States
```typescript
// /src/shared/components/SuspenseWrapper.tsx
interface SuspenseWrapperProps {
  fallback?: React.ReactNode
  children: React.ReactNode
}

// Diferentes tipos de loading:
// - TableSkeleton: para listas de contenido
// - FormSkeleton: para formularios
// - DashboardSkeleton: para dashboards
// - PageSkeleton: para páginas completas
```

### 8. Optimistic Updates y Offline Support

#### Optimistic Updates
```typescript
// Patrón para actualizaciones optimistas
export function useOptimisticUpdate<T>(
  mutationFn: (data: T) => Promise<T>,
  cacheKey: string
) {
  const { data: currentData, mutate } = useSWR(cacheKey)
  
  const optimisticUpdate = useCallback(async (newData: T) => {
    // 1. Actualizar cache inmediatamente (optimistic)
    mutate(newData, false)
    
    try {
      // 2. Ejecutar mutación en servidor
      const result = await mutationFn(newData)
      
      // 3. Actualizar con resultado real del servidor
      mutate(result, false)
      
      return result
    } catch (error) {
      // 4. Revertir a estado anterior en caso de error
      mutate(currentData, false)
      throw error
    }
  }, [mutationFn, mutate, currentData])

  return { optimisticUpdate }
}
```
```

---

## Orden de Implementación Recomendado

1. **PROMPT 1**: Configuración base y setup del proyecto
2. **PROMPT 7**: Design System (para tener componentes disponibles)
3. **PROMPT 2**: Sistema de autenticación (dependencia de otros módulos)
4. **PROMPT 8**: Estado global y data management
5. **PROMPT 4**: Módulo de configuración (backoffice básico)
6. **PROMPT 3**: Portal público (funcionalidad principal)
7. **PROMPT 5**: Módulos de contenido (CRUD)
8. **PROMPT 6**: Moderación y analytics (funcionalidad avanzada)

## Consideraciones Técnicas

### Performance
- **Code splitting** por módulo usando dynamic imports
- **Image optimization** con Next.js Image component
- **Bundle analysis** para detectar dependencias pesadas
- **Lazy loading** de componentes no críticos

### SEO y Accesibilidad
- **Server-side rendering** para contenido público
- **Meta tags** dinámicos por módulo
- **Structured data** (JSON-LD) para búsquedas
- **Sitemap** automático de contenido público
- **WCAG 2.1 AA** compliance

### Seguridad
- **CSRF protection** en formularios
- **XSS protection** con sanitización de HTML
- **Content Security Policy** headers
- **Rate limiting** en APIs públicas
- **Input validation** con Zod en frontend y backend

### Monitoreo
- **Error tracking** con Sentry
- **Performance monitoring** con Web Vitals
- **User analytics** con privacidad (no PII)
- **API monitoring** con health checks