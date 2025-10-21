# Технические детали проекта

## Конфигурация сборки

### Nx конфигурация

```json
{
  "targetDefaults": {
    "@angular-devkit/build-angular:application": {
      "cache": true,
      "dependsOn": ["^build"],
      "inputs": ["production", "^production"]
    },
    "@nx/jest:jest": {
      "cache": true,
      "inputs": ["default", "^production", "{workspaceRoot}/jest.preset.js"],
      "options": {
        "passWithNoTests": true
      }
    }
  }
}
```

### TypeScript конфигурация

- **Target**: ES2022
- **Module**: ESNext
- **Strict mode**: Включен
- **Decorators**: Поддержка экспериментальных декораторов
- **Path mapping**: Настроен для всех библиотек

### ESLint конфигурация

- Angular ESLint правила
- Prettier интеграция
- Import/export правила
- RxJS правила
- Cypress правила для E2E тестов

## Структура библиотек

### @breez/auth

```
libs/auth/
├── constants/
│   ├── auth-config.const.ts
│   └── index.ts
├── guards/
│   └── auth/
│       ├── auth.guard.ts
│       ├── auth.guard.spec.ts
│       └── index.ts
├── interceptors/
│   └── auth/
│       ├── auth.interceptor.ts
│       ├── auth.interceptor.spec.ts
│       └── index.ts
└── services/
    ├── auth/
    │   ├── auth.service.ts
    │   ├── auth.service.spec.ts
    │   └── index.ts
    └── auth-api/
        ├── auth-api.service.ts
        ├── auth-api.service.spec.ts
        └── index.ts
```

### @breez/core

```
libs/core/
├── classes/
│   ├── close-confirmation-dialog-handler/
│   ├── custom-control/
│   ├── list-data-source/
│   ├── selectable-table/
│   └── table/
├── components/
│   └── routable-dialog/
├── directives/
│   ├── bind-query-params/
│   ├── combo-box-auto-open/
│   ├── has-permission/
│   ├── infinite-scroll/
│   ├── native-date-range-transformer/
│   ├── native-date-transformer/
│   ├── switch-view-mode-control/
│   └── table-column-size/
├── guards/
│   ├── has-permission/
│   └── has-unsaved-changes/
├── helpers/
│   ├── bulk-actions-handler/
│   ├── combine-latest-into-object/
│   ├── country-code-to-flag/
│   ├── get-enum-value-by-index/
│   ├── identity-matcher-by-id/
│   ├── is-defined/
│   ├── is-internal-api-url/
│   ├── map-instance-to-plain/
│   ├── negative-boolean/
│   ├── phone-number-format/
│   ├── to-form-data/
│   ├── to-id/
│   ├── to-result/
│   ├── to-tui-day/
│   └── transform-keys/
├── interceptors/
│   └── transform-properties/
├── pipes/
│   ├── country-code-formatter/
│   ├── date-formatter/
│   ├── format-to-array-query-param/
│   └── phone-number-formatter/
├── providers/
│   ├── actions-service.provider.ts
│   ├── list-data-source-service.provider.ts
│   └── validation-errors.provider.ts
├── services/
│   ├── api/
│   ├── centrifuge/
│   ├── copy/
│   ├── environment/
│   ├── lazy-options/
│   ├── notifications/
│   ├── permissions/
│   ├── selection/
│   ├── table-bar/
│   └── table-selection/
└── types/
    ├── api-request-options.type.ts
    ├── bulk-response-map.type.ts
    ├── bulk-response.type.ts
    ├── constructor.type.ts
    ├── entity-list.type.ts
    ├── identifiable.type.ts
    ├── link.type.ts
    ├── list-data-source-adapter.type.ts
    └── list-query-params.type.ts
```

### @breez-fake-app/api/\*

Каждая API библиотека содержит:

- **Services**: HTTP сервисы для работы с API
- **Models**: TypeScript интерфейсы и классы
- **Enums**: Перечисления для типов данных
- **Constants**: Константы и маппинги

### @breez-fake-app/features/\*

Каждая feature библиотека содержит:

- **Components**: UI компоненты для feature
- **Routes**: Маршруты для lazy loading
- **Services**: Бизнес-логика feature
- **Models**: Модели данных feature
- **Constants**: Константы feature

## Система разрешений

### Permission enum

```typescript
export enum Permission {
  // Accounts
  AccountsRead = 'accounts:read',
  AccountsWrite = 'accounts:write',

  // Bots
  BotsRead = 'bots:read',
  BotsWrite = 'bots:write',
  BotsContentsRead = 'bots:contents:read',
  BotsContentsWrite = 'bots:contents:write',

  // Cycles
  CyclesRead = 'cycles:read',
  CyclesWrite = 'cycles:write',

  // Media
  MediaFoldersRead = 'media:folders:read',
  MediaFoldersWrite = 'media:folders:write',

  // Analytics
  AnalyticsMetricsRead = 'analytics:metrics:read',

  // Numbers
  NumbersRead = 'numbers:read',
  NumbersWrite = 'numbers:write',

  // Payments
  PaymentsSellersPhrasesRead = 'payments:sellers:phrases:read',
  PaymentsSellersPhrasesWrite = 'payments:sellers:phrases:write',
}
```

### HasPermission Guard

```typescript
export const hasPermissionGuard =
  (permissions: Permission | Permission[]) => (route: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
    // Проверка разрешений пользователя
  };
```

## Система маршрутизации

### Lazy Loading

Все feature модули загружаются лениво:

```typescript
{
  path: 'loops',
  canMatch: [hasPermissionGuard([Permission.BotsRead, Permission.CyclesRead, Permission.BotsContentsRead])],
  loadChildren: () => import('@breez-fake-app/loops/routes').then((m) => m.routes),
}
```

### Route Guards

- **authGuard**: Проверка аутентификации
- **hasPermissionGuard**: Проверка разрешений
- **hasUnsavedChangesGuard**: Проверка несохраненных изменений

## HTTP Interceptors

### Auth Interceptor

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Добавление токена авторизации
    // Обновление токена при необходимости
  }
}
```

### Error Handling Interceptor

```typescript
@Injectable()
export class ErrorHandlingInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Обработка HTTP ошибок
    // Показ уведомлений пользователю
  }
}
```

### Transform Properties Interceptor

```typescript
@Injectable()
export class TransformPropertiesInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Трансформация свойств запросов/ответов
    // Использование class-transformer
  }
}
```

## Real-time функциональность

### Centrifuge Service

```typescript
@Injectable()
export class CentrifugeService {
  init(): Observable<void> {
    // Инициализация WebSocket соединения
  }

  subscribe(channel: string): Observable<CentrifugeMessageData> {
    // Подписка на канал
  }

  disconnect(): void {
    // Отключение от WebSocket
  }
}
```

### Message Types

```typescript
export interface CentrifugeMessageData {
  type: string;
  payload: any;
  timestamp: number;
}
```

## Система тестирования

### Jest конфигурация

```javascript
module.exports = {
  displayName: 'breez-fake-app',
  preset: '../../jest.preset.js',
  setupFilesAfterEnv: ['<rootDir>/src/test-setup.ts'],
  coverageDirectory: '../../coverage/apps/breez-fake-app',
  transform: {
    '^.+\\.(ts|mjs|js|html)$': [
      'jest-preset-angular',
      {
        tsconfig: '<rootDir>/tsconfig.spec.json',
        stringifyContentPathRegex: '\\.(html|svg)$',
      },
    ],
  },
  transformIgnorePatterns: ['node_modules/(?!.*\\.mjs$)'],
  snapshotSerializers: [
    'jest-preset-angular/build/serializers/no-ng-attributes',
    'jest-preset-angular/build/serializers/ng-snapshot',
    'jest-preset-angular/build/serializers/html-comment',
  ],
};
```

### Cypress конфигурация

```typescript
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    baseUrl: 'http://localhost:4200',
    supportFile: 'src/support/e2e.ts',
    specPattern: 'src/e2e/**/*.cy.ts',
    video: false,
    screenshotOnRunFailure: false,
  },
});
```

## Система стилей

### SCSS структура

```
apps/breez-fake-app/src/
├── styles.scss (глобальные стили)
└── theme/
    ├── _variables.scss
    ├── _mixins.scss
    ├── _components.scss
    └── _utilities.scss
```

### Taiga UI интеграция

```scss
// Глобальные стили Taiga UI
@import '~@taiga-ui/core/styles/taiga-ui-theme.less';
@import '~@taiga-ui/core/styles/taiga-ui-fonts.less';
@import '~@taiga-ui/styles/taiga-ui-global.less';
```

## Система сборки

### Custom ESBuild

```typescript
// add-env-esbuild.plugin.ts
export function addEnvPlugin(): Plugin {
  return {
    name: 'add-env',
    setup(build) {
      // Добавление переменных окружения
    },
  };
}
```

### Build конфигурация

```json
{
  "build": {
    "executor": "@angular-builders/custom-esbuild:application",
    "options": {
      "plugins": ["./add-env-esbuild.plugin.ts"],
      "allowedCommonJsDependencies": ["@taiga-ui/core", "@taiga-ui/kit", "dompurify", "reflect-metadata"],
      "stylePreprocessorOptions": {
        "includePaths": ["apps/breez-fake-app/src/theme"]
      }
    }
  }
}
```

## Переменные окружения

### Environment файлы

```typescript
// apps/breez-fake-app/src/environments/environment.ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api',
  centrifugeUrl: 'ws://localhost:8000/connection/websocket',
  version: '1.0.0',
};
```

### Proxy конфигурация

```json
{
  "/api/*": {
    "target": "http://localhost:3000",
    "secure": false,
    "changeOrigin": true
  }
}
```

## Система уведомлений

### Notification Service

```typescript
@Injectable()
export class NotificationService {
  showSuccess(message: string): void {
    // Показ уведомления об успехе
  }

  showError(message: string): void {
    // Показ уведомления об ошибке
  }

  showWarning(message: string): void {
    // Показ предупреждения
  }
}
```

## Система валидации

### Validation Errors Provider

```typescript
export const VALIDATION_ERRORS_PROVIDER: Provider = {
  provide: VALIDATION_ERRORS,
  useValue: {
    required: 'Поле обязательно для заполнения',
    email: 'Некорректный email адрес',
    minlength: 'Минимальная длина: {requiredLength}',
    maxlength: 'Максимальная длина: {requiredLength}',
  },
};
```

## Система кэширования

### Nx Cache

- Локальное кэширование результатов сборки
- Удаленное кэширование через Nx Cloud
- Инкрементальная сборка
- Кэширование тестов

### HTTP Cache

- Кэширование API запросов
- Инвалидация кэша при изменениях
- Стратегии кэширования по типам данных

## Мониторинг производительности

### Bundle Analysis

- Анализ размера бандла
- Tree shaking оптимизация
- Lazy loading метрики
- Code splitting анализ

### Runtime Metrics

- Время загрузки модулей
- Производительность компонентов
- Memory usage мониторинг
- Network requests анализ
