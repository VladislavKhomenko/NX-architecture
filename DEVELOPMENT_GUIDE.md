# Руководство по разработке

## Быстрый старт

### Установка зависимостей

```bash
npm install
```

### Запуск в режиме разработки

```bash
npm start
# или
nx serve breez-fake-app
```

### Сборка для продакшена

```bash
npm run build
# или
nx build breez-fake-app
```

### Запуск тестов

```bash
# Unit тесты
npm test
# или
nx test

# E2E тесты
npm run testing
# или
nx e2e breez-fake-app-e2e
```

## Структура команд

### Основные команды

```bash
# Разработка
npm start                    # Запуск dev сервера
npm run build               # Сборка для продакшена
npm run serve:static        # Запуск статического сервера

# Тестирование
npm test                    # Unit тесты
npm run affected:test       # Тесты для измененных файлов
npm run testing             # E2E тесты (Cypress)

# Качество кода
npm run lint                # ESLint проверка
npm run lint:fix            # ESLint с автоисправлением
npm run stylelint           # Stylelint проверка
npm run stylelint:fix       # Stylelint с автоисправлением

# Анализ
npm run dep-graph           # Граф зависимостей
nx graph                    # Визуализация проекта
```

### Nx команды

```bash
# Генерация
nx generate @nx/angular:component my-component
nx generate @nx/angular:service my-service
nx generate @nx/angular:library my-lib

# Выполнение задач
nx run breez-fake-app:build
nx run breez-fake-app:test
nx run breez-fake-app:lint

# Анализ
nx affected:build
nx affected:test
nx affected:lint
```

## Создание новых компонентов

### 1. Генерация компонента

```bash
nx generate @nx/angular:component my-component --project=breez-fake-app
```

### 2. Структура компонента

```typescript
@Component({
  selector: 'sp-my-component',
  standalone: true,
  imports: [CommonModule, TuiButtonModule],
  templateUrl: './my-component.component.html',
  styleUrl: './my-component.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class MyComponentComponent {
  // Логика компонента
}
```

### 3. Добавление в библиотеку

```typescript
// libs/breez-fake-app/components/my-component/src/index.ts
export * from './my-component.component';
```

## Создание новых сервисов

### 1. Генерация сервиса

```bash
nx generate @nx/angular:service my-service --project=breez-fake-app
```

### 2. Структура сервиса

```typescript
@Injectable({
  providedIn: 'root',
})
export class MyService {
  constructor(private http: HttpClient) {}

  getData(): Observable<Data[]> {
    return this.http.get<Data[]>('/api/data');
  }
}
```

## Создание новых библиотек

### 1. Генерация библиотеки

```bash
nx generate @nx/angular:library my-lib
```

### 2. Настройка path mapping

```json
// tsconfig.base.json
{
  "paths": {
    "@breez-fake-app/my-lib": ["libs/breez-fake-app/my-lib/src/index.ts"]
  }
}
```

### 3. Экспорт из библиотеки

```typescript
// libs/breez-fake-app/my-lib/src/index.ts
export * from './lib/my-lib.service';
export * from './lib/my-lib.component';
```

## Работа с API

### 1. Создание API сервиса

```typescript
@Injectable({
  providedIn: 'root',
})
export class MyApiService extends BaseApiService {
  constructor(http: HttpClient) {
    super(http, '/api/my-endpoint');
  }

  getItems(): Observable<Item[]> {
    return this.get<Item[]>('');
  }

  createItem(item: CreateItemRequest): Observable<Item> {
    return this.post<Item>('', item);
  }
}
```

### 2. Создание моделей

```typescript
export interface Item {
  id: string;
  name: string;
  description?: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateItemRequest {
  name: string;
  description?: string;
}
```

## Работа с формами

### 1. Reactive Forms

```typescript
export class MyFormComponent {
  form = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email]],
    description: [''],
  });

  constructor(private fb: FormBuilder) {}

  onSubmit(): void {
    if (this.form.valid) {
      const data = this.form.value;
      // Обработка данных
    }
  }
}
```

### 2. Template

```html
<form [formGroup]="form" (ngSubmit)="onSubmit()">
  <tui-input formControlName="name" placeholder="Название"></tui-input>
  <tui-input formControlName="email" placeholder="Email"></tui-input>
  <tui-textarea formControlName="description" placeholder="Описание"></tui-textarea>

  <button tuiButton type="submit" [disabled]="form.invalid">Сохранить</button>
</form>
```

## Работа с таблицами

### 1. Использование TuiTable

```typescript
export class MyTableComponent {
  columns: TuiTableColumn[] = [
    { key: 'name', title: 'Название' },
    { key: 'email', title: 'Email' },
    { key: 'actions', title: 'Действия' },
  ];

  data$ = this.dataService.getData();

  constructor(private dataService: MyDataService) {}
}
```

### 2. Template

```html
<tui-table [columns]="columns" [data]="data$ | async">
  <ng-container tuiTableColumn="name">
    <span>{{ item.name }}</span>
  </ng-container>

  <ng-container tuiTableColumn="email">
    <span>{{ item.email }}</span>
  </ng-container>

  <ng-container tuiTableColumn="actions">
    <button size="s" tuiButton (click)="edit(item)">Редактировать</button>
  </ng-container>
</tui-table>
```

## Работа с диалогами

### 1. Создание диалога

```typescript
@Component({
  selector: 'sp-my-dialog',
  standalone: true,
  template: `
    <tui-dialog>
      <tui-dialog-header>
        <h2>Заголовок диалога</h2>
      </tui-dialog-header>

      <tui-dialog-content>
        <!-- Содержимое диалога -->
      </tui-dialog-content>

      <tui-dialog-footer>
        <button tuiButton (click)="close()">Отмена</button>
        <button tuiButton type="submit">Сохранить</button>
      </tui-dialog-footer>
    </tui-dialog>
  `,
})
export class MyDialogComponent {
  constructor(private dialogRef: TuiDialogRef<void>) {}

  close(): void {
    this.dialogRef.close();
  }
}
```

### 2. Открытие диалога

```typescript
export class MyComponent {
  constructor(private dialogService: TuiDialogService) {}

  openDialog(): void {
    this.dialogService
      .open(MyDialogComponent, {
        size: 'm',
      })
      .subscribe();
  }
}
```

## Тестирование

### 1. Unit тесты

```typescript
describe('MyComponent', () => {
  let component: MyComponent;
  let fixture: ComponentFixture<MyComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [MyComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

### 2. E2E тесты

```typescript
describe('My Feature', () => {
  beforeEach(() => {
    cy.visit('/my-feature');
  });

  it('should display the feature', () => {
    cy.get('[data-cy="my-feature"]').should('be.visible');
  });

  it('should allow user interaction', () => {
    cy.get('[data-cy="my-button"]').click();
    cy.get('[data-cy="result"]').should('contain', 'Expected result');
  });
});
```

## Стилизация

### 1. SCSS переменные

```scss
// apps/breez-fake-app/src/theme/_variables.scss
$primary-color: #007bff;
$secondary-color: #6c757d;
$success-color: #28a745;
$error-color: #dc3545;
$warning-color: #ffc107;
```

### 2. Миксины

```scss
// apps/breez-fake-app/src/theme/_mixins.scss
@mixin button-variant($color) {
  background-color: $color;
  border-color: $color;

  &:hover {
    background-color: darken($color, 10%);
    border-color: darken($color, 10%);
  }
}
```

### 3. Использование в компонентах

```scss
// my-component.component.scss
.my-button {
  @include button-variant($primary-color);
  padding: 8px 16px;
  border-radius: 4px;
}
```

## Лучшие практики

### 1. Именование

- Компоненты: `MyComponentComponent`
- Сервисы: `MyService`
- Модели: `MyModel`
- Константы: `MY_CONSTANT`
- Перечисления: `MyEnum`

### 2. Структура файлов

```
feature/
├── components/
│   ├── my-component/
│   │   ├── my-component.component.ts
│   │   ├── my-component.component.html
│   │   ├── my-component.component.scss
│   │   └── my-component.component.spec.ts
├── services/
│   ├── my-service.ts
│   └── my-service.spec.ts
├── models/
│   └── my-model.ts
└── index.ts
```

### 3. Импорты

```typescript
// Порядок импортов:
// 1. Angular imports
import { Component, OnInit } from '@angular/core';

// 2. Third-party imports
import { Observable } from 'rxjs';

// 3. Internal imports
import { MyService } from './my.service';
```

### 4. Типизация

```typescript
// Всегда используйте типы
interface User {
  id: string;
  name: string;
  email: string;
}

// Используйте generic типы
getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users');
}
```

### 5. Обработка ошибок

```typescript
getData(): Observable<Data[]> {
  return this.http.get<Data[]>('/api/data').pipe(
    catchError(error => {
      console.error('Error fetching data:', error);
      this.notificationService.showError('Ошибка загрузки данных');
      return of([]);
    })
  );
}
```

## Отладка

### 1. Angular DevTools

- Установите Angular DevTools в браузере
- Используйте для отладки компонентов и сервисов

### 2. Console логирование

```typescript
// Используйте console.log для отладки
console.log('Component data:', this.data);

// Используйте console.error для ошибок
console.error('Error occurred:', error);
```

### 3. RxJS отладка

```typescript
import { tap } from 'rxjs/operators';

this.dataService
  .getData()
  .pipe(tap((data) => console.log('Data received:', data)))
  .subscribe();
```

## Производительность

### 1. OnPush Change Detection

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class MyComponent {}
```

### 2. TrackBy функции

```typescript
trackByFn(index: number, item: any): any {
  return item.id;
}
```

### 3. Lazy Loading

```typescript
{
  path: 'feature',
  loadChildren: () => import('./feature/feature.module').then(m => m.FeatureModule)
}
```

## Безопасность

### 1. Санитизация HTML

```typescript
import { DomSanitizer } from '@angular/platform-browser';

constructor(private sanitizer: DomSanitizer) {}

getSafeHtml(html: string): SafeHtml {
  return this.sanitizer.bypassSecurityTrustHtml(html);
}
```

### 2. Валидация данных

```typescript
// Всегда валидируйте данные от API
interface ApiResponse {
  data: unknown;
  status: string;
}

validateResponse(response: ApiResponse): boolean {
  return response.status === 'success' && response.data !== null;
}
```

## Развертывание

### 1. Environment конфигурация

```typescript
// environment.prod.ts
export const environment = {
  production: true,
  apiUrl: 'https://api.production.com',
  centrifugeUrl: 'wss://ws.production.com/connection/websocket',
};
```

### 2. Docker

```dockerfile
FROM nginx:alpine
COPY dist/apps/breez-fake-app/browser /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 3. CI/CD

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build
      - run: npm run test
```
