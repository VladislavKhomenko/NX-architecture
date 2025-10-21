# Руководство по интеграции микрофронтов

## Обзор

Данное руководство описывает процесс интеграции микрофронтов в существующую архитектуру проекта breez-fake-app. Текущая модульная архитектура уже подготовлена для такого перехода, что значительно упрощает процесс миграции.

## Анализ готовности текущей архитектуры

### ✅ Преимущества текущей архитектуры

#### 1. Модульность

- Каждая функциональность выделена в отдельную библиотеку
- Четкое разделение ответственности между модулями
- Независимые API библиотеки для каждого домена

#### 2. Lazy Loading

- Все feature модули загружаются лениво
- Готовая система маршрутизации
- Оптимизированная загрузка ресурсов

#### 3. Компонентная архитектура

- Standalone компоненты (Angular 17+)
- Переиспользуемые UI компоненты
- SCAM (Single Component Angular Module) подход

#### 4. API изоляция

- Отдельные API библиотеки для каждого домена
- Типизированные модели данных
- Централизованное управление HTTP запросами

### 📊 Оценка готовности

| Аспект                       | Готовность | Комментарий                      |
| ---------------------------- | ---------- | -------------------------------- |
| **Модульность**              | 10/10      | Каждая feature уже изолирована   |
| **Lazy Loading**             | 10/10      | Все модули загружаются лениво    |
| **API изоляция**             | 9/10       | Отдельные API библиотеки         |
| **Компонентная архитектура** | 9/10       | Standalone компоненты            |
| **Маршрутизация**            | 8/10       | Нужна настройка для микрофронтов |
| **Общая готовность**         | **9/10**   | Отличная основа для микрофронтов |

## Стратегии интеграции

### 1. Module Federation (Рекомендуемый подход)

**Преимущества:**

- Нативная поддержка в Nx
- Минимальные изменения в коде
- Отличная производительность
- Простота настройки

**Недостатки:**

- Привязка к webpack
- Ограниченная гибкость

### 2. Single-SPA (Альтернативный подход)

**Преимущества:**

- Технологическая независимость
- Большая гибкость
- Поддержка разных фреймворков

**Недостатки:**

- Больше настройки
- Сложность интеграции

## План миграции

### Этап 1: Подготовка инфраструктуры (1-2 недели)

#### 1.1 Установка зависимостей

```bash
# Установка Module Federation
npm install @module-federation/webpack

# Установка дополнительных зависимостей
npm install @nx/webpack
```

#### 1.2 Настройка Nx конфигурации

```json
// nx.json
{
  "projects": {
    "breez-fake-app-shell": "apps/breez-fake-app-shell",
    "accounts-mfe": "apps/accounts-mfe",
    "analytics-mfe": "apps/analytics-mfe",
    "loops-mfe": "apps/loops-mfe",
    "media-mfe": "apps/media-mfe",
    "ton-numbers-mfe": "apps/ton-numbers-mfe",
    "rotations-mfe": "apps/rotations-mfe"
  }
}
```

#### 1.3 Создание shell приложения

```bash
# Генерация host приложения
nx generate @nx/angular:host breez-fake-app-shell

# Настройка конфигурации
nx generate @nx/webpack:webpack breez-fake-app-shell
```

### Этап 2: Миграция первого микрофронта (1 неделя)

#### 2.1 Создание accounts-mfe

```bash
# Генерация remote приложения
nx generate @nx/angular:remote accounts-mfe --host=breez-fake-app-shell

# Структура проекта
apps/accounts-mfe/
├── src/
│   ├── main.ts
│   ├── bootstrap.ts
│   ├── app/
│   │   ├── accounts.component.ts
│   │   ├── accounts.routes.ts
│   │   └── accounts.module.ts
│   └── assets/
├── webpack.config.js
├── project.json
└── tsconfig.json
```

#### 2.2 Конфигурация webpack

```typescript
// apps/accounts-mfe/webpack.config.js
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
  mode: 'development',
  devServer: {
    port: 4201,
    headers: {
      'Access-Control-Allow-Origin': '*',
    },
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'accounts',
      filename: 'remoteEntry.js',
      exposes: {
        './AccountsModule': './src/app/accounts.module.ts',
        './AccountsComponent': './src/app/accounts.component.ts',
      },
      shared: {
        '@angular/core': { singleton: true, strictVersion: true },
        '@angular/common': { singleton: true, strictVersion: true },
        '@angular/router': { singleton: true, strictVersion: true },
        '@taiga-ui/core': { singleton: true },
        '@taiga-ui/kit': { singleton: true },
        rxjs: { singleton: true },
      },
    }),
  ],
};
```

#### 2.3 Миграция кода

```typescript
// apps/accounts-mfe/src/app/accounts.module.ts
import { NgModule } from '@angular/core';
import { RouterModule } from '@angular/router';
import { CommonModule } from '@angular/common';

import { AccountsComponent } from './accounts.component';
import { accountsRoutes } from './accounts.routes';

@NgModule({
  declarations: [AccountsComponent],
  imports: [
    CommonModule,
    RouterModule.forChild(accountsRoutes),
    // Импорты из @breez-fake-app/accounts
  ],
})
export class AccountsModule {}
```

```typescript
// apps/accounts-mfe/src/app/accounts.routes.ts
import { Routes } from '@angular/router';

export const accountsRoutes: Routes = [
  {
    path: '',
    component: AccountsComponent,
    children: [
      {
        path: 'bots',
        loadChildren: () => import('@breez-fake-app/accounts/bots').then((m) => m.routes),
      },
      {
        path: 'channels',
        loadChildren: () => import('@breez-fake-app/accounts/channels').then((m) => m.routes),
      },
      {
        path: 'posting',
        loadChildren: () => import('@breez-fake-app/accounts/posting').then((m) => m.routes),
      },
      {
        path: 'sales',
        loadChildren: () => import('@breez-fake-app/accounts/sales').then((m) => m.routes),
      },
    ],
  },
];
```

### Этап 3: Обновление shell приложения (1 неделя)

#### 3.1 Конфигурация shell webpack

```typescript
// apps/breez-fake-app-shell/webpack.config.js
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
  mode: 'development',
  devServer: {
    port: 4200,
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'breez_fake_app_shell',
      remotes: {
        accounts: 'accounts@http://localhost:4201/remoteEntry.js',
        analytics: 'analytics@http://localhost:4202/remoteEntry.js',
        loops: 'loops@http://localhost:4203/remoteEntry.js',
        media: 'media@http://localhost:4204/remoteEntry.js',
        'ton-numbers': 'tonNumbers@http://localhost:4205/remoteEntry.js',
        rotations: 'rotations@http://localhost:4206/remoteEntry.js',
      },
      shared: {
        '@angular/core': { singleton: true, strictVersion: true },
        '@angular/common': { singleton: true, strictVersion: true },
        '@angular/router': { singleton: true, strictVersion: true },
        '@angular/forms': { singleton: true, strictVersion: true },
        '@taiga-ui/core': { singleton: true },
        '@taiga-ui/kit': { singleton: true },
        '@taiga-ui/cdk': { singleton: true },
        rxjs: { singleton: true },
        'class-transformer': { singleton: true },
      },
    }),
  ],
};
```

#### 3.2 Обновление маршрутов

```typescript
// apps/breez-fake-app-shell/src/app/app.routes.ts
import { Routes } from '@angular/router';

import { authGuard } from '@breez/auth/guards';
import { Permission } from '@breez/core/enums';
import { hasPermissionGuard } from '@breez/core/guards';
import { AccessDeniedComponent } from '@breez/error-handling/ui';

export const routes: Routes = [
  {
    path: '',
    canActivate: [authGuard],
    children: [
      {
        path: 'loops',
        canMatch: [hasPermissionGuard([Permission.BotsRead, Permission.CyclesRead, Permission.BotsContentsRead])],
        loadChildren: () => import('loops/LoopsModule').then((m) => m.LoopsModule),
      },
      {
        path: 'media',
        canMatch: [hasPermissionGuard(Permission.MediaFoldersRead)],
        loadChildren: () => import('media/MediaModule').then((m) => m.MediaModule),
      },
      {
        path: 'ton-numbers',
        canMatch: [hasPermissionGuard(Permission.NumbersRead)],
        loadChildren: () => import('tonNumbers/TonNumbersModule').then((m) => m.TonNumbersModule),
      },
      {
        path: 'accounts',
        canMatch: [hasPermissionGuard(Permission.AccountsRead)],
        loadChildren: () => import('accounts/AccountsModule').then((m) => m.AccountsModule),
      },
      {
        path: 'analytics',
        canMatch: [hasPermissionGuard(Permission.AnalyticsMetricsRead)],
        loadChildren: () => import('analytics/AnalyticsModule').then((m) => m.AnalyticsModule),
      },
      {
        path: 'rotations',
        canMatch: [hasPermissionGuard(Permission.PaymentsSellersPhrasesRead)],
        loadChildren: () => import('rotations/RotationsModule').then((m) => m.RotationsModule),
      },
      {
        path: 'access-denied',
        component: AccessDeniedComponent,
      },
      {
        path: '',
        redirectTo: 'loops',
        pathMatch: 'full',
      },
    ],
  },
  { path: '**', redirectTo: 'loops', pathMatch: 'full' },
];
```

#### 3.3 Обновление shell компонента

```typescript
// apps/breez-fake-app-shell/src/app/app.component.ts
import { NgOptimizedImage, NgTemplateOutlet } from '@angular/common';
import { ChangeDetectionStrategy, Component, DestroyRef, inject, OnDestroy, OnInit } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { Meta } from '@angular/platform-browser';
import { RouterOutlet } from '@angular/router';
import { TuiTableBarsHostModule } from '@taiga-ui/addon-tablebars';
import { TUI_SANITIZER, TuiAlertModule, TuiButtonModule, TuiDialogModule, TuiRootModule } from '@taiga-ui/core';
import { TuiPushModule } from '@taiga-ui/kit';
import { NgDompurifySanitizer } from '@tinkoff/ng-dompurify';
import { filter, Observable, switchMap } from 'rxjs';

import { AuthService } from '@breez/auth/services';
import { CentrifugeMessageData, CentrifugeService } from '@breez/core/services';
import { ErrorService } from '@breez/error-handling/services';

import { User, UserService } from '@breez-fake-app/api/users';

import { environment } from '../environments/environment';

import { HeaderComponent } from './components';

@Component({
  selector: 'sp-app-root',
  imports: [
    HeaderComponent,
    NgOptimizedImage,
    NgTemplateOutlet,
    RouterOutlet,
    TuiAlertModule,
    TuiButtonModule,
    TuiDialogModule,
    TuiRootModule,
    TuiPushModule,
    TuiTableBarsHostModule,
  ],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss',
  providers: [{ provide: TUI_SANITIZER, useClass: NgDompurifySanitizer }],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class AppComponent implements OnInit, OnDestroy {
  readonly #meta = inject(Meta);
  readonly #authService = inject(AuthService);
  readonly #errorService = inject(ErrorService);
  readonly #centrifugeService = inject(CentrifugeService);
  readonly #userService = inject(UserService);
  readonly #destroyRef = inject(DestroyRef);

  readonly $isLoggedIn = this.#authService.$isLoggedIn;
  readonly $error = this.#errorService.$error;

  ngOnInit(): void {
    this.#initCentrifuge();
    this.#subscribeToUserChannel();
    this.#addTag();
  }

  refresh(): void {
    return window.location.reload();
  }

  ngOnDestroy(): void {
    this.#centrifugeService.disconnect();
  }

  #initCentrifuge(): void {
    this.#centrifugeService.init().pipe(takeUntilDestroyed(this.#destroyRef)).subscribe();
  }

  #subscribeToUserChannel(): void {
    const toUserChannel = (user: User): Observable<CentrifugeMessageData> => this.#centrifugeService.subscribe(user.id);

    this.#userService.user$.pipe(filter(Boolean), switchMap(toUserChannel)).pipe(takeUntilDestroyed(this.#destroyRef)).subscribe();
  }

  #addTag(): void {
    this.#meta.addTag({ name: 'version', content: environment.version });
  }
}
```

### Этап 4: Миграция остальных микрофронтов (1-2 недели)

#### 4.1 Создание всех микрофронтов

```bash
# Генерация всех remote приложений
nx generate @nx/angular:remote analytics-mfe --host=breez-fake-app-shell
nx generate @nx/angular:remote loops-mfe --host=breez-fake-app-shell
nx generate @nx/angular:remote media-mfe --host=breez-fake-app-shell
nx generate @nx/angular:remote ton-numbers-mfe --host=breez-fake-app-shell
nx generate @nx/angular:remote rotations-mfe --host=breez-fake-app-shell
```

#### 4.2 Настройка портов

```typescript
// Конфигурация портов для каждого микрофронта
const MFE_PORTS = {
  'breez-fake-app-shell': 4200,
  'accounts-mfe': 4201,
  'analytics-mfe': 4202,
  'loops-mfe': 4203,
  'media-mfe': 4204,
  'ton-numbers-mfe': 4205,
  'rotations-mfe': 4206,
};
```

## Структура после миграции

### Диаграмма архитектуры микрофронтов

```mermaid
graph TB
    subgraph "Shell Application"
        A[breez-fake-app Shell<br/>Port: 4200]
    end

    subgraph "Micro Frontends"
        B[Accounts MFE<br/>Port: 4201]
        C[Analytics MFE<br/>Port: 4202]
        D[Loops MFE<br/>Port: 4203]
        E[Media MFE<br/>Port: 4204]
        F[TON Numbers MFE<br/>Port: 4205]
        G[Rotations MFE<br/>Port: 4206]
    end

    subgraph "Shared Libraries"
        H[@breez/auth]
        I[@breez/core]
        J[@breez/error-handling]
        K[@breez-fake-app/api/*]
        L[@breez-fake-app/components/*]
        M[@breez-fake-app/ui/*]
    end

    A --> B
    A --> C
    A --> D
    A --> E
    A --> F
    A --> G

    B --> H
    B --> I
    B --> J
    B --> K
    B --> L
    B --> M

    C --> H
    C --> I
    C --> J
    C --> K
    C --> L
    C --> M

    D --> H
    D --> I
    D --> J
    D --> K
    D --> L
    D --> M
```

### Финальная структура проекта

```
apps/
├── breez-fake-app-shell/           # Host приложение
│   ├── src/
│   │   ├── main.ts
│   │   ├── app/
│   │   │   ├── app.component.ts
│   │   │   ├── app.routes.ts
│   │   │   └── components/
│   │   └── environments/
│   ├── webpack.config.js
│   └── project.json
├── accounts-mfe/           # Accounts микрофронт
│   ├── src/
│   │   ├── main.ts
│   │   ├── bootstrap.ts
│   │   └── app/
│   ├── webpack.config.js
│   └── project.json
├── analytics-mfe/          # Analytics микрофронт
├── loops-mfe/              # Loops микрофронт
├── media-mfe/              # Media микрофронт
├── ton-numbers-mfe/        # TON Numbers микрофронт
└── rotations-mfe/          # Rotations микрофронт

libs/
├── auth/                   # Общие библиотеки
├── core/
├── error-handling/
├── breez-fake-app/
│   ├── api/               # API библиотеки
│   ├── components/        # Переиспользуемые компоненты
│   ├── services/          # Общие сервисы
│   └── ui/                # UI библиотека
└── tree/
```

## Конфигурация разработки

### 1. Package.json скрипты

```json
{
  "scripts": {
    "start": "nx serve breez-fake-app-shell",
    "start:accounts": "nx serve accounts-mfe",
    "start:analytics": "nx serve analytics-mfe",
    "start:loops": "nx serve loops-mfe",
    "start:media": "nx serve media-mfe",
    "start:ton-numbers": "nx serve ton-numbers-mfe",
    "start:rotations": "nx serve rotations-mfe",
    "start:all": "concurrently \"npm run start\" \"npm run start:accounts\" \"npm run start:analytics\" \"npm run start:loops\" \"npm run start:media\" \"npm run start:ton-numbers\" \"npm run start:rotations\"",
    "build:all": "nx run-many --target=build --all",
    "test:all": "nx run-many --target=test --all"
  }
}
```

### 2. Environment конфигурация

```typescript
// apps/breez-fake-app-shell/src/environments/environment.ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api',
  centrifugeUrl: 'ws://localhost:8000/connection/websocket',
  version: '1.0.0',
  microfrontends: {
    accounts: 'http://localhost:4201/remoteEntry.js',
    analytics: 'http://localhost:4202/remoteEntry.js',
    loops: 'http://localhost:4203/remoteEntry.js',
    media: 'http://localhost:4204/remoteEntry.js',
    tonNumbers: 'http://localhost:4205/remoteEntry.js',
    rotations: 'http://localhost:4206/remoteEntry.js',
  },
};
```

```typescript
// apps/breez-fake-app-shell/src/environments/environment.prod.ts
export const environment = {
  production: true,
  apiUrl: 'https://api.production.com',
  centrifugeUrl: 'wss://ws.production.com/connection/websocket',
  version: '1.0.0',
  microfrontends: {
    accounts: 'https://accounts.production.com/remoteEntry.js',
    analytics: 'https://analytics.production.com/remoteEntry.js',
    loops: 'https://loops.production.com/remoteEntry.js',
    media: 'https://media.production.com/remoteEntry.js',
    tonNumbers: 'https://ton-numbers.production.com/remoteEntry.js',
    rotations: 'https://rotations.production.com/remoteEntry.js',
  },
};
```

## CI/CD для микрофронтов

### 1. GitHub Actions конфигурация

```yaml
# .github/workflows/microfrontends.yml
name: Micro Frontends CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        project: [breez-fake-app-shell, accounts-mfe, analytics-mfe, loops-mfe, media-mfe, ton-numbers-mfe, rotations-mfe]

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: nx build ${{ matrix.project }}

      - name: Run tests
        run: nx test ${{ matrix.project }}

      - name: Run linting
        run: nx lint ${{ matrix.project }}

  deploy:
    needs: build-and-test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Deploy to production
        run: |
          # Деплой каждого микрофронта
          nx deploy breez-fake-app-shell
          nx deploy accounts-mfe
          nx deploy analytics-mfe
          nx deploy loops-mfe
          nx deploy media-mfe
          nx deploy ton-numbers-mfe
          nx deploy rotations-mfe
```

### 2. Docker конфигурация

```dockerfile
# Dockerfile для shell приложения
FROM nginx:alpine

COPY dist/apps/breez-fake-app-shell/browser /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf для shell приложения
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;

        location / {
            try_files $uri $uri/ /index.html;
        }

        # Проксирование микрофронтов
        location /accounts/ {
            proxy_pass http://accounts-mfe:4201/;
        }

        location /analytics/ {
            proxy_pass http://analytics-mfe:4202/;
        }

        location /loops/ {
            proxy_pass http://loops-mfe:4203/;
        }

        location /media/ {
            proxy_pass http://media-mfe:4204/;
        }

        location /ton-numbers/ {
            proxy_pass http://ton-numbers-mfe:4205/;
        }

        location /rotations/ {
            proxy_pass http://rotations-mfe:4206/;
        }
    }
}
```

## Мониторинг и отладка

### 1. Логирование

```typescript
// apps/breez-fake-app-shell/src/app/services/microfrontend-logger.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class MicrofrontendLoggerService {
  log(microfrontend: string, message: string, data?: any): void {
    console.log(`[${microfrontend}] ${message}`, data);
  }

  error(microfrontend: string, error: Error): void {
    console.error(`[${microfrontend}] Error:`, error);
  }

  warn(microfrontend: string, message: string): void {
    console.warn(`[${microfrontend}] Warning: ${message}`);
  }
}
```

### 2. Мониторинг производительности

```typescript
// apps/breez-fake-app-shell/src/app/services/performance-monitor.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class PerformanceMonitorService {
  measureMicrofrontendLoad(microfrontend: string): void {
    const startTime = performance.now();

    return () => {
      const endTime = performance.now();
      const loadTime = endTime - startTime;

      console.log(`[${microfrontend}] Load time: ${loadTime}ms`);

      // Отправка метрик в систему мониторинга
      this.sendMetrics(microfrontend, loadTime);
    };
  }

  private sendMetrics(microfrontend: string, loadTime: number): void {
    // Интеграция с системой мониторинга
    // Например, отправка в DataDog, New Relic и т.д.
  }
}
```

## Тестирование микрофронтов

### 1. Unit тесты

```typescript
// apps/accounts-mfe/src/app/accounts.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { AccountsComponent } from './accounts.component';

describe('AccountsComponent', () => {
  let component: AccountsComponent;
  let fixture: ComponentFixture<AccountsComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [AccountsComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(AccountsComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should load accounts data', () => {
    // Тестирование загрузки данных
  });
});
```

### 2. Integration тесты

```typescript
// apps/breez-fake-app-shell/src/app/integration/microfrontend-integration.spec.ts
import { TestBed } from '@angular/core/testing';
import { RouterTestingModule } from '@angular/router/testing';
import { AppComponent } from '../app.component';

describe('Microfrontend Integration', () => {
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [AppComponent, RouterTestingModule],
    }).compileComponents();
  });

  it('should load accounts microfrontend', async () => {
    // Тестирование загрузки accounts микрофронта
  });

  it('should handle microfrontend errors gracefully', async () => {
    // Тестирование обработки ошибок
  });
});
```

### 3. E2E тесты

```typescript
// apps/breez-fake-app-e2e/src/e2e/microfrontends.cy.ts
describe('Micro Frontends E2E', () => {
  beforeEach(() => {
    cy.visit('/');
  });

  it('should navigate to accounts microfrontend', () => {
    cy.get('[data-cy="accounts-link"]').click();
    cy.url().should('include', '/accounts');
    cy.get('[data-cy="accounts-content"]').should('be.visible');
  });

  it('should navigate to analytics microfrontend', () => {
    cy.get('[data-cy="analytics-link"]').click();
    cy.url().should('include', '/analytics');
    cy.get('[data-cy="analytics-content"]').should('be.visible');
  });

  it('should handle microfrontend loading states', () => {
    cy.get('[data-cy="accounts-link"]').click();
    cy.get('[data-cy="loading-spinner"]').should('be.visible');
    cy.get('[data-cy="accounts-content"]').should('be.visible');
  });
});
```

## Лучшие практики

### 1. Управление состоянием

```typescript
// Использование shared state через RxJS
// apps/breez-fake-app-shell/src/app/services/shared-state.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class SharedStateService {
  private userSubject = new BehaviorSubject<any>(null);
  public user$ = this.userSubject.asObservable();

  setUser(user: any): void {
    this.userSubject.next(user);
  }

  getUser(): any {
    return this.userSubject.value;
  }
}
```

### 2. Обработка ошибок

```typescript
// apps/breez-fake-app-shell/src/app/services/microfrontend-error-handler.service.ts
import { Injectable } from '@angular/core';
import { ErrorService } from '@breez/error-handling/services';

@Injectable({
  providedIn: 'root',
})
export class MicrofrontendErrorHandlerService {
  constructor(private errorService: ErrorService) {}

  handleMicrofrontendError(microfrontend: string, error: Error): void {
    console.error(`Error in ${microfrontend}:`, error);

    this.errorService.showError(`Ошибка в модуле ${microfrontend}. Попробуйте обновить страницу.`);
  }
}
```

### 3. Безопасность

```typescript
// Валидация загружаемых микрофронтов
// apps/breez-fake-app-shell/src/app/services/microfrontend-security.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class MicrofrontendSecurityService {
  private allowedMicrofrontends = ['accounts', 'analytics', 'loops', 'media', 'ton-numbers', 'rotations'];

  isMicrofrontendAllowed(name: string): boolean {
    return this.allowedMicrofrontends.includes(name);
  }

  validateMicrofrontendUrl(url: string): boolean {
    // Валидация URL микрофронта
    try {
      const parsedUrl = new URL(url);
      return parsedUrl.protocol === 'https:' || parsedUrl.hostname === 'localhost';
    } catch {
      return false;
    }
  }
}
```

## Производительность

### 1. Оптимизация загрузки

```typescript
// Lazy loading с preloading
// apps/breez-fake-app-shell/src/app/app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withPreloading, PreloadAllModules } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes, withPreloading(PreloadAllModules))],
};
```

### 2. Кэширование

```typescript
// Service Worker для кэширования микрофронтов
// apps/breez-fake-app-shell/src/app/services/microfrontend-cache.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class MicrofrontendCacheService {
  private cache = new Map<string, any>();

  set(microfrontend: string, data: any): void {
    this.cache.set(microfrontend, data);
  }

  get(microfrontend: string): any {
    return this.cache.get(microfrontend);
  }

  has(microfrontend: string): boolean {
    return this.cache.has(microfrontend);
  }

  clear(): void {
    this.cache.clear();
  }
}
```

## Заключение

### Преимущества миграции

✅ **Независимые развертывания** - каждый микрофронт может развертываться отдельно  
✅ **Командная автономия** - команды могут работать независимо  
✅ **Масштабируемость** - легкое добавление новых микрофронтов  
✅ **Технологическая гибкость** - возможность использовать разные технологии  
✅ **Отказоустойчивость** - ошибка в одном микрофронте не влияет на другие

### Временные затраты

- **Подготовка**: 1-2 недели
- **Миграция первого микрофронта**: 1 неделя
- **Миграция остальных**: 1-2 недели
- **Тестирование и оптимизация**: 1 неделя

**Общее время: 4-6 недель**

### Рекомендации

1. **Начните с одного микрофронта** - протестируйте подход на accounts
2. **Используйте Module Federation** - лучший выбор для Angular + Nx
3. **Настройте мониторинг** - отслеживайте производительность и ошибки
4. **Документируйте процесс** - создайте руководства для команды
5. **Планируйте CI/CD** - настройте автоматическое развертывание

Текущая архитектура проекта **идеально подготовлена** для интеграции микрофронтов, что делает процесс миграции быстрым и эффективным.
